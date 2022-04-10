# Kubernetes-basics

~~~
kubectl run hello-minicube
kubectl cluster-info
kubectl get nodes
# get OS of the node
kubectl get nodes -o wide 
~~~

## Set up kubectl

~~~bash
# update apt repo and package installation
apt-get update
apt-get install -y apt-transport-https ca-certificates curl

# Download google cloud public signing key
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

# Add Kubernetes Repo
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install kubectl
apt-get update
apt-get install -y kubectl
~~~

# Install minikube ( Ubuntu)

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
install minikube-linux-amd64 /usr/local/bin/minikube
# If any error is encounetred , reload the profile
# Install docker, change the instance type to t3medium add ubuntu user in docker usergroup 
sudo usermod -aG docker ubuntu
minikube start
minikube status

# sample deployment
kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4
kuebctl get deployments

#Expose it as a service 
kubectl expose deployment hello-node --type=LoadBalancer --port=8080

minikube service hello-node --url

# cleanup env
kubectl delete service hello-node
kubectl delete deployment hello-node 


# Create POD
kubectl run nginx --image=nginx
# POD status
kubectl get pods
# decribe pods
kubectl describe pods <podename>
# delete pod
kubectl delete pod <podname>
# to get more information on pods

kubectl get pods -o wide

```

## sample yaml file for pod creation

~~~yml
apiVersion: v1
kind: Pod
metadata:
    name: myapp-pod
    labels:
        app: myapp
        type: frontend
spec:
    containers:
        - name: nginx-container
          image: nginx
~~~

~~~bash
# create a pod by yaml file
kubectl create -f pod-definition.yml 


#createReplication Controller( rc-definition.yml)

$kubectl create -f rc-definition.yml
replicationcontroller/myapp-rc created

$kubectl get replicationcontroller
NAME       DESIRED   CURRENT   READY   AGE
myapp-rc   3         3         3       2m30s

$kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
myapp-rc-lr5jv   1/1     Running   0          3m1s
myapp-rc-m69wm   1/1     Running   0          3m1s
myapp-rc-xk9gt   1/1     Running   0          3m1s

$kubectl delete rc myapp-rc
replicationcontroller "myapp-rc" deleted


# Create replicaset ()
$kubectl create -f replicaset-definition.yaml
replicaset.apps/myapp-replicaset created

$kubectl get replicaset
NAME               DESIRED   CURRENT   READY   AGE
myapp-replicaset   3         3         3       111s

$kubectl get pods

NAME                     READY   STATUS    RESTARTS   AGE
myapp-replicaset-4bjw8   1/1     Running   0          2m29s
myapp-replicaset-fg74t   1/1     Running   0          2m29s
myapp-replicaset-h2vtp   1/1     Running   0          2m29s

# increase the replica
$kubectl replace -f replicaset-definition.yaml
replicaset.apps/myapp-replicaset replaced

$kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
myapp-replicaset-4bjw8   1/1     Running   0          17m
myapp-replicaset-fg74t   1/1     Running   0          17m
myapp-replicaset-h2vtp   1/1     Running   0          17m
myapp-replicaset-r9xv2   1/1     Running   0          6s

$kubectl scale --replicas=6 -f replicaset-definition.yaml
replicaset.apps/myapp-replicaset scaled
# yaml is not updated when scaling is done through cli

$kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
myapp-replicaset-4bjw8   1/1     Running   0          19m
myapp-replicaset-bz55d   1/1     Running   0          28s
myapp-replicaset-fg74t   1/1     Running   0          19m
myapp-replicaset-h2vtp   1/1     Running   0          19m
myapp-replicaset-m2s7g   1/1     Running   0          28s
myapp-replicaset-r9xv2   1/1     Running   0          119s

$kubectl scale --replicas=8 replicaset myapp-replicaset
replicaset.apps/myapp-replicaset scaled

$kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
myapp-replicaset-4bjw8   1/1     Running   0          24m
myapp-replicaset-6bsc8   1/1     Running   0          17s
myapp-replicaset-bz55d   1/1     Running   0          5m44s
myapp-replicaset-fg74t   1/1     Running   0          24m
myapp-replicaset-h2vtp   1/1     Running   0          24m
myapp-replicaset-hktbt   1/1     Running   0          17s
myapp-replicaset-m2s7g   1/1     Running   0          5m44s
myapp-replicaset-r9xv2   1/1     Running   0          7m15s

# check the rs and pod details
$ kubectl describe rs myapp-replicaset
$ kubectl edit rs myapp-replicaset

$ kubectl delete rs myapp-replicaset
replicaset.apps "myapp-replicaset" deleted

# deployment
$ kubectl create -f deployment.yaml
deployment.apps/myapp-deployment created

$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
myapp-deployment   3/3     3            3           13s

$ kubectl describe deployment  <deploymentanme>
$ kubectl get all

NAME                                    READY   STATUS    RESTARTS   AGE
pod/myapp-deployment-6ffd468748-8kv5g   1/1     Running   0          4m35s
pod/myapp-deployment-6ffd468748-fnq9f   1/1     Running   0          4m35s
pod/myapp-deployment-6ffd468748-mscc4   1/1     Running   0          4m35s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   7d23h

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/myapp-deployment   3/3     3            3           4m35s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-deployment-6ffd468748   3         3         3       4m35s

# deployment strategy - Recreate & Rolling out , Rollback
# create

$ kubectl create -f deployment.yaml --record
deployment.apps/myapp-deployment created

#update

kubectl edit deployment myapp-deployment --record


kubectl apply -f deployment-definition.yml
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1

#status
$ kubectl rollout status deployment/myapp-deployment
deployment "myapp-deployment" successfully rolled out

$ kubectl rollout history deploy
deployment.apps/myapp-deployment
REVISION  CHANGE-CAUSE
1         kubectl create --filename=deployment.yaml --record=true

# rollback
kubectl rollout undo deployment/myapp-deployment 
