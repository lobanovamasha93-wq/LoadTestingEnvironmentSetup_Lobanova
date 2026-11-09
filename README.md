# Стенд мониторинга для нагрузочного тестирования

Домашнее задание к лекции 2 «Подготовка стенда нагрузочного тестирования» курса Netology «Нагрузочное тестирование».

Развёрнуты две независимые системы сбора метрик, отображаемые в одной Grafana:

- **telegraf → InfluxDB** — пушит метрики хоста каждые **60 секунд**.
- **node-exporter → Prometheus** — Prometheus опрашивает node-exporter каждые **36 секунд**.

## Состав стенда

| Сервис | Образ | Порт | Назначение |
|---|---|---|---|
| telegraf | `telegraf:1.31-alpine` | — | сбор метрик хоста, отправка в InfluxDB |
| influxdb | `influxdb:1.8-alpine` | 8086 | хранилище метрик telegraf |
| node-exporter | `prom/node-exporter:v1.8.1` | 9100 | экспорт метрик хоста в формате Prometheus |
| prometheus | `prom/prometheus:v2.53.0` | 9090 | сбор и хранение метрик node-exporter |
| grafana | `grafana/grafana:11.1.0` | 3000 | дашборды по обоим источникам |
| renderer | `grafana/grafana-image-renderer:3.12.3` | 8081 | headless-рендер дашбордов в PNG (для скриншотов) |

Оба стека мониторят метрики хост-машины: `telegraf` и `node-exporter` монтируют `/proc`, `/sys` и корневую ФС хоста в режиме read-only, а не метрики самих контейнеров.

## Структура репозитория

```
docker-compose.yml
telegraf/telegraf.conf          # конфиг telegraf
prometheus/prometheus.yml       # конфиг prometheus
grafana/provisioning/           # автопровижининг источников данных и дашбордов
grafana/dashboards/             # JSON дашбордов (928 и 1860 с grafana.com)
screenshots/                    # скриншоты метрик системы в покое (15+ минут)
```

## Что настроено по заданию

- **telegraf**: интервал сбора и отправки метрик — `interval = "60s"`, подключены плагины `mem`, `swap`, `disk`, `diskio`, `net`, `netstat` (плюс `cpu`, `system`, `kernel`, `processes` для полноты аппаратных метрик).
- **prometheus**: `scrape_interval: 36s`.
- **grafana**: дашборд [Telegraf: system dashboard (928)](https://grafana.com/grafana/dashboards/928) для стека telegraf/InfluxDB и [Node Exporter Full (1860)](https://grafana.com/grafana/dashboards/1860) для стека node-exporter/Prometheus, оба провиженированы автоматически из `grafana/dashboards/`.
- **screenshots/**: снимки обоих дашбордов, когда система была в состоянии покоя (без искусственной нагрузки) на протяжении от 15 минут.

## Как запустить

```bash
docker compose up -d
```

- Grafana: http://localhost:3000 (admin/admin)
- Prometheus: http://localhost:9090
- InfluxDB: http://localhost:8086

После старта подождать 1–2 минуты, пока оба датасорса наберут первые точки данных.

## Скриншоты

Все скриншоты лежат в [`screenshots/`](./screenshots):

- `telegraf-start.png`, `prometheus-start.png` — начало окна наблюдения.
- `telegraf-after-15min.png`, `prometheus-after-15min.png` — состояние метрик после 15+ минут непрерывной работы системы в покое (без нагрузки).
