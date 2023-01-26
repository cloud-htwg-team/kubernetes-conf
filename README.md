# cloud-app-dev-project
HTWG Cloud Application Development Project

## QR Code microservice 
- `JAVA 17`
- library
  - https://github.com/nayuki/QR-Code-generator
- build Docker image
  - `docker build  -t qrcode_generator .`
- run Docker image
  - `docker run -d -p 8080:8080 qrcode_generator`
  - `-d` is for `detached mode`
  
### tag image with a repository name on dockerhub
- docker tag qrcode_generator zeyesm/java-app-qr
### push image to dockerhub
- docker push zeyesm/java-app-qr

## for local setup 
- `install minikube`
- minkube start
- `install kubectl`
check if it's pointing to minikube :
- kubectl config current-context 

## kubernetes deployment commands
- kubectl create -f kube/qr-nodeport.yaml
- kubectl create -f kube/qr-cluster-service.yaml
- kubectl create -f kube/qr-deployment.yaml
- kubectl get deployments
- kubectl get pods -o wide
- kubectl get services

### Kubernetes service activation commands
- minikube service qrjava-nodeport # on a separate terminal
- kubectl port-forward service/qrjava-nodeport 30080:8080

### Kubernetes service deactivation commands
- kubectl delete services/qrjava-nodeport
- kubectl delete deployments/qrjava

### kubernetes commands 
- kubectl exec -it qrjava-776774968-5hhnz --namespace default -- bash
- kubectl logs qrjava-776774968-5hhnz --namespace default
- kubectl describe pod qrjava-776774968-5hhnz

# Useful commands for gcloud and kubernetes
- gcloud components install kubectl
- gcloud auth login
- gcloud config set project PROJECT_ID

`make sure you are in the right context, not on local machine`
- kubectl config current-context

## create a cluster 
- gcloud container clusters create qr-code-app --release-channel None --zone europe-west3-a --node-locations europe-west3-a
### connect to the cluster
- gcloud container clusters get-credentials qr-code-app --zone europe-west3-a

### for testing, add a pod to the cluster with the docker image in containers of gcloud
- kubectl run qrcode-java-app --image eu.gcr.io/qrcode-374515/qrcode-service --cluster gke_qrcode-374515_europe-west3-a_qr-code-app

### add your own kubes from manifest to the cluster, works for pods and services
- kubectl apply -f qr-nodeport.yaml

## add an ingress
`download helm`
- helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
- helm repo update
- helm install nginx-ingress ingress-nginx/ingress-nginx
- kubectl get deployment nginx-ingress-ingress-nginx-controller
- kubectl get service nginx-ingress-ingress-nginx-controller
 `keep the ip in mind or in a variable and add it to the dns of your web domain`
- kubectl apply -f ingress-resource.yaml
- kubectl describe ingress-ressource 

### create a secret with certificate and key
- kubectl create secret tls qreach-ingress-tls --key qreach-ingress-tls.key --cert qreach-ingress-tls.crt

### add virtualserver
`in helm repo`
- git clone https://github.com/nginxinc/kubernetes-ingress/
- cd kubernetes-ingress/deployments
- kubectl apply -f common/crds/k8s.nginx.org_virtualservers.yaml
- kubectl apply -f common/crds/k8s.nginx.org_virtualserverroutes.yaml
- kubectl apply -f common/crds/k8s.nginx.org_transportservers.yaml
- kubectl apply -f common/crds/k8s.nginx.org_policies.yaml

### delete an ingress
- kubectl delete -f ingress-resource.yaml
`in helm terminal`
- helm del nginx-ingress

## setup tls key with linux and openssl
- openssl req -new -x509 -days 9999 -keyout ca-key.pem -out ca-crt.pem
- openssl genrsa -out server-key.pem 2048
`the CN must be your dns`
- openssl req -new -key server-key.pem -out server-csr.pem 
- openssl x509 -req -days 9999 -in server-csr.pem -CA ca-crt.pem -CAkey ca-key.pem -CAcreateserial -out server-crt.pem
`back on gcloud`
- kubectl create secret tls qreach-ingress-tls --key server-key.pem --cert server-crt.pem


## Workload identity
 - enable workload identity du cluster
 - type following command to update node pool
 - gcloud container node-pools update default-pool --cluster=qr-code-app --workload-metadata=GKE_METADATA --region europe-west3-a
 ### create account with the access
 - gcloud iam service-accounts create iam-kube-account
 - gcloud iam service-accounts add-iam-policy-binding iam-kube-account@qrcode-374515.iam.gserviceaccount.com --role roles/iam.workloadIdentityUser --member "serviceAccount:qrcode-374515.svc.id.goog[default/default]"
- kubectl annotate serviceaccount default--namespace default iam.gke.io/gcp-service-account=iam-kube-account@qrcode-374515.iam.gserviceaccount.com

### Google info
- @google-cloud/firestore
- @google-cloud/storage
- create a key for each user
