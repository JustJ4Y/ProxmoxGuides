# Kubernetes Testumgebung mit Find
Ist vielleicht ein wenig umständlich und verschachtelt, wegen LXC zu Docker, zu Kubernetes und verfehlt damit den Zweck von einem echten Kubernetes-Cluster, aber für Testzwecke um mal kurz was zu testen reichts

### Container erstellen:
```
LXC - Ubuntu 24.04 (Unprivileged)
```
### Auf Container:
```
apt update && apt upgrade -y
```
```
apt install curl -y
```
### Docker installieren:
Falls es später Probleme mit Docker gibt, schaut Mal in den ```DockerApparmor.ad``` Guide
```
curl -sSL https://get.docker.com | sh
```
### Kind installieren:
```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.33.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```
### Cluster erstellen:
```
kind create cluster
```
### Kubectl installieren:
```
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update && sudo apt-get install -y kubectl
```
### Checken ob alles läuft:
```
kind get clusters
```
```
kubectl cluster-info
```
### Erstes nginx Deployment testen:
```
kubectl create deployment nginx --image=nginx:latest
```
```
kubectl get deployments
kubectl get pods
kubectl rollout status deployment/nginx
```
### Service erreichbar machen:
```
kubectl expose deployment nginx \
  --port=80 \
  --target-port=80 \
  --type=ClusterIP
```
```
kubectl port-forward \
  --address LXC-IP-Adresse \
  service/nginx 80:80
```
Die nginx Landingpage sollte jetzt unter Port 80 verfügbar sein
