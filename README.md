# Kubernetes deployment via Helm
Helm chart to deploy a fully functional and secure [`vaultwarden`](https://github.com/dani-garcia/vaultwarden) application in [Kubernetes](https://kubernetes.io/).

Based on: https://github.com/Skeen/helm-bitwarden_rs

## Requirements
Requires a Kubernetes cluster setup, with DNS and persistent storage configured.

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
helm install --wait --set "ingress.domain=bitwarden.yourdomain.com" .
```
This will setup `vaultwarden` with a persistent storage and a backup volume, with backups being shot at 3:00 every night.

HTTPS certificates will automatically be generated using [Let's Encrypt](https://letsencrypt.org/) and HTTPS will be terminated at the Ingress Controller.
*This assumes that a [Kubernetes NGINX Ingress Controller](https://github.com/kubernetes/ingress-nginx) is running, and that [cert-manager](https://github.com/jetstack/cert-manager) has been set up and configured.*

If you do not have Ingress and cert-manager set up, you can disable ingressing completely, by deploying with:
```
helm install --wait --set "ingress.enabled=false"
```
You will however need another way to access the bitwarden pod, for instance via port-forwarding:
```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=vaultwarden" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 31111:80
```

# Removal
To remove the installation run:
```
helm delete --purge $(helm list | grep "vaultwarden" | cut -f1)
```


## Configuration
Several configuration options are available, they can be seen in [`values.yaml`](./values.yaml), and override like above using `--set` or using `--values`, see more [here](https://docs.helm.sh/helm/#helm-install).
