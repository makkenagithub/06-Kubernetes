# 06-Kubernetes

### Docker Disadvantages
1. There is no reliability as there is only one docker host. If some problem (network issue etc) comes to docker host server, then containers go down
2. There is no auto scaling and no load balancing
3. Volumes are inside docker host. So poor volume management
4. Security - no secret management
5. No proper communication between containers i.e one container in a docker wants to communicate with another container in another docker host, its not possible. - network management is not good

As all the above features are not possible with docker, we need something called orchestration. 
Orchestration means, as the number of hosts increase , we need someone to manage them (someone need to give instructions to them / guide them to manage them properly) i.e. orchestration.

Docker swarm is an orchestration tool from Docker. But its very poor. We have another orchestration tool i.e. Kubernetes.

We build the docker images with Docker and we run the images with Kubernetes.

### Kubernetes:
We have master / control plane and nodes.

We are writing docker files and storing them to github. We pull the docker files to docker host and build images and then stores them to dcoker hub. 
After that here in K8s , nodes can pull the docker images from docker hub directly.  Nodes pull the images from docker hub through instructions. 
So we have to give instructions from master node. 

Wehn a service is running, we need a client to communicate with that service. To communicate with facebook server, we need http client. Similarly ssh client, mysql client, docker client etc. 
So here in k8s we have kubectl client. To communicate with k8s service we need kubectl client. 

kubectl -> is k8s client
eksctl -> is command to create, update , delete cluster. eksctl is to manage cluster. eksctl is provided by AWS

Usually we create a server called work station. Workstation server is having docker, kubectl, eksctl. This server need to be communicated with AWS. 
So aws configure (aws cli) also exists in this work station server.

As we have docker compose file (yaml) in docker, similarly we have Manifest files (yaml file) in k8s. We create manifest files and push to github, then pull them to workstation server.

Assignment:
1. there is a docker host running. No space left in the server. We need to add extra disk to running instance.
2. Make sure docker directory /var/lib/docker is mounted to new disk. Migrate the existing date to new mount.

In interveiews, they may ask, how you add extra volume to running instance. 

Setup for workstation server:
1. Create t3 micro instance. Make sure atleast 50gb and allocate more storage to /var. Refer resize_desk file in docker git
2. install docker
3. install kubectl
4. intsall eksctl
5. run aws configure

Initailly login to aws instace created in step 1 with ec2-user and then follow the next steps

step 2:

To install docker / to run a shell script  by using script present in github
```
curl <git url> | sudo bash
```
step 3:

kubectl installation commands from aws documentation
https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html
```
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.31.3/2024-12-12/bin/linux/amd64/kubectl
chmod +x ./kubectl
```
Move the kubctl to /usr/local/bin to run the kubectl commands by any user
```
sudo mv kubectl /usr/local/bin/kubectl
or
sudo mv kubectl /usr/local/bin/
```
check the version
```
kubectl version
```

step 4:

install eksctl from official aws site https://eksctl.io/installation/

```
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
step 5:

run the aws configure command and enter the access key ID of aws account
```
aws configure
```

We can try all these steps by using terraform and keep these shell scrips commands in user_data. steps 5 can be run ouside of shell script i.e. manually in ec2 instance.

Next create eks cluster  https://eksctl.io/getting-started/

create cluster.yaml file. To create spot instance we need to add spot: true fiels in manifest file
```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: expense
  region: us-east-1

managedNodeGroups:
  - name: ng-1
    instanceType: m5.large
    desiredCapacity: 3
    spot: true
```
```
eksctl create cluster -f cluster.yaml
or
eksctl create cluster --config-file=cluster.yaml
```
To delete the cluster
```
eksctl delete cluster --config-file=cluster.yaml
```

spot instances:

AWS has huge data center. And there may be lot of unused capacity in data center. Until now we created on demand ec2 instances.

spot instances -> 90% discount. When AWS requires this unused capacity to on demand clients , aws can take back the instances with 2 minutes notice.

We can't run production work loads in spot instances. We may run dev/testing purpose work loads in spot instances.

After creating the cluster , to see list of nodes or to see howmnay nodes are there in cluster
```
kubectl get nodes
```

Syntax in k8s:

apiVersion:
kind: 
metadata:
  name:
  labels:
spec:

This is general/overall sysntax in k8s manifest file. It changes slightly based on the reosurce we are creating.

### Resources

Namesapaces: Just like vpc we will have a dedicated isolated project to create our workloads / resources. Its a isolated project work space in the cluster.
The resources in one name space cant be accessed by resources in another work space.

After creating a resource yaml file, we can apply it in ec2 instance using kubectl apply command
```
kubectl apply -f <reosurce-file-name.yaml>
```
To delete the above resource
```
kubectl delete -f <reosurce-file-name.yaml>
```

To list the namespace in k8s
```
kubectl get namespaces
```
#### pod
pod: pod is a smallest deployable unit/resource in k8s. Pod can contain one or many containers.

pod vs container:

1. pod is smallest deployable unit in k8s
2. pod can contain one/many containers
3. all containers in  a pod can share same network identity i.e. All the container share same IP, port and also storage
4. these are useful in sidecar and proxy patterns

Assume we have nginx container in a pod. Nginx writes logs to /var/log/nginx directory in container. When the container is deleted, we lost the logs.
Hence in k8s we write logs to elastic search.

To go inside the pod:
```
kubectl exec -it <pod-name> -- bash
```
When a pod is having multiple containers, To go inside a specific container, the command is
```
kubectl exec -it <pod-name> -c <container-name> -- bash
```

A Pod can't contain two containers with same name. Container names should be different in a pod.
But pod1 has nginx container. And pod2 can have the same container name (nginx) of other pod pod1.

TO see full info about a pod
```
kubectl describe pod <pod-name>
docker inspect <container-name/id>
```

#### Annotations:

Labels are used for k8s internal resource selctors. Annotations are used for external resource selectors. We can use special chars, URLs in annotations, but not in labels.

ENV in Docker image file vs ENV in k8s manifest file:

1. If any change made in ENV in docker, we need to rebuild the image and then need to restart (delete and apply) the manifest in k8s.
2. If any change made in ENV in k8s manifest file, then only restart (delete and apply manifest file) is enough

env in k8s manifest file is a list of key value pairs.We can see the env value by going insdie the pod and type env and press enetr and see env variables in the pod.

#### Resource utilisation in K8s and Docker:

In docker assume 5 containers are running in a host. Due to some code issue in one container, say memory leak, this one container can start consuming more memory and due to that other containers can stop working due to low memory.

In k8s we can acllocate resource softlimit, hardlimit.

softlimit (request) :- (100m, 64Mi) is used when the pod/container is getting started.

hardlimit (limit): (120m, 128Mi) is used when pod/container is running. Pod can use this limit as max values when running.

1 cpu = 1000 milli core cpu.

We have pod resoruce limits and container resource limits in k8s. https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

If the hard limit also crosses while running the container, then OOM (out of memory error comes)

#### Config-map as env variable
configmap is also similar to env variable. which can be used as variable in the code
To see the config maps
```
kubectl get configmap
kubectl describe configmap <configmap-name>
```

When a config map is created and its used by some pod. Then assume pod is running. Now we can make changes to config map without touching the pod manifest file and the change will reflect in the pod after restarting the pod
```
kubectl edit configmap <config-map-name>
kubectl delete -f <pod-file-name>
kubectl apply -f <pod-file-name>
```
vi editor will open for the above edit command and make the changes. The changes key value reflects in the pod after restarting pod.

We can refer each key from a config map as env variabele, or we can also refer entire config map as env variable.

configmap is for non-confidential information.

#### Secret:
Its similar to config maps, but we can use for confidential info, like passwords. We need to put encoded values in manifest

We need to encode the confidential things and then write to manifest file.
Encode with base64 as below
```
echo "username" | base64
echo "password" | base64
```
To decode
```
echo "c3VyZXNoCg==" | base64 --decode
```

We create secrets with type Opaque in manifest files. TO see list of secrets
```
kubectl get secrets
kubectl describe secret <secret-name>
```
But this kind of encoding/decoding is not used in real time , as others can easily decode the secrets. So in realtime  we use encryption by using aws secret manager / aws ssm paramstore

HOW we can access the pod externally by using internet ? In DOcker we have host port and container port. But how in k8s?

Ans: By exposing to services.

#### Services:
Pod to pod communications can happen. Get the IP of a pod by describing it. Then go inside a pod and curl that IP, it will be accessable. But POD IPs are ephemeral (temporary).
In VMs we used route53 resords to overcome this IP changes. Everytime IP changes, we update route53 records and then access with DNS. But in k8s we have services.

Services are 3 types in k8s.
1. cluster IP - its a default service. Its for internal pod to pod communication
2. node port - open a port on underlying node(host) (worker node)
3. load balancer

When we create a service, based on the selectors given in the service, that service will go and attach to that pod. 
When we create pods, we give labels to it. The services attach to the pods by using selectors.

Service will have a port.

service example: https://kubernetes.io/docs/concepts/services-networking/service/
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: proxy
spec:
  containers:
  - name: nginx
    image: nginx:stable
    ports:
      - containerPort: 80
        name: http-web-svc

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app.kubernetes.io/name: proxy
  ports:
  - name: name-of-service-port
    protocol: TCP
    port: 80
    targetPort: http-web-svc
```
To get services
```
kubectl get services
```
To know , on which worker node , pords are running
```
kubectl get pods -o wide
```
After creating a node-port servie, k8s allots a port number from range 30000 to 32767 to the all worker nodes. But we need to open the inbound rule for that port from any one worker node.
Once we open the port say 32276 from a worker node, then that port will automatically opens that port for other worker nodes also. Its the responsibility of k8s. Because the pod can run on any of the worker node.

When we create a node-port service, it also opens cluster IP also, (see with kubectl get services command). So cluster-IP is subset of node-port

When we create a load balancer service, it opens a node-port  range 30000 to 32767 also. Directly with the lb url, we are able to access. Node-port is subset of lb service. cluster-IP is subset of Node-port.

If we want a specific node port to open, then we can add below filed in the manifest
```
nodePort: 31200
```

Services are used for pod to pod communication using dns and also load balancing purpose. Expose the pod using service to access from internet.


#### Sets:
1. ReplicaSet
2. DeploymentSet
3. DaemonSet
4. StatefulSet

Replicaset make sure our desired number of pods running all the time. Replica set exists in apiVersion apps/v1

Once we create a replica set and attach it to pods, then if we modify the replica set with the updated image version, and then apply this replica set with (kubectl apply -f replica-set.yaml). 
it wont change the pods with new image verion. So replicaset can't update image version, its only responsibility is to maintain desired number of replicas. 
So the disadvantage of replicaset is , it wont update if the image version updates are there. we can overcome this with deployment.

To see replicaset
```
kubectl get replicaset
```
To delete the replica set
```
kubectl delete -f <replicaset.yaml>
```

TO get/delete the deployments
```
kubectl get deployment
kubectl get pods
kubectl delete deployment <deployment-name>
```
Deployment will create replicaset. So replica set is part/subset of deployment. Replica set creates pods.
After applying deployment.yaml, see the pods and observer pod names.

Change the image version and apply the deployment file. Pods will be updated with new image version. Replicaset also gets updated.
So when we change the image version, Deployment will create new replicaset, and creates new pods in the new replica set and deleted the pods in old replicaset.

Pod is subset of replicaset. Replicaset subset of deployment. 
We use deployments in projects.

To get pods in a particular namespace and go inside it
```
kubectl get pods -n <namespace>
kubectl exec -it <pod-name> -n <namespace> -- bash
```
To get services
```
kubectl get services -n <namespace>
```
Its always better give app configuration variables during run time i.e. in manifest file. Just by deploying the manifest, app configs come into effect.
But If we give in docker file, we need to rebuild the docker image and then deploy the manifest.

TO see the logs
```
kubectl logs <pod-name> -n <namespace>
```
Everytime we need to give namespace explicitly. We can instal kubectx and set the namespace. https://github.com/ahmetb/kubectx
```
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
kubens <namespace>
```

In minimum size images, we may not find all commands (such as telnet, curl etc) inside the pod. For this purpose we can create a debug docker image and run that debug pod to troubleshoot issues.














