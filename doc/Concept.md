# Порівняльний аналіз інструментів для локального Kubernetes: Minikube, kind, k3d

## Вступ
Для розробки та тестування системи ascii-art на основі Machine Learning команда AsciiArtify порівняла три популярні інструменти для розгортання локальних Kubernetes-кластерів: **Minikube**, **kind** (Kubernetes IN Docker) та **k3d**.  
Усі демо і гіфки з прикладами роботи знаходяться у папці [`doc/`](./doc).

---

## Характеристики інструментів

### 1. Minikube
- **Підтримка ОС:** Windows, macOS, Linux (x86-64, ARM).
- **Автоматизація:** Легко скриптується, є підтримка запуску через CI/CD.
- **Додаткові функції:** Плагіни для моніторингу, дашборди, інтеграція з Helm, MetalLB та іншими add-on.
- **Контейнеризація:** Підтримує Docker, Podman, cri-o як runtime.

### 2. kind (Kubernetes IN Docker)
- **Підтримка ОС:** Windows, macOS, Linux (x86-64, ARM).  
- **Автоматизація:** Чудово підходить для CI/CD (тести, build-пайплайни).
- **Додаткові функції:** Легка інсталяція, багатокластерність, простий API для скриптів.
- **Контейнеризація:** Працює всередині Docker-контейнерів.

### 3. k3d
- **Підтримка ОС:** Windows, macOS, Linux (x86-64, ARM).
- **Автоматизація:** Добре інтегрується в build-системи, запуск та знищення кластерів за секунди.
- **Додаткові функції:** Швидкість старту, підтримка кількох кластерів, мінімальний overhead.
- **Контейнеризація:** На базі Rancher K3s, запускається в Docker.

---

## Таблиця: Порівняння інструментів

| Інструмент | Демо | Переваги | Недоліки |
|------------|------|----------|----------|
| Minikube | ![minikube.gif](./minikube.gif) | Легко встановити, підтримує різні драйвери та add-on, максимальна схожість до продуктивного Kubernetes | Вимагає більше ресурсів, не ідеально для CI, де важлива швидкість |
| kind      | ![kind.gif](./kind.gif)         | Дуже простий, ідеальний для CI/CD, легкий запуск кількох кластерів | Менше можливостей моніторингу, залежність від Docker |
| k3d       | ![k3d.gif](./k3d.gif)           | Миттєвий старт, мінімальні ресурси, ідеально для PoC | Не всі production-фічі Kubernetes, залежність від Docker |

---

## Висновки

- **Minikube** — найкраще для повноцінної емуляції production-кластера локально, якщо є запас ресурсів.
- **kind** — оптимально для швидких тестів, CI/CD пайплайнів та сценаріїв, де важлива простота і швидкість.
- **k3d** — наш вибір для PoC AsciiArtify: надшвидкий старт, мінімум налаштувань, відмінна інтеграція з сучасними dev workflow.

**Рекомендація:**  
Для локального прототипування та PoC — **k3d**.  
Для інтеграції в складніші розробки та емуляції enterprise — розглядати **Minikube**.  
Для автоматизації та CI/CD — **kind**.

---

## Практичний гайд: Як розгорнути демо "Hello from Ascii" у кожному кластері

> **Файл для розгортання:** [ascii-artify.yaml](./ascii-artify.yaml)  
> Примітка: yaml використовує простий імейдж, який повертає відповідь `Hello from ascii`.

### 1. Minikube

#### Встановлення:
```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube && sudo mv minikube /usr/local/bin
minikube start
```
#### Деплой:
```bash
kubectl apply -f ascii-artify.yaml
kubectl get pods,svc
kubectl port-forward svc/ascii-artify 8081:80 &
```

#### Перевірка:
```bash
curl localhost:8081
kubectl logs $(kubectl get pods -l app=ascii-artify -o name)
```

#### Видалити:
```bash
minikube delete
```

### 2. Kind

#### Встановлення:
```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
chmod +x kind && sudo mv kind /usr/local/bin
kind create cluster --name asciiartify
```

#### Деплой:
```bash
kubectl apply -f ascii-artify.yaml
kubectl get pods,svc
kubectl port-forward svc/ascii-artify 8081:80 &
```

#### Перевірка:
```bash
curl localhost:8081
kubectl logs $(kubectl get pods -l app=ascii-artify -o name)
```

#### Видалити:
```bash
kind delete cluster --name asciiartify
```

### 2. k3d

#### Встановлення:
```bash
curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
k3d cluster create k3d-asciiartify
```

#### Деплой:
```bash
kubectl apply -f ascii-artify.yaml
kubectl get pods,svc
kubectl port-forward svc/ascii-artify 8081:80 &
```

#### Перевірка:
```bash
curl localhost:8081
kubectl logs $(kubectl get pods -l app=ascii-artify -o name)
```

#### Видалити:
```bash
k3d cluster delete k3d-asciiartify
```