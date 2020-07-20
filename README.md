Analytics
============


To run locally on linux systems 
create kind cluster with haproxy, load this application image in that (for testing locally)
configure metallb for load balancing, specify namespace, metallb,secret and configmap with specific ports.


# build container with docker

`docker build -f Dockerfile -t sargam-analytics:latest .`

test it out and run 
`docker run -it -p 5000:5000 sargam-analytics:latest`


setup kind k8s cluster for local development
=======================

apply kind config 

`kubectl config use-context kind-dev-py`

`kind create cluster --name dev-py --config kind-example-config.yaml`
`kind load docker-image sargam-analytics --name dev-py`

`kind delete cluster --name dev-py`

setup metalb for load balancing


# setup metallb instructions
`kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml`
`kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml`
# On first install only
`kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"`

# now run kubectl get nodes -o wide see the IP range
`kubectl get nodes -o wide`
# run this and keep the IPs different from the above IP range cause in the above IP range those are the nodes.

```cat <<EOF | kubectl create -f -                      

apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 172.18.0.220-172.18.0.250
EOF
```

# apply deployment and services to k8s cluster

`kubectl apply -f deployment`