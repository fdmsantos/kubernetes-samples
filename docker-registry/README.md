# Deploy


## Nginx Ingress Controller in Bare Metal without TLS

```sh
# Deploy Nginx Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/service-nodeport.yaml

# Deploy Docker Registry
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/docs/examples/docker-registry/deployment.yaml

# Deploy Ingress Rule
kubectl apply -f ingress-without-tls.yaml

# To configure docker using insecure registry
https://docs.docker.com/registry/insecure/

sudo cat /etc/docker/daemon.json
{
     "insecure-registries" : ["NodePort:30359"]
}


# Get Endpoints
kubectl get svc ingress-nginx -n ingress-nginx
NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx   NodePort   100.67.255.166   <none>        80:30359/TCP,443:30648/TCP   63m


docker pull ubuntu:16.04
docker tag ubuntu:16.04 NodePort:30359/ubuntu:16.04
docker push NodePort:30359/ubuntu:16.04
```


## Nginx Ingress Controller in AWS with TLS

```sh
# Deploy Nginx Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml

# Create Certificate
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=registry.tiagomsantos.com/O=registry.tiagomsantos.com"
kubectl create secret tls tls-secret --key tls.key --cert tls.crt
# upload certificate to AWS ACM to be used by LoadBalancer
# Na AWS Route 53 add registry.tiagomsantos.com

# Deploy Ingress Service (AWS Load Balancer)
kubectl apply -f service-l7.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/aws/patch-configmap-l7.yaml

# Deploy Docker Registry
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/docs/examples/docker-registry/deployment.yaml


# Deploy Ingress Rule
kubectl apply -f ingress-with-tls.yaml

# Do the following steps to signed certificate

Go to /usr/local/share/ca-certificates/
Create a new folder, i.e. "sudo mkdir school"
Copy the .crt file into the school folder
Make sure the permissions are OK (755 for the folder, 644 for the file)
Run "sudo update-ca-certificates"


docker pull ubuntu:16.04
docker tag ubuntu:16.04 registry.tiagomsantos.com/ubuntu:16.04
docker push registry.tiagomsantos.com/ubuntu:16.04
```