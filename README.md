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
  
# tag image with a repository name on dockerhub
docker tag qrcode_generator zeyesm/java-app-qr
# push image to dockerhub
docker push zeyesm/java-app-qr

# for local tests
- `install minikube`
- `better to pull the docker image before as pulling at kube creation doesn't always work`

# kubernetes deployment commands
kubectl create -f kube/qr-nodeport.yaml
kubectl create -f kube/qr-cluster-service.yaml
kubectl create -f kube/qr-deploiement.yaml
kubectl get deployments
kubectl get pods -o wide
kubectl get services

# Kubernetes service activation commands
minikube service qrjava-nodeport # on a separate terminal
kubectl port-forward service/qrjava-nodeport 30080:8080

# Kubernetes service deactivation commands
kubectl delete services/qrjava-nodeport
kubectl delete deployments/qrjava

# kubernetes useful commands 
kubectl exec -it qrjava-776774968-5hhnz --namespace default -- bash
kubectl logs qrjava-776774968-5hhnz --namespace default
kubectl describe pod qrjava-776774968-5hhnz


### Usage
curl  http://localhost:8080/qr-code --header "Content-Type: application/json" -d '{"text":"mael"}' --output output.png

Invoke-WebRequest -Uri http://localhost:8080/qr-code -Method Post -ContentType "application/json" -Body '{"text":"mael"}' -OutFile output.png

```
// returns dummy string just to verify its working
GET http://localhost:8080/


// returns generated qr code, you can change the text in request's payload
POST http://localhost:8080/qr-code
Content-Type: application/json

{
  "text": "htwg-konstanz.de"
}
```

---

### Potentional QR code libs:
- JS https://github.com/oblakstudio/qr-code-styling
- Python https://www.geeksforgeeks.org/how-to-generate-qr-codes-with-a-custom-logo-using-python/
