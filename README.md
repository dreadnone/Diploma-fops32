# Дипломный практикум в Yandex.Cloud

## Этапы выполнения

### Создание облачной инфраструктуры

Инфраструктура создана с помощью Terraform. Включает VPC с 3 подсетями в зонах доступности, группу безопасности, Managed Kubernetes кластер и Container Registry.

<img width="1640" height="408" alt="Снимок экрана (38)" src="https://github.com/user-attachments/assets/af0dddd6-6638-4556-bdcd-55b0d208fa83" />
<img width="1672" height="419" alt="Снимок экрана (35)" src="https://github.com/user-attachments/assets/b3994ebe-3138-4ffb-9470-c4565453f1d3" />
<img width="1668" height="281" alt="Снимок экрана (36)" src="https://github.com/user-attachments/assets/3cb34ed3-9a49-4403-bcec-c287ffc823b2" />
<img width="682" height="296" alt="Снимок экрана (28)" src="https://github.com/user-attachments/assets/8f03b84c-041f-4835-8ed4-a28b035dd0cc" />



---

### Создание Kubernetes кластера

Использован Managed Kubernetes (zonal мастер) с группой из 2 preemptible нод.

<img width="1898" height="891" alt="Снимок экрана (34)" src="https://github.com/user-attachments/assets/7c91b8bb-03ff-44ba-8f76-76a312c7c66f" />
<img width="1330" height="319" alt="Снимок экрана (33)" src="https://github.com/user-attachments/assets/42451c92-6338-4447-9ebf-d381c72a1e4b" />

<img width="1637" height="322" alt="Снимок экрана (39)" src="https://github.com/user-attachments/assets/9539f74c-c7fa-4eff-b795-0df4e24e5030" />
*Managed K8s API недоступен из внешних сетей из-за ограничений провайдера. Для управления кластером использован Bastion Host в той же подсети, что решает проблему сетевой доступности и является стандартной практикой для защищённого доступа к закрытым ресурсам.*

---

### Создание тестового приложения

Тестовое приложение — Nginx на Alpine, отдающий статическую HTML-страницу. Docker образ хранится в Yandex Container Registry.

**Репозиторий:** [diploma-devops-app](https://github.com/dreadnone/diploma-devops-app)  
**Docker образ:** `cr.yandex/crp54bul7miqfdk5it8q/diploma-app:v2.0.0`

<img width="1614" height="884" alt="Снимок экрана (37)" src="https://github.com/user-attachments/assets/82900943-fac4-411c-b3a2-43de3116fe24" />
**Репозиторий с Kubernetes манифестами:** [diploma-k8s-config](https://github.com/dreadnone/diploma-k8s-config)

---

### Подготовка системы мониторинга и деплой приложения

Установлен kube-prometheus-stack (Prometheus + Grafana + Alertmanager + Node Exporter).  
Приложение задеплоено через NodePort.

<img width="1906" height="881" alt="Снимок экрана (26)" src="https://github.com/user-attachments/assets/76344d0e-f6e7-4ed4-8a3d-1b5f11b4e994" />
<img width="1912" height="891" alt="Снимок экрана (25)" src="https://github.com/user-attachments/assets/d6067ba3-778c-448d-bce5-f0b67ed5af51" />
<img width="1915" height="912" alt="Снимок экрана (24)" src="https://github.com/user-attachments/assets/c907bf44-3052-466b-bb3e-c35ba5f145df" />
<img width="1920" height="777" alt="Снимок экрана (23)" src="https://github.com/user-attachments/assets/8bf5fcef-26cf-4dec-80c6-844d0d17261b" />
<img width="1920" height="899" alt="Снимок экрана (22)" src="https://github.com/user-attachments/assets/7c76b15a-a02c-4b3c-9f9a-faaa825157f1" />

**Доступы:**

| Сервис | URL | Логин | Пароль |
|--------|-----|-------|--------|
| Приложение | http://111.88.253.31:31275 | — | — |
| Grafana | http://111.88.249.56:31816 | admin | Diploma2026 |

---

### Деплой инфраструктуры в terraform pipeline

Выбран вариант 3: автоматический запуск Terraform из Git-репозитория через CI/CD систему.

Инфраструктура создаётся одной командой из репозитория [diploma-terraform](https://github.com/dreadnone/diploma-terraform).

<img width="682" height="296" alt="Снимок экрана (28)" src="https://github.com/user-attachments/assets/8f03b84c-041f-4835-8ed4-a28b035dd0cc" />

---

### Установка и настройка CI/CD

CI/CD реализован на GitHub Actions.

**Pipeline:**
- При пуше в `main` → сборка Docker образа и пуш в Container Registry с тегом `latest`
- При создании тега (v*) → сборка с версией и автоматический деплой в Kubernetes кластер

**GitHub Actions:**
<img width="1526" height="891" alt="Снимок экрана (21)" src="https://github.com/user-attachments/assets/bc42b535-9cf8-4109-b410-2e7858c7bc1f" />

**Приложение v1.0.0 (до CI/CD):**
<img width="1920" height="370" alt="Снимок экрана (19)" src="https://github.com/user-attachments/assets/278b4610-6da4-4411-8e45-f0aa536eecb7" />

**Приложение v2.0.0 (после CI/CD):**
<img width="1920" height="454" alt="Снимок экрана (20)" src="https://github.com/user-attachments/assets/a3c826a0-cb8d-4398-b04f-967e62e15593" />

**Container Registry с историей всех сборок:**
<img width="1614" height="884" alt="Снимок экрана (37)" src="https://github.com/user-attachments/assets/82900943-fac4-411c-b3a2-43de3116fe24" />

**Пример запуска деплоя:**
```bash
git tag v2.0.0
git push origin v2.0.0




