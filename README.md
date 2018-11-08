# Magento Kubernetes components

## NFS server

Configure your sources directory as export (/etc/exports) in the NFS server that runs on your host machine. This way containers can mount source code. This is a lot faster than a default VirtualBox shared folder mount. You only have to do this once, the NFS service will load /etc/exports at (re)boot.

NOTE: The Minikube IP can be different after a minikube delete and minikube start command. Make sure that your NFS export contains the correct Minikube IP again.

### Mac OS X
echo "$(realpath .)/sources -alldirs -mapall="$(id -u)":"$(id -g)" $(minikube ip)" | sudo tee -a /etc/exports && sudo nfsd restart
Check if the entry is active by executing on your host machine:

```bash
showmount -e 127.0.0.1
```

This should output something like:

```bash
Exports list on 127.0.0.1:
/Absolute/path/to/sources 192.168.99.100
```

## Apply configs

```bash
kubectl apply -f config/default-sources-volume.yaml -f config/default-sources-volume-claim.yaml -f config/magento2-deployment.yaml
```
