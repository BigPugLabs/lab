# K3S control

## install

`curl -sfL https://get.k3s.io | sh -s - server --disable=servicelb,local-storage --write-kubeconfig-mode=644`

### Longhorn

`kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/master/deploy/longhorn.yaml`
 then to expose the dashboard temporarily
`kubectl -n longhorn-system expose deployment longhorn-ui --type=NodePort`

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

