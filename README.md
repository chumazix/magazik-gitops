Magazik GitOps — ArgoCD-конфигурация для интернет-магазина

Этот репозиторий содержит все манифесты и Helm-чарты для развёртывания проекта Magazik в Kubernetes через GitOps (ArgoCD). Здесь описывается желаемое состояние кластера: инфраструктура (PostgreSQL, Redis, Kafka), приложения (backend, frontend) и вспомогательные сервисы (мониторинг, логирование).

Структура репозитория

magazik-gitops/
├── infrastructure/ # Инфраструктурные сервисы
│ ├── namespaces/ # Неймспейсы (plain YAML)
│ ├── postgres/ # PostgreSQL (plain YAML)
│ ├── redis/ # Redis (plain YAML)
│ ├── kafka/ # Kafka + ZooKeeper (plain YAML)
│ ├── monitoring/ # Prometheus + Grafana (Helm, опционально)
│ └── logging/ # Loki + Promtail (Helm, опционально)
├── apps/ # Приложения
│ ├── backend/ # Backend (Helm, зависит от magazik-app/charts)
│ │ ├── staging/ # Staging окружение
│ │ └── production/ # Production окружение
│ └── frontend/ # Frontend (plain YAML)
│ └── staging/
└── argocd-apps/ # ArgoCD Application манифесты
├── namespaces.yaml
├── postgres.yaml
├── redis.yaml
├── kafka-cluster.yaml
├── magazik-backend-staging.yaml
├── magazik-frontend-staging.yaml
├── monitoring.yaml
└── logging.yaml

Применение манифестов вручную

Если вы не используете автоматический скрипт `deploy.sh` из `magazik-app`, можете применить манифесты пошагово:

1. Установите ArgoCD (если ещё нет)
```bash
Kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.8.0/manifests/install.yaml
```

2. Примените неймспейсы
```bash
kubectl apply -f argocd-apps/namespaces.yaml
argocd app sync namespaces
```

3. Примените остальные приложения
```bash
kubectl apply -f argocd-apps/postgres.yaml
kubectl apply -f argocd-apps/redis.yaml
kubectl apply -f argocd-apps/kafka-cluster.yaml
kubectl apply -f argocd-apps/magazik-backend-staging.yaml
kubectl apply -f argocd-apps/magazik-frontend-staging.yaml
kubectl apply -f argocd-apps/monitoring.yaml   # опционально
kubectl apply -f argocd-apps/logging.yaml      # опционально
```

4. Синхронизируйте все приложения
```bash
argocd app sync --name namespace
argocd app sync --all
```

Зависимости
Backend: Helm-чарт из репозитория magazik-app (charts/magazik-backend).

PostgreSQL и Redis: plain манифесты (без Helm), чтобы избежать проблем с OCI-репозиториями Bitnami.

Kafka: plain манифесты на основе wurstmeister/kafka (ZooKeeper + Kafka).

Monitoring и Logging: Helm-чарты (опционально, по умолчанию отключены в скрипте deploy.sh).

После установки ArgoCD:

kubectl port-forward svc/argocd-server -n argocd 8080:443 &
ARGOCD_PASSWORD=$(kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d)
echo "Password: $ARGOCD_PASSWORD"
Логин: admin, пароль из вывода.

Проверка состояния

argocd app list
kubectl get pods -A

Важно
Репозиторий magazik-gitops не содержит кода приложений, только декларативное описание инфраструктуры.

Для работы бэкенда необходим образ magazik-app-backend:latest, который должен быть загружен в кластер (автоматически делается в deploy.sh).

Для фронтенда — образ magazik-frontend:latest.
