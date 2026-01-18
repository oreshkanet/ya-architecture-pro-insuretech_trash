# Настройка Rate Limiting

[nginx.conf](./nginx.conf)

```yaml
http {
    # Настройка зоны для ограничения запросов
    limit_req_zone $binary_remote_addr zone=req_limit:10m rate=10r/m;
    limit_req_status 429;

    # Настройка upstream для балансировки нагрузки
    upstream backend_servers {
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
    }

    server {
        listen 80;

        location / {
            limit_req zone=req_limit burst=9 nodelay;
            
            proxy_pass http://backend_servers;
        }

    }
} 
```

Чтобы ограничить количество запросов до 10 в минуту на одного клиента и возвращать ошибку `429 Too Many Requests` при превышении лимита, необходимо использовать модуль `ngx_http_limit_req_module`, который входит в стандартную поставку Nginx:

- `limit_req_zone $binary_remote_addr zone=perip:10m rate=10r/m` — создает зону памяти `perip` размером 10 МБ для хранения состояния клиентов (по IP), ограничивая их до 10 запросов в минуту (`rate=10r/m`).
- `limit_req zone=perip burst=0 nodelay` — применяет ограничение без буферизации (`burst`) и без задержек (`nodelay`), то есть любой 11-й запрос в течение минуты сразу получит 429.
- `limit_req_status 429` - устанавливает статус, который нужно вернуть при превышении лимита.