# K8s Config — StatusBoard

Kubernetes манифесты для проекта StatusBoard.

## Структура

    namespaces/
    ├── app.yaml          — namespace statusboard
    └── monitoring.yaml   — namespace monitoring
    app/
    ├── api/
    │   ├── deployment.yaml   — 2 реплики, liveness/readiness probes, Prometheus аннотации
    │   ├── service.yaml      — ClusterIP
    │   └── networkpolicy.yaml — разрешает трафик только от frontend и ingress
    └── frontend/
        ├── deployment.yaml   — 2 реплики
        ├── service.yaml      — ClusterIP
        └── networkpolicy.yaml — разрешает трафик только от ingress
    rbac/
    ├── service-accounts.yaml — сервисный аккаунт для приложения
    ├── roles.yaml            — роль с минимальными правами
    └── rolebindings.yaml     — привязка роли к сервисному аккаунту
    monitoring/
    └── grafana-secret.yaml   — пароль Grafana в K8s Secret
    ingress/
    ├── ingress.yaml          — роутинг на frontend и api
    └── grafana-ingress.yaml  — роутинг на Grafana

## Применение

    kubectl apply -f namespaces/
    kubectl apply -f app/api/
    kubectl apply -f app/frontend/
    kubectl apply -f rbac/
    kubectl apply -f monitoring/grafana-secret.yaml
    kubectl apply -f ingress/

## Мониторинг

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update
    helm upgrade --install monitoring prometheus-community/kube-prometheus-stack \
      --namespace monitoring \
      --set grafana.service.type=ClusterIP \
      --set grafana.admin.existingSecret=grafana-admin-secret \
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
- Grafana: http://<INGRESS_IP>/grafana

Логин Grafana — из секрета grafana-admin-secret в namespace monitoring.
