# K3S control

## install

`curl -sfL https://get.k3s.io | sh -s - server --disable=servicelb,local-storage --write-kubeconfig-mode=644`
- add to .bashrc `source <(kubectl completion bash)`

### Helm

- download release binary from `https://github.com/helm/helm/releases`
- extract `tar -zxvf helm-v3.0.0-linux-amd64.tar.gz`
- copy bin `sudo mv linux-amd64/helm /usr/local/bin/helm`
- make a copy of kubeconfig so helm can use it `kubectl config view --raw >~/.kube/config`
  - that last one will depend on where helm is being run and if it has access to a kubeconfig already, this is all being run on the vm

### metallb

- using helm `helm repo add metallb https://metallb.github.io/metallb`
- update repos `helm repo update`

**But first!**
- grab the default values from helm because we need to check and fix!
`helm show values metallb/metallb >> metallb.yaml`
- made some changes to the configInLine section

- manually add a namespace `kubectl create ns metallb-system`
- `helm install metallb metallb/metallb -f yamls/metallb.yaml --namespace metallb-system`

### Longhorn

`kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/master/deploy/longhorn.yaml`
 then to expose the dashboard temporarily
`kubectl -n longhorn-system expose deployment longhorn-ui --type=LoadBalancer`

### drone

- `helm repo add drone https://charts.drone.io`
- `helm repo update`
- `helm show values drone/drone >> yamls/drone.yaml`

#### secret
- make a shared secret `openssl rand -hex 16`
- get the github oauth clientID/client secret
- `kubectl create namespace drone`
```
kubectl -n drone create secret generic my-drone-secrets \
--from-literal=DRONE_RPC_SECRET='<---SeCrEtKey--->' \
--from-literal=DRONE_GITHUB_CLIENT_ID='S!B\*d$dsfdfdfzDsb=' \
--from-literal=DRONE_GITHUB_CLIENT_SECRET='S!B\*d$zDfdsfdgdfdafsb='
```
- edit the drone yaml to use the secret, add the host and proto

- `helm install --namespace drone drone drone/drone -f yamls/drone.yaml`