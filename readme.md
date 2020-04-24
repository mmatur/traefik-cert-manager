# Traefik v2 + cert-manager

## Install the kubernetes cluster in GCP US region

```bash
export CLUSTER_NAME="cluster-traefik-v2"

gcloud container clusters create "${CLUSTER_NAME}" \
  --zone="us-west1-a" \
  --project="${GCLOUD_PROJECT}"
```

## Install traefik v2

```
helm repo add traefik https://containous.github.io/traefik-helm-chart
helm repo update

kubectl create namespace traefik
helm install --namespace traefik traefik traefik/traefik --values traefik/values.yaml
```

## Access to the dashboard

```
kubectl port-forward -n traefik $(kubectl get pods -n traefik --selector "app.kubernetes.io/name=traefik" --output=name) 9000:9000
```

http://127.0.0.1:9000/dashboard/

## Install whoami

```
kubectl apply -f whoami/
```

## Install cert-manager

```
# Install the CustomResourceDefinition resources separately
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.11/deploy/manifests/00-crds.yaml

# Create the namespace for cert-manager
kubectl create namespace cert-manager

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v0.14.2
```

- Verifying the installation

```
kubectl get pods --namespace cert-manager
```

- Create cluster issuer + certificate for `whoami.cert.containous.cloud`and `powpow.cert.containous.cloud`

```
kubectl apply -f cert-manager/
```

- Check that the certificate has been generated

```
kubectl describe certificate -n whoami whoami-cert
kubectl describe certificate -n whoami powpow-cert
```

- Check the certificate issuer with the command:

```
echo | openssl s_client -showcerts -servername whoami.cert.containous.cloud -connect whoami.cert.containous.cloud:443 2>/dev/null | openssl x509 -inform pem -text | grep 'Issuer'
```


## Cleanup

``````bash
export CLUSTER_NAME="cluster-traefik-v2"

gcloud container clusters delete "${CLUSTER_NAME}" \
  --zone="us-west1-a" \
  --project="${GCLOUD_PROJECT}"
```
