# Magento Kubernetes components

Install docker, enable kubernetes and install minicube.

Enable ingress addon.
```bash
minikube addons enable ingress
```

## NFS server

Configure your sources directory as export (/etc/exports) in the NFS server that runs on your host machine. This way containers can mount source code. This is a lot faster than a default VirtualBox shared folder mount. You only have to do this once, the NFS service will load /etc/exports at (re)boot.

NOTE: The Minikube IP can be different after a minikube delete and minikube start command. Make sure that your NFS export contains the correct Minikube IP again.

[More info](http://pietervogelaar.nl/minikube-nfs-mounts)

### Mac OS X
echo "$(python -c 'import os,sys;print os.path.realpath(".")')/magento -alldirs -mapall="$(id -u)":"$(id -g)" $(minikube ip)" | sudo tee -a /etc/exports && sudo nfsd restart
Check if the entry is active by executing on your host machine:

```bash
showmount -e 127.0.0.1
```

This should output something like:

```bash
Exports list on 127.0.0.1:
/Absolute/path/to/magento 192.168.99.100
```

## Apply configs
Without ingress and second service:
```bash
python local_deploy.py default-sources-volume default-sources-volume-claim.yaml magento2-deployment.yaml
```
With ingress, or all service:
```bash
python local_deploy.py --all --ingress
```

If you use ingress with same url but different subpath, 
Bash into pod and install magento via cli to set base urls.
