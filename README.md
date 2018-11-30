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
If you do not have python, install it using brew:
```bash
brew install python
```
echo "$(python -c 'import os,sys;print os.path.realpath(".")')/sources -alldirs -mapall="$(id -u)":"$(id -g)" $(minikube ip)" | sudo tee -a /etc/exports && sudo nfsd restart
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

## Installation

To install Magento, use next variables:

| Name | Value|
| --- | --- |
| DB host | `magento2-mysql` |
| DB user | `root` |
| DB password | `1234` |
| DB name | `magento` |


### Installation from CLI:
`./bin/magento setup:install --db-host=magento2-mysql --db-name=magento --db-user=root  --db-password=1234 --backend-frontname=admin --base-url=http://192.168.99.100/ --language=en_US --timezone=America/Chicago --currency=USD --admin-lastname=Admin --admin-firstname=Admin --admin-email=admin@example.com --admin-user=admin --admin-password=123123q --cleanup-database --use-rewrites=1`

Notes:
1. `magento` DB must be created manually. Installation to the default `mysql` DB does not work.
1. `base-url` may vary.
1. Do not use `minikube service magento2` to open Magento instance because it uses port incompatible with current ingress configuration.


## Login to containers

1. Get list of available pods `kubectl get pods`
2. Login to Magento container on magento2 pod `kubectl exec -it <magento2-pod> --container magento2 -- /bin/bash`
2. Login to Nginx container on magento2 pod `kubectl exec -it <magento2-pod> --container nginx -- /bin/bash`
2. Login to MySQL container on mysql pod `kubectl exec -it <mysql-pod> /bin/bash`
