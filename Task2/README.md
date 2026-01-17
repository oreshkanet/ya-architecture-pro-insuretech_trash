# Динамическое масштабирование контейнеров

[Образ готового приложения](https://github.com/yandex-practicum/scaletestapp/pkgs/container/scaletestapp)

```sh
docker pull ghcr.io/yandex-practicum/scaletestapp:sha256-eff20ae3ae2d596375f9ed6d612a78d149a35a66cd2907ea90d7175ca918c993.sig
```

Методы приложения доступны по порту 8080:
`GET /` — получение идентификатора пода;
`GET /metrics` — получение метрик в формате Prometheus.

Метрика `http_requests_total` возвращает количество запросов для метода получения идентификатора пода.

Разворачивание системы и настройка масштабирования выполняется в Managed Service for Kubernetes в Yandex Cloud в отдельном неймспейсе `insure-tech`.

```sh
# Создание неймспейса
kubectl create namespace insure-tech
```

## Динамическая маршрутизация на основании показателей утилизации памяти

### Deployment

[Deployment.yaml](./deployment.yaml)

```sh
# Деплоймент приложения
kubectl apply -f deployment.yaml
```

![deployment-1](./assets/deployment-1.png)

![pod-1](./assets/pod-1.png)

### Service

[Service.yaml](./service.yaml)

```sh
# Запуск сервиса
kubectl apply -f service.yaml
```

![service-1](./assets/service-1.png)

### Ingress

[ingress.yaml](./ingress.yaml)

```sh
# Ингресс для доступа к API сервиса
kubectl apply -f ingress.yaml
```

![ingress-1](./assets/ingress-1.png)

### Тестирование сервиса

Запрос `http://insure.tech/`:
![request-1](./assets/request-1.png)

Метрики `http://insure.tech/metrics`:
![metrics-1](./assets/metrics-1.png)

### Horizontal Pod Autoscaler

```sh
# Установка HPA для приложения
kubectl apply -f hpa.yaml

# Проверка состояния HPA
kubectl get hpa -n insure-tech
```

![hpa-1](./assets/hpa-1.png)

![hpa-2](./assets/hpa-2.png)

### Locust

[locustfile](./locustfile.py)

```sh
locust
```

Результаты запуска locust:
![locust-1](./assets/locust-1.png)

![locust-2](./assets/locust-2.png)

### Тестирование возрастания потребления памяти

Превышение порога памяти:
![hpatest-1](./assets/hpatest-1.png)

Увеличение количества подов до 3х:
![hpatest-2](./assets/hpatest-2.png)

История горизонтального масштабирования:
![hpatest-3](./assets/hpatest-3.png)

В итоге 4 пода:
![hpatest-4](./assets/hpatest-4.png)

### Тестирование уменьшения потребления памяти

После отключения нагрузки Locust происходит плавное снижение затрат памяти:
![hpatest-5](./assets/hpatest-5.png)

Количество подов сокращается:
![hpatest-6](./assets/hpatest-6.png)

История HPA:
![hpatest-7](./assets/hpatest-7.png)

## Динамическая маршрутизация на основании показателей количества запросов в секунду

### Установка Prometheus через Helm

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update

kubectl create namespace monitoring

helm install prometheus-stack prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
```

Устанавливаем адаптер для кастомной метрики:

```sh
helm install prometheus-adapter prometheus-community/prometheus-adapter -f values.yaml --namespace monitoring

helm upgrade prometheus-adapter prometheus-community/prometheus-adapter -f values.yaml --namespace monitoring

helm uninstall prometheus-adapter --namespace monitoring
helm delete prometheus-adapter --namespace monitoring

```

Проверяем метрику:

```sh
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1
```

![metrics-custom-1](./assets/metrics-custom-1.png)

Порт-форвардинг для Prometheus: 
```sh
kubectl port-forward -n monitoring svc/prometheus-stack-kube-prom-prometheus 9090
```

UI доступен на http://localhost:9090. Для Grafana используйте аналогично порт 80 или 3000.

### Добавление сервиса в мониторинг

[service_monitor.yaml](./service_monitor.yaml)

```sh
kubectl apply -f ./service_monitor.yaml
```

### HPA по RPS

```sh
kubectl apply -f ./hpa-rps.yaml
```

![hparpstest-1](./assets/hparpstest-1.png)

### Тестирование HPA по RPS

В locust запущены запросы:
![hparpstest-3](./assets/hparpstest-3.png)

Количество подов начинает расти:
![hparpstest-2](./assets/hparpstest-2.png)
![hparpstest-4](./assets/hparpstest-4.png)
![hparpstest-5](./assets/hparpstest-5.png)

В locust уменьшаем количество запросов:
![hparpstest-6](./assets/hparpstest-6.png)

Количество подов постепенно начинает сокращаться:
![hparpstest-7](./assets/hparpstest-7.png)
![hparpstest-8](./assets/hparpstest-8.png)

История HPA:
![hparpstest-9](./assets/hparpstest-9.png)


