## How to deploy an internal container registry on DigitalOcean Kubernetes Cluster

You gonna need:
- Digital Ocean account (https://www.digitalocean.com/)
- kubectl (https://kubernetes.io/docs/tasks/tools/)


Clone trow repository
```
git clone https://github.com/ContainerSolutions/trow
```
Set namespace and create Trow resources
```
cd trow/quick-install
namespace=trow
sed "s/{{namespace}}/${namespace}/" trow.yaml | kubectl apply -f -
```

Approving certificate.
```
kubectl certificate approve "trow.${namespace}"
```
Define certfile var as /tmp/ca-cert.XXXXXX
```
cert_file=$(mktemp /tmp/ca-cert.XXXXXX)
```

Saving cluster certficate as trow-ca-cert
```
kubectl config view --raw --minify --flatten \
  -o jsonpath='{.clusters[].cluster.certificate-authority-data}' \
  | base64 --decode | tee -a $cert_file
kubectl create configmap trow-ca-cert --from-file=cert=$cert_file \
  --dry-run=client -o json | kubectl apply -n "$namespace" -f -
```  
Copy certificate to cluster nodes
```
./copy-certs.sh "$namespace"
```
To test your registry from outside the cluster install ed and run configure-host.sh to setup Docker in your host:
```
apt-get install ed -y
./configure-host.sh --namespace="$namespace" --add-hosts;
```

Resources:

https://github.com/digitalocean/Kubernetes-Starter-Kit-Developers

https://github.com/ContainerSolutions/trow


digitalocean-kubernetes-challenge
https://www.digitalocean.com/community/pages/kubernetes-challenge
