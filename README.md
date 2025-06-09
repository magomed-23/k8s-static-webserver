# Nginx Static Webserver in Kubernetes

Минималистичный Kubernetes-проект: веб-сервер на Nginx, отдающий HTML-страницу из `ConfigMap` и защищённый HTTP Basic-аутентификацией через `Secret`.

## Состав

| Файл               | Назначение                                 |
|--------------------|--------------------------------------------|
| `config-map.yaml`  | содержит HTML и `nginx.conf`               |
| `secret.yaml`      | содержит `htpasswd` файл                   |
| `deployment.yaml`  | описание Pod'а с volumeMount'ами           |
| `service.yaml`     | NodePort для доступа снаружи               |

## Запуск

> Требуется: установленный Kubernetes (например, Minikube) и `kubectl`.

1. Применить все манифесты:
    ```bash
    kubectl apply -f config-map.yaml
    kubectl apply -f secret.yaml
    kubectl apply -f deployment.yaml
    kubectl apply -f service.yaml
    ```

2. Найти IP-адрес minikube:
    ```bash
    minikube ip
    ```

3. Проверить доступ с авторизацией:
    ```bash
    curl -u <логин>:<пароль> http://<minikube-ip>:30080
    ```

    Пример:
    ```bash
    curl -u admin:qwerty123 http://192.168.49.2:30080
    ```

    В случае успеха вы увидите HTML-страницу из `ConfigMap`. При неверном логине/пароле — код `401 Unauthorized`.

## Как сгенерирован пароль

1. Сгенерируйте строку с `htpasswd`:
    ```bash
    htpasswd -nb admin qwerty123
    ```

    Пример результата:
    ```
    admin:$apr1$...$...
    ```

2. Закодируйте строку в base64:
    ```bash
    echo -n 'admin:$apr1$...$...' | base64
    ```

3. Добавьте в `secret.yaml`:
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: nginx-secret
    type: Opaque
    data:
      auth: <base64-строка>
    ```
