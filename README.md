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
