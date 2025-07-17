# Проєкт: lesson-8-9-JenkinsCI-ArgoCD  
## Тема: CI/CD з Jenkins, ECR, Helm, Terraform та Argo CD

Цей проєкт реалізує повноцінний DevOps-процес розгортання застосунку в кластер Kubernetes з використанням підходу GitOps.

CI/CD побудований за допомогою таких інструментів:
- Jenkins для збірки та публікації Docker-образу
- Amazon ECR для зберігання образів
- Helm Chart для опису розгортання
- Argo CD для автоматичної синхронізації з Git
- Terraform для інфраструктурного коду (IaC)

Проєкт розроблений у межах курсу DevOps від GoIT.

---

## 📂 Структура проєкту

- `Jenkinsfile` — опис CI-процесу
- `helm/app` — Helm-чарт застосунку
- `django-app/` — вихідний код Django та Dockerfile
- `modules/` — модулі Terraform:
  - `vpc`, `ecr`, `s3-backend`, `eks`, `jenkins`, `argocd`
- `main.tf`, `outputs.tf`, `variables.tf` — основні конфігураційні файли Terraform

---

## 🧰 Використані технології

- **Terraform** — для автоматичного створення інфраструктури (VPC, ECR, EKS, Jenkins, Argo CD)
- **Helm** — для управління релізами Jenkins і Argo CD, а також застосунку
- **Docker** — для створення контейнерного образу Django-застосунку
- **Amazon ECR** — як реєстр контейнерів
- **Jenkins** — для CI: збірка образу, пуш у ECR, оновлення Helm-чарту
- **Argo CD** — для автоматичного CD: спостереження за змінами у Git і деплой у кластер
- **Kubernetes (EKS)** — цільове середовище для деплойменту

---

## ✅ Що реалізовано

- Розгорнуто інфраструктуру AWS за допомогою Terraform:
  - S3 + DynamoDB для зберігання стану
  - VPC з публічними/приватними підмережами
  - ECR для зберігання образів
  - EKS кластер
  - Jenkins (через Helm)
  - Argo CD (через Helm)

- Створено Jenkins pipeline, який:
  - Збирає Docker-образ із Django-застосунком
  - Публікує образ у Amazon ECR
  - Оновлює файл `values.yaml` з новим тегом образу
  - Пушить зміни у GitHub-репозиторій (гілка `lesson-8-9`)

- Налаштовано Argo CD Application:
  - Спостерігає за Helm-чартом застосунку
  - Автоматично синхронізує стан кластера з Git

---

## 📦 Як застосувати Terraform

1. Перейдіть у корінь проєкту:
```bash
cd ~/projects/lesson-8-9
```
2. Ініціалізуйте Terraform:
```bash
terraform init
```
3. Перевірте план:
```bash
terraform plan
```
4. Застосуйте інфраструктуру:
```bash
terraform apply
```
⚠️ Після завершення роботи рекомендується видалити ресурси, щоб уникнути витрат:
```bash
terraform destroy
```

---

## 🛠 Як перевірити Jenkins pipeline

1. Відкрийте інтерфейс Jenkins (адресу ви отримаєте з Terraform Outputs).
2. Увійдіть під обліковими даними адміністратора (також в Outputs або в secrets).
3. Відкрийте Jenkins job або pipeline.
4. Запустіть збірку вручну або через push у репозиторій.
5. Переконайтесь, що виконуються наступні етапи:
   - Збірка Docker-образу
   - Пуш в Amazon ECR
   - Оновлення Helm values.yaml (тег образу)
   - Push у GitHub (гілка `lesson-8-9`)

---

## 🚀 Як перевірити результат в Argo CD

1. Відкрийте веб-інтерфейс Argo CD (адресу можна отримати з Terraform Outputs або через команду `kubectl get svc`).
2. Увійдіть за допомогою логіну `admin` та пароля з Helm values або secrets.
3. Перейдіть до вкладки **Applications**.
4. Знайдіть застосунок `lesson-8-app` і переконайтесь, що:
   - Статус: `Synced ✅`
   - Стан: `Healthy ✅` або `Progressing` (на початку)
5. Клацніть на застосунок, щоб переглянути ресурси (Deployment, Service, Pod).
6. Перевірте, що Argo CD автоматично оновлює кластер після змін у Git.

---

## 👨‍💻 Автор

**Олександр Ребенок**  

GitHub: [999Ralex999](https://github.com/999Ralex999)

