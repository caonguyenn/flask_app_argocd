# flask_app_argocd


# Install argo tool
# Detect OS
ARGO_OS="darwin"
if [[ uname -s != "Darwin" ]]; then
  ARGO_OS="linux"
fi
# Download the binary
curl -sLO "https://github.com/argoproj/argo-workflows/releases/download/v3.6.5/argo-$ARGO_OS-amd64.gz"

# Unzip
gunzip "argo-$ARGO_OS-amd64.gz"

# Make binary executable
chmod +x "argo-$ARGO_OS-amd64"

# Move binary to path
mv "./argo-$ARGO_OS-amd64" /usr/local/bin/argo

# Test installation
argo version

# Install Argo workflow
kubectl create namespace argo
kubectl apply -n argo -f https://github.com/argoproj/argo-workflows/releases/latest/download/install.yaml


# install Argo Events - The Event-Based Dependency Manager for Kubernetes
https://argoproj.github.io/argo-events/installation/

kubectl create namespace argo-events

kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/manifests/install.yaml
# Install with a validating admission controller
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/manifests/install-validating-webhook.yaml
# Deploy the eventbus.
kubectl apply -n argo-events -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/eventbus/native.yaml


# Create docker config secret
kubectl create secret generic docker-config --from-file=config.json=$HOME/.docker/config.json

# Create github secret
kubectl create secret generic github-secret \
  --from-literal=secret=mygithubwebhooksecret \
  -n argo-events

# Create and config argo workflow 
# Apply all yaml files on argo-workflow folder

# Test github-event
SECRET="mysecr^C123"RET="mysecret123"
DATA='{"ref": "refs/heads/master", "repository": {"name": "flask_app_argocd"}, "pusher": {"name": "tester"}}'
SIG=$(echo -n "$DATA" | openssl dgst -sha1 -hmac "$SECRET" | sed 's/^.* //')

curl -X POST   http://<EXTERNAL-IP>:12000/push   -H "Content-Type: application/json"   -H "X-Hub-Signature: sha1=$SIG"   -d "$DATA"