# 📦 Проєкт: lesson-10-db-module

## 🧩 Тема: RDS + Aurora через Terraform-модуль

Цей проєкт демонструє створення універсального Terraform-модуля для розгортання бази даних PostgreSQL або Aurora PostgreSQL в AWS.  
DevOps-інженер може легко перемикатись між звичайною RDS або Aurora Cluster за допомогою змінної `use_aurora`.

---

## 📂 Структура проєкту

- `main.tf`, `variables.tf`, `outputs.tf` — основні конфігураційні файли Terraform  
- `backend.tf.disabled` — резервне зберігання стану (S3 + DynamoDB)
- `modules/` — модулі інфраструктури:
  - `vpc/` — VPC з сабнетами
  - `s3-backend/` — S3 бакет і DynamoDB для бекенду
  - `rds/` — універсальний модуль для RDS/Aurora
- `tfplan` — збережений план застосування

---

## 🛠️ Використані технології

- **Terraform** — IaC для створення інфраструктури
- **AWS RDS** — керована база даних PostgreSQL
- **AWS Aurora** — масштабований кластер сумісний з PostgreSQL
- **S3 + DynamoDB** — зберігання стейтів
- **VPC** — ізольована мережа з публічними/приватними сабнетами

---

## ✅ Що реалізовано

- Універсальний модуль `rds`, який:
  - створює або RDS, або Aurora (через `use_aurora`)
  - автоматично створює `Subnet Group`, `Security Group`, `Parameter Group`
  - підтримує `Multi-AZ`, `backup`, `read replicas`
- Робота з параметрами через змінні
- Обробка виводів через `try(...)`
- Протестовано на `PostgreSQL 17.2`

---

## 🧪 Як застосувати Terraform

```bash
cd ~/lesson-10-db-module
terraform init
terraform plan -out=tfplan
terraform apply "tfplan"
```
⚠️ Після завершення роботи рекомендується видалити ресурси, щоб уникнути витрат:
```bash
terraform destroy
```
---

## 🔧 Приклад використання модуля rds

module "rds" {
  source = "./modules/rds"

  name                       = "myapp-db"
  use_aurora                 = false
  engine                     = "postgres"
  engine_version             = "17.2"
  parameter_group_family_rds = "postgres17"
  instance_class             = "db.t3.micro"
  allocated_storage          = 20
  db_name                    = "myapp"
  username                   = "postgres"
  password                   = "admin123AWS23"
  subnet_private_ids         = module.vpc.private_subnets
  subnet_public_ids          = module.vpc.public_subnets
  publicly_accessible        = true
  vpc_id                     = module.vpc.vpc_id
  multi_az                   = true
  backup_retention_period    = 7

  parameters = {
    max_connections = "200"
    log_min_duration_statement = "500"
  }

  tags = {
    Environment = "dev"
    Project     = "myapp"
  }
}

---

## 📘 Основні змінні

| Назва                         | Тип      | Опис                                                      |
|------------------------------|----------|-----------------------------------------------------------|
| `use_aurora`                 | `bool`   | Визначає, чи створювати Aurora (`true` = Aurora, `false` = RDS) |
| `engine`                     | `string` | Тип бази даних для RDS (`postgres`, `mysql`)             |
| `engine_version`             | `string` | Версія для RDS (`наприклад, 17.2`)                        |
| `engine_version_cluster`     | `string` | Версія для Aurora (`наприклад, 15.3`)                     |
| `parameter_group_family_rds`| `string` | Сімейство параметрів для RDS (`postgres17`, `mysql8.0`)  |
| `parameter_group_family_aurora` | `string` | Сімейство параметрів для Aurora (`aurora-postgresql15`)  |
| `instance_class`             | `string` | EC2-клас інстанса для БД (`db.t3.micro`, `db.t3.medium`) |
| `allocated_storage`          | `number` | Розмір в ГБ (для RDS, наприклад `20`)                    |
| `db_name`                    | `string` | Назва бази даних                                          |
| `username`                   | `string` | Ім’я адміністратора бази                                  |
| `password`                   | `string` | Пароль адміністратора                                     |
| `multi_az`                   | `bool`   | Чи розгортати резервну репліку в іншій AZ                 |
| `backup_retention_period`    | `number` | Період збереження бекапів (в днях)                        |
| `parameters`                | `map`    | Словник параметрів (`max_connections`, `log_statement`)  |
| `subnet_private_ids`         | `list`   | ID приватних сабнетів                                     |
| `subnet_public_ids`          | `list`   | ID публічних сабнетів (для `publicly_accessible = true`) |
| `vpc_id`                     | `string` | ID VPC, в якій створюється база                          |
| `publicly_accessible`        | `bool`   | Доступність з Інтернету (`false` — тільки з VPC)         |
| `tags`                       | `map`    | Мапа тегів (`Environment`, `Project` тощо)               |

---

## ✅ Результат

- 📦 **RDS інстанс створено:** `db-JWR636QDURPHV2QQ63LEZOPT3U`
- 🐘 **Використовується версія PostgreSQL:** `17.2`

### 🔎 Outputs:
```hcl
ecr_url    = "655307635386.dkr.ecr.us-west-2.amazonaws.com/lesson-7-ecr"
eks_endpoint = "https://FB576D69D7C4147868F54FCD66306FF4.gr7.us-west-2.eks.amazonaws.com"
vpc_id     = "vpc-05fd59dc45a192c6c"
```

---

## 👨‍💻 Автор

**Олександр Ребенок**  

GitHub: [999Ralex999](https://github.com/999Ralex999)
