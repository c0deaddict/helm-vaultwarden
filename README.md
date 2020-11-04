# Kubernetes deployment via Helm
[![Build Status](https://travis-ci.com/Skeen/helm-bitwarden_rs.svg?branch=master)](https://travis-ci.com/Skeen/helm-bitwarden_rs)
Helm chart to deploy a fully functional and secure [`bitwarden_rs`](https://github.com/dani-garcia/bitwarden_rs) application in [Kubernetes](https://kubernetes.io/).
 
## Requirements
Requires a Kubernetes cluster setup, with dns, storage and [Helm and Tiller](https://docs.helm.sh/) configured.

A cluster for testing, can be setup (on Ubuntu) using:
```
snap install microk8s --classic
microk8s.start
microk8s.status --wait-ready
microk8s.enable dns dashboard
microk8s.status --wait-ready
snap install helm --classic
helm init --wait
```

## Usage
The minimal deployment using all default values;
```
DOMAIN=bitwarden.yourdomain.com
helm install --wait --set "ingress.hosts={$DOMAIN},ingress.tls[0].hosts={$DOMAIN},ingress.tls[0].secretName=bitwarden-tls-secret" .
```
This will setup `bitwarden_rs` with a persistent storage and a backup volume, with backups being shot at 3:00 every night.

HTTPS certificates will automatically be generated using [Let's Encrypt](https://letsencrypt.org/) and HTTPS will be terminated at the Ingress Controller.
*This assumes that a [Kubernetes NGINX Ingress Controller](https://github.com/kubernetes/ingress-nginx) is running, and that [cert-manager](https://github.com/jetstack/cert-manager) has been set up and configured. See [here (ingress)](https://github.com/icicimov/kubernetes-bitwarden_rs#nginx-proxy) and [here (cert-manager)](https://github.com/icicimov/kubernetes-bitwarden_rs#lets-encrypt) for examples on how to set either up.*

If you do not have Ingress and cert-manager set up, you can disable ingressing completely, by deploying with:
```
helm install --wait --set "ingress.enabled=false"
```
You will however need another way to access the bitwarden pod, for instance via port-forwarding:
```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=bitwarden" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 31111:80
```

# Removal
To remove the installation run:
```
helm delete --purge $(helm list | grep "bitwarden" | cut -f1)
```


## Configuration
Several konfiguration options are available, they can be seen in [`values.yaml`](https://github.com/Skeen/helm-bitwarden_rs/blob/master/values.yaml), and override like above using `--set` or using `--values`, see more [here](https://docs.helm.sh/helm/#helm-install).

## Screenshot

![bitwarden_rs](bitwarden.png "bitwarden_rs")
