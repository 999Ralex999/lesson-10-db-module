# Проект: lesson-7-helm-eks

## Тема: Kubernetes, Helm та AWS EKS

---

### 🎯 Ціль

Реалізувати повний DevOps-цикл інфраструктури з Terraform, Docker, AWS та Helm:

- створити EKS-кластер через Terraform;
- зібрати Docker-образ та завантажити його у ECR;
- задеплоїти Django-застосунок через Helm;
- налаштувати HPA, ConfigMap, LoadBalancer.

---

### 📂 Структура проєкту

lesson-7/

├── aws-auth.yaml

├── backend.tf.disabled

├── django-app

│   ├── Dockerfile

│   ├── app

│   │   ├── __init__.py

│   │   ├── __pycache__

│   │   ├── asgi.py

│   │   ├── settings.py

│   │   ├── urls.py

│   │   └── wsgi.py

│   ├── manage.py

│   └── requirements.txt

├── django-chart

│   ├── Chart.yaml

│   ├── charts

│   ├── templates

│   │   ├── _helpers.tpl

│   │   ├── configmap.yaml

│   │   ├── deployment.yaml

│   │   ├── hpa.yaml

│   │   └── service.yaml

│   └── values.yaml

├── eks update-kubeconfig \

├── main.tf

├── modules

│   ├── ecr

│   │   ├── ecr.tf

│   │   ├── outputs.tf

│   │   └── variables.tf

│   ├── s3-backend

│   │   ├── dynamodb.tf

│   │   ├── outputs.tf

│   │   ├── s3.tf

│   │   └── variables.tf

│   └── vpc

│       ├── nat.tf

│       ├── outputs.tf

│       ├── routes.tf

│       ├── variables.tf

│       └── vpc.tf

├── outputs.tf

├── terraform.tfstate

├── terraform.tfstate.backup

├── .gitignore

└── README.md

### 🛠️ Етапи реалізації

#### 1. Terraform

- VPC + public/private subnets
- EKS кластер (`terraform-aws-modules/eks/aws`)
- ECR репозиторій (для зберігання образів)
- S3 + DynamoDB для бекенду Terraform

#### 2. Docker

- Скопійовано Dockerfile з проєкту теми 4
- Побудовано Docker-образ:
  ```bash
  docker build -t lesson-7-django-app:latest .
  ```
- Завантажено до AWS ECR:
  ```bash
  docker tag lesson-7-django-app:latest <ECR_URL>/lesson-7-ecr:latest
  docker push <ECR_URL>/lesson-7-ecr:latest
  ```

#### 3. Helm

- Створено чарт `django-chart`
- Підключено шаблони:
  - `deployment.yaml` з configMapRef
  - `service.yaml` типу LoadBalancer
  - `hpa.yaml` (CPU autoscaling)
  - `configmap.yaml` з env-параметрами
- Значення параметрів винесено в `values.yaml`
- Установка та оновлення:
  ```bash
  helm install django-app ./django-chart
  helm upgrade django-app ./django-chart
  ```

#### 4. Перевірка

- `kubectl get pods` → Running
- `kubectl get svc` → LoadBalancer з EXTERNAL-IP
- `kubectl get hpa` → активне масштабування

---

### ⚠️ Інцидент: kubectl доступ

> **Проблема:** після створення кластеру `kubectl` не мав доступу — помилка `The server has asked for the client to provide credentials`.

#### Причина:
- terraform-модуль EKS версії 20+ не оновлював aws-auth ConfigMap
- Kubernetes API намагався зв'язатись через `localhost:80`

#### Рішення:
- змінено версію модуля на `"19.21.0"`
- вимкнено `manage_aws_auth_configmap`
- застосовано ручний `aws-auth.yaml`:
  ```bash
  kubectl apply -f aws-auth.yaml --validate=false
  ```

Після цього `kubectl get nodes` успішно повертав статус `Ready`.

---

### ✅ Приклади команд

```bash
# Створити кластер
terraform init && terraform apply

# Підключитися до кластеру
aws eks update-kubeconfig --region us-west-2 --name lesson-7-cluster

# Зібрати образ
docker build -t lesson-7-django-app .

# Завантажити в ECR
docker push <ECR_REPO>

# Деплой Helm-чарту
helm install django-app ./django-chart
```

---

### 📌 Висновки

- Налаштовано повноцінне середовище з CI/CD-ознаками
- Вирішено критичну помилку з доступом до кластеру
- Використано Helm як production-ready інструмент деплою

---

**Автор:** 

Олександр Ребенок  

[GitHub профіль](https://github.com/999Ralex999)
