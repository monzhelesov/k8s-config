# K8s Config

Конфигурация Kubernetes кластера для проекта StatusBoard.

## Структура

    namespaces/       - Namespace для приложения и мониторинга
    app/
        api/          - Deployment + Service для API
        frontend/     - Deployment + Service для Frontend
    monitoring/       - kube-prometheus stack
    ingress/          - Ingress манифесты

## Применение манифестов

    kubectl apply -f namespaces/
    kubectl apply -f app/api/
    kubectl apply -f app/frontend/
    kubectl apply -f ingress/

## Мониторинг

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm upgrade --install monitoring prometheus-community/kube-prometheus-stack \
      --namespace monitoring \
      --set grafana.service.type=ClusterIP \
      --set grafana.adminPassword=admin123

## Ingress

    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
      --namespace ingress-nginx \
      --create-namespace \
      --set controller.service.type=LoadBalancer

После получения внешнего IP:
- Приложение: http://IP/
- Grafana: http://IP/grafana
