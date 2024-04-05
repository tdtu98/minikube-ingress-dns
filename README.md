# Minikube-ingress-dns

This is my note for using ingress-dns inside minikube on ARM64 environment (Mac m1).

## Environment
+ Mac m1 Sonoma 14.0
+ Docker desktop 4.28.0
+ Minikube  v1.32.0
+ Kubectl v1.29.3
## Usage
In this project, I used the hello-world-app.yaml which is simillar to the one in the [official tutorial of minikube](https://minikube.sigs.k8s.io/docs/handbook/addons/ingress-dns/#mac-os). In minikube for mac, we cannot directly connect to pods of cluster inside minikube without using minikube tunnel. To fix that, please install [docker-mac-net-connect](https://github.com/chipmk/docker-mac-net-connect):
```
brew install chipmk/tap/docker-mac-net-connect
sudo brew services start chipmk/tap/docker-mac-net-connect
```
Since the hello world app and kube-ingress-dns-minikube only supports amd64 arch, we cannot run them inside the minikube. To fix that please run these commands:
```
minikube ssh "sudo apt-get update && sudo apt-get -y install qemu-user-static"
minikube stop
minikube start
```
Then, you can follow the [tutorial of minikube](https://minikube.sigs.k8s.io/docs/handbook/addons/ingress-dns/#mac-os):
```
# Add the test ingress
kubectl apply -f hello-world-app.yaml

# check minikube ip
minikube ip

# confirm that dns query return record
nslookup hello-john.test $(minikube ip)

# adding minikube ip as a DNS server
sudo nano /etc/resolver/minikube-test
'''
domain test
nameserver $(minikube ip)
search_order 1
timeout 5
'''

# Confirm that domain names are resolving on the host OS
ping hello-john.test
ping hello-jane.test

# Check result
curl http://hello-john.test
curl http://hello-jane.test
```
Please notice that our top level domain is test as our webs are .test, you can change to other top level domains like .com, .net. However, this also affects your connection to other websites with the same top level domain.
## Reference
https://github.com/kubernetes/minikube/issues/12424#issuecomment-1929241678

https://github.com/kubernetes/minikube/issues/16530#issuecomment-2001903804

https://gitlab.com/cryptexlabs/public/development/minikube-ingress-dns/-/tree/master

https://minikube.sigs.k8s.io/docs/handbook/addons/ingress-dns/#mac-os
