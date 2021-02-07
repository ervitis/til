# Kubernetes

## KubeAdmn

### Install

For one node, a master, follow the next commands inside the file https://github.com/sighupio/k8s-conformance-environment/blob/master/modules/aws-k8s-conformance/templates/master.tpl.yaml

Install the dependencies:

- kubectl
- helm
- tiller (for version 2 of helm)
- kubeadm
- docker

To install kubeadm: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/


### Setup

Then, create a file inside the folder `/etc/NetworkManager/conf.d/calico.conf`, copy and paste the following:

```toml
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:tunl*
```

Execute the command:

```bash
kubeadm init --pod-network-cidr=192.168.0.0/16  --node-name=$(hostname -f)
```

And then the last commands

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Finally, because some servers and hostings have some problems writing data into disk, execute this

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.12/deploy/local-path-storage.yaml
kubectl patch storageclass local-path -p '{\"metadata\": {\"annotations\":{\"storageclass.kubernetes.io/is-default-class\":\"true\"}}}'
```

If the second one fails, just edit it manually with `kubectl get sc && kubectl edit sc <name>`

## Calico

### Setup

With `kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml`

## Imperative commands

[Here](./commands/index.md)

## About secrets in K8S

[Here](./secrets/index.md)

## Rolling strategy

[Here](./rollingupdate/index.md)
