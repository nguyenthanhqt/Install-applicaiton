# CÀI ĐẶT DOCKER
## Bước 1. Cài đặt git
```bash
yum install git -y
```


## Bước 2. Cài đặt Docker
```bash
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
docker -v
```


## Bước 3. Cài đặt Docker Compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

## Bước 4: Tạo file docker-compose.yml với nội dung
```bash
version: '3.7'

services:
  sentry-redis:
    image: redis:latest
    container_name: sentry-redis
    hostname: sentry-redis
    restart: always
    networks:
      - sentry
    volumes:
      - './data/sentry/redis/data:/data'
  
  sentry-postgres:
    image: postgres:latest
    container_name: sentry-postgres
    hostname: sentry-postgres
    restart: always
    environment:
      POSTGRES_USER: sentry
      POSTGRES_PASSWORD: 89PsZXyRStOT2
      POSTGRES_DB: sentry
    networks:
      - sentry
    volumes:
      - './data/sentry/postgres:/var/lib/postgresql/data'

  sentry-base:
    image: sentry:latest
    container_name: sentry-base
    hostname: sentry-base
    restart: always
    ports:
      - '9000:9000'
    env_file:
      - .env
    depends_on:
      - sentry-redis
      - sentry-postgres
    networks:
      - sentry
    volumes:
      - './data/sentry/sentry:/var/lib/sentry/files'

  sentry-cron:
    image: sentry:latest
    container_name: sentry-cron
    hostname: sentry-cron
    restart: always
    env_file:
      - .env
    depends_on:
      - sentry-redis
      - sentry-postgres
    command: "sentry run cron"
    networks:
      - sentry
    volumes:
      - './data/sentry/sentry:/var/lib/sentry/files'

  sentry-worker:
    image: sentry:latest
    container_name: sentry-worker
    hostname: sentry-worker
    restart: always
    env_file:
      - .env
    depends_on:
      - sentry-redis
      - sentry-postgres
    command: "sentry run worker"
    networks:
      - sentry
    volumes:
      - './data/sentry/sentry:/var/lib/sentry/files'

networks:
  sentry:
    driver: bridge
```
## Bước 5: tạo file biến môi trường .env có nội dung như sau
```bash
SENTRY_SECRET_KEY=='6(@#k63elpr@*b(+x(!-#0n^cc864fv@sy22c069dy_hrl9y_='
SENTRY_POSTGRES_HOST=sentry-postgres
SENTRY_POSTGRES_PORT=5432
SENTRY_DB_NAME=sentry
SENTRY_DB_USER=sentry
SENTRY_DB_PASSWORD=89PsZXyRStOT2
SENTRY_REDIS_HOST=sentry-redis
SENTRY_REDIS_PORT=6379
```

## Bước 6: Generate secret key và lấy key thay vào file biến môi trường .env

```bash
docker-compose run --rm sentry-base config generate-secret-key
```
## Bước 7: chạy lệnh
```bash
    Docker-compose up -d
```

## Bước 8: truy cập vào http://{ip máy cài đặt}:9000 
### username; password (lúc cài đặt có yêu cầu)
# --- Thank ---