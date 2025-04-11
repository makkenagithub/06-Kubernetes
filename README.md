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

pod: pod is a smallest deployable unit/resource in k8s. Pod can contain one or many containers.

pod vs container:

1. pod is smallest deployable unit in k8s
2. pod can contain one/many containers
3. containers in  a pod can share same network identity i.e. All the container share same IP, port and also storage
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








