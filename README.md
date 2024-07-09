# This is an implementation of prometheus scraping and exporting metrics from a single ns on k8s and grafana with an example dashboard spewing them

```
wget https://get.helm.sh/helm-v3.14.4-linux-amd64.tar.gz
tar -zxvf helm-v3.14.4-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
export NAMESPACE=<namespacenamethatisthenamespacenameofthenamespacethatyouwanttonameyournamespacewiththatname>
alias k=kubectl
alias kk="kubectl -n $NAMESPACE"

kubectl create namespace $NAMESPACE || true

git clone https://github.com/nashpaz123/grafanaprom_single_ns.git
cd grafanaprom_single_ns

sed -i "s/app-namespace/$NAMESPACE/g" servicemonitor.yaml
sed -i "s/app-namespace/$NAMESPACE/g" prometheus-values.yaml
sed -i "s/app-namespace/$NAMESPACE/g" grafana-values.yaml

kubectl apply -f servicemonitor.yaml
helm install prometheus-$NAMESPACE prometheus-community/prometheus --namespace $NAMESPACE --version 25.22.1 --values prometheus-values.yaml
helm install grafana grafana/grafana --namespace $NAMESPACE --version 8.3.2 --values grafana-values.yaml

#prom takes a minute to load the server and another two minutes to load pod data
#kubectl expose deployment prometheus-server -n $NAMESPACE --type=NodePort --name promnport && kubectl get svc -n $NAMESPACE |grep promnport
#kubectl expose deployment grafana -n $NAMESPACE --type=NodePort --name grafananport && kubectl get svc -n $NAMESPACE |grep grafananport
#a simple query with a specific ns can be set e.g: sum(rate(container_cpu_usage_seconds_total{namespace="kube-system"}[5m])) by (pod) , the example shows all pods available from prometheus
```
