# Домашнее задание к занятию «Kubernetes. Причины появления. Команда kubectl»

> Репозиторий: hw-33\
> Выполнил: Асадбек Асадбеков\
> Дата: сентябрь 2025

## Цели
- Развернуть **MicroK8S** на виртуальной машине Debian 12  
- Установить и настроить **kubectl**  
- Проверить работу кластера и Kubernetes Dashboard  

---

## Задание 1. Установка MicroK8S

### Подготовка системы
```bash
sudo apt update
sudo apt install snapd -y
```

### Установка MicroK8S
```bash
sudo snap install microk8s --classic
```

### Настройка прав доступа
```bash
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
```
> После выполнения потребовался **перезапуск SSH-сессии**

### Проверка статуса кластера
```bash
microk8s status --wait-ready
```

### Включение аддонов
```bash
microk8s enable dashboard
microk8s enable metrics-server
```

### Настройка сертификатов для внешнего доступа
```bash
ip a   # определение IP-адреса VM

sudo nano /var/snap/microk8s/current/certs/csr.conf.template
# Добавлена строка:
IP.4 = 10.0.2.15
```

### Обновление сертификатов
```bash
sudo /snap/bin/microk8s refresh-certs --cert front-proxy-client.crt
```

**Результат:** MicroK8S успешно установлен и настроен. Кластер готов к работе.  

---

## Задание 2. Установка и настройка kubectl

### Установка kubectl
```bash
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

### Настройка конфигурации
```bash
microk8s config > ~/.kube/config
```

### Проверка подключения
```bash
kubectl get nodes
```

### Вывод:
```
NAME   STATUS   ROLES    AGE   VERSION
asad   Ready    <none>   40m   v1.32.3
```

### Автодополнение
```bash
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

### Доступ к Dashboard
```bash
kubectl port-forward -n kube-system service/kubernetes-dashboard 10443:443 --address 0.0.0.0
kubectl -n kube-system create token default
```

**Результат:** kubectl успешно установлен и подключен к кластеру.  

---

## Создание тестового окружения

### Namespace
```bash
kubectl create namespace test-dashboard
```

### Тестовое приложение
```bash
kubectl create deployment nginx-test --image=nginx -n test-dashboard
kubectl create service clusterip nginx-test --tcp=80:80 -n test-dashboard
```

### Проверка развертывания
```bash
kubectl get all -n test-dashboard
```

### Вывод:
```
NAME                             READY   STATUS    RESTARTS   AGE
pod/nginx-test-b548755db-k2zlh   1/1     Running   0          7m8s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/nginx-test   ClusterIP   10.152.183.157   <none>        80/TCP    6m58s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-test   1/1     1            1           7m8s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-test-b548755db   1         1         1       7m8s
```

### Проверка логов
```bash
kubectl logs nginx-test-b548755db-k2zlh -n test-dashboard
```

---

## Kubernetes Dashboard

- **URL:** `https://10.0.2.15:10443`  
- **Метод аутентификации:** Token-based  
- Получение токена:
  ```bash
  kubectl -n kube-system create token default
  ```

**Наблюдения:**
- Dashboard успешно запускается и доступен  
- При первом открытии вышло сообщение *"There is nothing to display here"*  
- После создания ресурсов данные должны появится  

---

## Общие выводы

- MicroK8S успешно установлен и настроен на VM c Debian 12  
- Kubernetes кластер функционирует корректно  
- Dashboard доступен и готов к использованию  
- kubectl корректно работает с кластером  

---

## Проблемы и решения
- Потребовалось обновление **snapd** для совместимости  
- Для некоторых команд требовался полный путь к `microk8s`  

---

## Скриншоты
- ![Интерфейс Kubernetes Dashboard](https://github.com/asad-bekov/hw-34/blob/main/img/1.PNG)
- ![Вывод `kubectl get all -n test-dashboard`](https://github.com/asad-bekov/hw-34/blob/main/img/2.PNG)
- ![Вывод `kubectl get nodes`](https://github.com/asad-bekov/hw-34/blob/main/img/3.PNG) 
  
---

