## KUBE-CLOUD (Minikube - Cloud Environment)

===============================================================

# \--- Initial Setup ---

sudo apt update \&\& sudo apt upgrade -y

# Remove unnecessary snap packages to free disk space

sudo snap remove --purge firefox
sudo snap remove --purge gnome-42-2204
sudo snap remove --purge gtk-common-themes
sudo snap remove --purge snapd

# Clean up

sudo apt clean
sudo apt autoremove -y
sudo journalctl --vacuum-size=20M
docker system prune -a -f

# Check disk space

df -h

# \--- Docker Installation ---

sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER

# Fix docker permissions after restart (used instead of newgrp)

sudo chmod 666 /var/run/docker.sock

# Verify docker works

docker ps

# \--- kubectl Installation ---

# First install curl (failed first time, needed fix)

sudo apt update --fix-missing
sudo apt install -y curl

# Install kubectl

curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify

kubectl version --client

# \--- Minikube Installation ---

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Verify

minikube version

# \--- Start Minikube ---

# First attempt failed due to disk space issues

# Start Minikube successfully

minikube start --driver=docker --memory=3000 --cpus=2

# Verify

minikube status
kubectl get nodes

# \--- Deploy Nginx VNF ---

kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort

# Verify deployment

kubectl get pods
kubectl get svc nginx

# Hello World test

minikube service nginx --url
curl http://192.168.49.2:31843

# \--- Install Helm ---

curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# \--- Install Prometheus ---

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/prometheus --set server.retention=2h --set server.global.scrape\_interval=30s

# \--- Install Grafana ---

helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana --set adminPassword=admin123 --set persistence.enabled=false

# Verify all pods running

kubectl get pods

# \--- Enable Metrics Server ---

minikube addons enable metrics-server

# Check metrics (needed to wait \~2 mins)

kubectl top nodes
kubectl top pods

# \--- Traffic Tests ---

sudo apt install -y wrk iperf3

# Ping test

ping -c 20 192.168.49.2

# wrk HTTP load tests

wrk -t2 -c10 -d30s http://192.168.49.2:31843   # Low load
wrk -t4 -c50 -d30s http://192.168.49.2:31843   # Medium load
wrk -t8 -c100 -d30s http://192.168.49.2:31843  # High load

# iperf3 test (port 5202 used as 5201 was in use)

iperf3 -s -p 5202                               # Terminal 1
iperf3 -c localhost -p 5202 -t 30              # Terminal 2

# Collect resource metrics

kubectl top nodes
kubectl top pods

# Save command history

history > \~/commands\_history.txt



===============================================================

## KUBE-EDGE (K3s - Edge Environment)

===============================================================

# \--- Initial Setup ---

sudo apt update \&\& sudo apt upgrade -y
sudo apt install -y curl

# \--- K3s Installation ---

curl -sfL https://get.k3s.io | sh -

# Verify K3s running

sudo systemctl status k3s

# Setup kubeconfig

mkdir -p \~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml \~/.kube/config
sudo chown $USER:$USER \~/.kube/config
echo "export KUBECONFIG=\~/.kube/config" >> \~/.bashrc
source \~/.bashrc

# Verify cluster

kubectl get nodes

# \--- Deploy Nginx VNF ---

kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort

# Verify deployment

kubectl get pods
kubectl get svc nginx

# Port was: 80:31942/TCP

# Hello World test

curl http://localhost:31942

# \--- Install Helm ---

curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Fix helm cache permissions (encountered permission denied error)

sudo chown -R $USER:$USER \~/.cache/helm
sudo chown -R $USER:$USER \~/.config/helm

# \--- Install Prometheus ---

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/prometheus --set server.retention=2h --set server.global.scrape\_interval=30s

# \--- Install Grafana ---

helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana --set adminPassword=admin123 --set persistence.enabled=false

# Verify all pods running

kubectl get pods

# Access Grafana

kubectl port-forward svc/grafana 3000:80

# Open browser: http://localhost:3000

# Login: admin / admin123

# \--- Collect Resource Metrics ---

kubectl top nodes
kubectl top pods

# \--- Traffic Tests ---

sudo apt install -y wrk iperf3

# Ping test

ping -c 20 localhost

# wrk HTTP load tests

wrk -t2 -c10 -d30s http://localhost:31942   # Low load
wrk -t4 -c50 -d30s http://localhost:31942   # Medium load
wrk -t8 -c100 -d30s http://localhost:31942  # High load

# iperf3 test (port 5202 used as 5201 was in use)

iperf3 -s -p 5202                           # Terminal 1
iperf3 -c localhost -p 5202 -t 30          # Terminal 2

# Save results to files

wrk -t2 -c10 -d30s http://localhost:31942 > \~/results\_edge\_low.txt
wrk -t4 -c50 -d30s http://localhost:31942 > \~/results\_edge\_medium.txt
wrk -t8 -c100 -d30s http://localhost:31942 > \~/results\_edge\_high.txt

