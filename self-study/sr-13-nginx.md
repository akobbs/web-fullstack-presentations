# Самостійна робота №13: Nginx як reverse proxy

**Тема:** Nginx як reverse proxy

**Мета:** Ознайомитися з веб-сервером Nginx, навчитися налаштовувати його як reverse proxy, обслуговувати статичні файли та налаштовувати HTTPS.

**Кількість годин:** 2

---

## Теоретичний матеріал

### Nginx - огляд

Nginx - високопродуктивний веб-сервер та reverse proxy. Обробляє мільйони одночасних з'єднань завдяки асинхронній подійно-орієнтованій архітектурі. Основні сценарії використання: обслуговування статичних файлів, reverse proxy, балансування навантаження, SSL termination.

### Конфігурація Nginx

Конфігурація описується у `/etc/nginx/nginx.conf` або у файлах `/etc/nginx/conf.d/*.conf`. Структура: блоки `http`, `server`, `location`.

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        root /var/www/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
}
```

Директива `try_files` важлива для SPA (React): спочатку шукає файл, потім папку, потім відправляє на `index.html` для клієнтської маршрутизації.

### Reverse Proxy

Reverse proxy приймає запити від клієнтів та перенаправляє їх на backend-сервери. Клієнт взаємодіє тільки з Nginx, не знаючи про внутрішню архітектуру.

```nginx
server {
    listen 80;

    location / {
        root /var/www/frontend;
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://localhost:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Переваги: єдина точка входу, SSL termination, кешування на рівні proxy, захист backend-серверів.

### Статичні файли

Nginx ефективно обслуговує статичні файли (HTML, CSS, JS, зображення). Gzip-стиснення зменшує розмір відповідей: `gzip on; gzip_types text/css application/javascript;`. Кешування через заголовки `expires` або `add_header Cache-Control`.

### HTTPS (SSL/TLS)

Налаштування HTTPS у Nginx:

```nginx
server {
    listen 443 ssl;
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
}
```

Let's Encrypt - безкоштовні SSL-сертифікати. Certbot - CLI для автоматичного отримання та оновлення сертифікатів. Рекомендовано перенаправляти HTTP на HTTPS через окремий `server` блок.

---

## Контрольні питання

1. Що таке Nginx і які основні сценарії його використання?
2. Чим reverse proxy відрізняється від forward proxy?
3. Як налаштувати Nginx для обслуговування SPA (React-застосунку)?
4. Як налаштувати проксіювання API-запитів на backend-сервер?
5. Для чого використовуються заголовки `X-Real-IP` та `X-Forwarded-For`?
6. Як налаштувати HTTPS у Nginx за допомогою Let's Encrypt?

---

## Порядок виконання

1. Вивчити теоретичний матеріал, наведений у цьому документі.
2. Ознайомитись з рекомендованими джерелами.
3. Відповісти на контрольні питання.

---

## Рекомендовані джерела

- [Nginx - офіційна документація](https://nginx.org/en/docs/) - повна документація Nginx
- [Nginx - Reverse Proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/) - гайд по налаштуванню reverse proxy
- [DigitalOcean - Nginx Tutorial](https://www.digitalocean.com/community/tutorials/understanding-nginx-http-proxying-load-balancing-buffering-and-caching) - практичний гайд
- [Let's Encrypt](https://letsencrypt.org/) - безкоштовні SSL-сертифікати
- [Certbot](https://certbot.eff.org/) - інструмент для автоматизації SSL
- [Nginx - Wikipedia](https://en.wikipedia.org/wiki/Nginx) - загальний опис та історія
