# k8s-mycluster

## Installation

```bash
# Nginx ingress
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-0.32.0/deploy/static/provider/cloud/deploy.yaml
kubectl -n ingress-nginx patch service ingress-nginx-controller -p "{\"spec\": {\"loadBalancerIP\": \"$LOADBALANCER_IP_ADDRESS\"}}"

# Cert-manager
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.2/cert-manager.yaml

# Tekton
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.15.2/release.yaml
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/previous/v0.7.0/release.yaml
kubectl apply -f https://storage.googleapis.com/tekton-releases/dashboard/previous/v0.9.0/tekton-dashboard-release.yaml

# Argo CD
kubectl create namespace argocd
kubectl -n argocd apply -f https://raw.githubusercontent.com/argoproj/argo-cd/v1.7.6/manifests/install.yaml
# disable authentication (default password is argocd-server pod name)
kubectl -n argocd patch deployment argocd-server -p '{"spec": {"template": {"spec": {"containers": [{"name": "argocd-server", "command": ["argocd-server" ,"--staticassets" ,"/shared/app" ,"--disable-auth" ,"--insecure"]}]}}}}'

```

## Get personal access token on GitHub for Tekton pipeline

```bash
export YOUR_GITHUB_ORG_NAME=xxxx
export YOUR_GITHUB_USER=xxxxxx
export YOUR_GITHUB_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

cat << _EOF_ | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: github-credentials
  namespace: tekton-pipelines
  annotations:
    tekton.dev/git-0: https://github.com
type: kubernetes.io/basic-auth
stringData:
  username: ${YOUR_GITHUB_USER}
  password: ${YOUR_GITHUB_TOKEN}
  organization: ${YOUR_GITHUB_ORG_NAME}
_EOF_
```

## Create DockerHub credential for Tekton pipeline

```bash
export YOUR_DOCKERHUB_USER=xxxx
export YOUR_DOCKERHUB_PASSWORD=xxxxxx

cat << _EOF_ | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: dockerhub-credentials
  namespace: tekton-pipelines
  annotations:
    tekton.dev/docker-0: https://index.docker.io/v1/
type: kubernetes.io/basic-auth
stringData:
  username: ${YOUR_DOCKERHUB_USER}
  password: ${YOUR_DOCKERHUB_PASSWORD}
_EOF_
```

## Add webhook settings

```
* Payload URL: https://tekton.YOUR_DOMAIN/event-listener
* Content type: application/json
* Secret: sample-github-webhook-secret
  * if you want to change, please edit manifests/infra/tekton-triggers.yaml
* Enable SSL verification: [*]
* Let me select individual events: [*]
    * [*] Pushes
    * [*] Pull requests
* Active: [*]
```

## Reference

<https://github.com/kubernetes-native-testbed/kubernetes-native-cicd>