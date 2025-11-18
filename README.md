# Nginx Logs Monitoring Stack

Полнофункциональный стек для сбора, анализа и визуализации логов Nginx с использованием современных инструментов observability.

## 🛠️ Технологии

- **Grafana 10.4.1** - визуализация метрик и логов
- **Loki 2.9.6** - агрегация логов
- **Prometheus v2.55.0** - сбор метрик
- **Vector 0.41.0** - log shipper для передачи логов в Loki
- **Nginx (stable)** - веб-сервер с демо endpoints

## 🚀 Быстрый старт

### 1. Клонируй репозиторий

```bash
git clone https://github.com/username/logging-project.git
cd logging-project
```

### 2. Создай необходимые директории

```bash
mkdir -p loki/data prometheus/data nginx/log grafana
```

### 3. Запусти стек

```bash
docker compose up -d
```

### 4. Открой Grafana

- **URL**: http://localhost:3000
- **Логин**: admin
- **Пароль**: admin

## 📊 Доступные сервисы

| Сервис | URL | Описание |
|--------|-----|---------|
| **Grafana** | http://localhost:3000 | Дашборды и визуализация |
| **Prometheus** | http://localhost:9090 | Метрики и алерты |
| **Loki** | http://localhost:3100 | API логов |
| **Vector** | http://localhost:8686 | Health check Vector |
| **Nginx** | http://localhost:8080 | Веб-сервер |

## 🔍 Генерируем тестовые логи

### Успешные запросы (200)

```bash
curl http://localhost:8080/
```

### Ошибки сервера (500)

```bash
curl http://localhost:8080/fail
```

### Задержки (для тестирования)

```bash
curl http://localhost:8080/delay
```

### Массовая генерация логов

```bash
for i in {1..100}; do curl http://localhost:8080/; done
```

## 📈 Дашборд "Nginx Logs Dashboard"

Основной дашборд содержит 8 панелей:

### Информационные панели (Text)

1. **Nginx Logs Monitoring Dashboard** - описание дашборда и технологий
2. **LogQL Queries Reference** - справка по LogQL запросам

### Метрики и логи

3. **Error Rate** (Gauge) - процент ошибок 4xx и 5xx
4. **Top HTTP Methods** (Bar Chart) - распределение типов запросов
5. **HTTP Status Codes Over Time** (Time Series) - тренд кодов ответов
6. **Log Volume Over Time** (Area Chart) - объём логов за время
7. **Top Requested URIs** (Table) - самые нагруженные endpoints
8. **Recent Error Logs** (Logs) - последние 100 ошибок

## 📝 Примеры LogQL запросов

### Все логи доступа

```logql
{log_type="access", service="nginx"}
```

### Только ошибки 5xx

```logql
{log_type="access", service="nginx"} |~ "HTTP/[0-9.]+ 5[0-9]{2}"
```

### Ошибки 4xx

```logql
{log_type="access", service="nginx"} |~ "HTTP/[0-9.]+ 4[0-9]{2}"
```

### Логи ошибок Nginx

```logql
{log_type="error", service="nginx"}
```

### Поиск конкретного пути

```logql
{log_type="access", service="nginx"} |= "/fail"
```

### Количество ошибок за период

```logql
count_over_time({log_type="access", service="nginx"} |~ "5[0-9]{2}" [5m])
```

## 🔧 Импорт дашборда в Grafana

### Автоматически (рекомендуется)

Если в `docker-compose.yml` настроен volume для provisioning:

```yaml
grafana:
  volumes:
    - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
```

Дашборд загружается автоматически при запуске.

### Вручную через UI

1. Открой **Grafana** → http://localhost:3000
2. Нажми **Dashboards** → **Import**
3. Нажми **"Upload dashboard JSON file"**
4. Выбери файл `grafana/dashboards/nginx-logs-dashboard.json`
5. Нажми **Import**

## 📂 Структура проекта

```
logging-project/
├── .gitignore                          # Исключения для Git
├── .dockerignore                       # Исключения для Docker
├── .env.example                        # Пример переменных окружения
├── README.md                           # Этот файл
├── LICENSE                             # Лицензия (MIT)
├── docker-compose.yml                  # Оркестрация сервисов
│
├── grafana/
│   ├── dashboards/
│   │   └── nginx-logs-dashboard.json   # Экспортированный дашборд
│   ├── grafana.db                      # База данных (в .gitignore)
│   └── plugins/                        # Плагины (опционально)
│
├── loki/
│   ├── config/
│   │   └── loki-config.yml             # Конфигурация Loki
│   └── data/                           # Данные логов (в .gitignore)
│
├── nginx/
│   ├── conf.d/
│   │   └── default.conf                # Конфигурация Nginx
│   └── log/                            # Логи Nginx (в .gitignore)
│
├── prometheus/
│   ├── prometheus.yml                  # Конфигурация Prometheus
│   └── data/                           # Время-серийная БД (в .gitignore)
│
└── vector/
    └── vector.yaml                     # Конфигурация Vector log shipper
```

## 🛑 Остановка сервисов

```bash
docker compose down
```

Для удаления также данных:

```bash
docker compose down -v
```

## 🐛 Решение проблем

### Логи не собираются

Проверь, что Vector работает:

```bash
docker compose logs vector
```

### Grafana не подключается к Loki

1. Проверь Data Source: **Settings** → **Data Sources** → **Loki**
2. URL должна быть: `http://loki:3100`

### Prometheus не scrapes метрики

```bash
docker compose logs prometheus
```

Проверь `prometheus/prometheus.yml` на синтаксис.

## 📚 Дополнительные ресурсы

- [Grafana документация](https://grafana.com/docs/grafana/latest/)
- [Loki документация](https://grafana.com/docs/loki/latest/)
- [LogQL запросы](https://grafana.com/docs/loki/latest/logql/)
- [Prometheus метрики](https://prometheus.io/docs/prometheus/latest/)
- [Vector документация](https://vector.dev/docs/)

## 📄 Лицензия

MIT License - смотри файл LICENSE

## 👨‍💻 Для разработчиков

Этот проект идеален для:

- Изучения Grafana и Loki
- Обучения работе с observability стеком
- Мониторингу Nginx логов
- Портфолио DevOps инженера

### Рекомендуемые улучшения

- [ ] Добавить alert rules в Prometheus
- [ ] Настроить notification channels (Email, Slack)
- [ ] Добавить роли пользователей в Grafana
- [ ] Настроить persistence volumes для production
- [ ] Добавить nginx-exporter для метрик производительности
- [ ] Создать CI/CD pipeline для автоматического развёртывания

## 🤝 Контрибьютинг

Contributions приветствуются! Создавай pull requests для улучшений и исправлений ошибок.
