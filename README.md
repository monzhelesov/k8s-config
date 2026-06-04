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

kube-prometheus stack — Prometheus, Grafana, Alertmanager, node-exporter:

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update
    helm upgrade --install monitoring prometheus-community/kube-prometheus-stack \
      --namespace monitoring \
      --set grafana.service.type=ClusterIP \
      --set grafana.adminPassword=admin123 \
      --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false

## Ingress

Один внешний IP для всех сервисов:

    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    helm repo update
    helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
      --namespace ingress-nginx \
      --create-namespace \
      --set controller.service.type=LoadBalancer

После получения IP:
- Приложение: http://IP/
- Grafana: http://IP/grafana (admin / admin123)
