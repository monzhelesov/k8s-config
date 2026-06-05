# K8s Config — StatusBoard

Kubernetes манифесты для проекта StatusBoard.

## Структура

    namespaces/
    ├── app.yaml          — namespace statusboard
    └── monitoring.yaml   — namespace monitoring
    app/
    ├── api/
    │   ├── deployment.yaml   — 2 реплики, liveness/readiness probes, Prometheus аннотации
    │   └── service.yaml      — ClusterIP
    └── frontend/
        ├── deployment.yaml   — 2 реплики
        └── service.yaml      — ClusterIP
    ingress/
    ├── ingress.yaml          — роутинг на frontend и api
    └── grafana-ingress.yaml  — роутинг на Grafana

## Применение

    kubectl apply -f namespaces/
    kubectl apply -f app/api/
    kubectl apply -f app/frontend/
    kubectl apply -f ingress/

## Мониторинг

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update
    helm upgrade --install monitoring prometheus-community/kube-prometheus-stack \
      --namespace monitoring \
      --set grafana.service.type=ClusterIP \
      --set grafana.adminPassword=admin123 \
      --set grafana.grafana\\.ini.server.domain=<INGRESS_IP> \
      --set grafana.grafana\\.ini.server.root_url="%(protocol)s://%(domain)s/grafana/" \
      --set grafana.grafana\\.ini.server.serve_from_sub_path=true \
      --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false

## Ingress

    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    helm repo update
    helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
      --namespace ingress-nginx \
      --create-namespace \
      --set controller.service.type=LoadBalancer

После получения IP:
- Приложение: http://<INGRESS_IP>/
- Grafana: http://<INGRESS_IP>/grafana (admin / admin123)
