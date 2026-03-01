# yandex_etl

Учебный ETL-стенд с Airflow, Spark standalone, PostgreSQL, ClickHouse, Kafka, MinIO и landing page со ссылками на UI и параметрами подключения к БД.

Airflow в базовом стенде поднимается на чистом официальном образе без дополнительных provider-пакетов, чтобы старт был предсказуемым и не ломался из-за конфликтов зависимостей.

## Структура

```text
yandex_etl/
  dags/
  plugins/
  dbt/
  scripts/
  kafka_connect/
  jupyter_notebooks/
  infra/
    docker/
      docker-compose.yml
      .env.example
      airflow/
        Dockerfile
        requirements.txt
      landing/
        index.html
```

`dags/` и `plugins/` монтируются в Airflow напрямую, поэтому после деплоя или локального обновления новые DAG-файлы автоматически появляются в UI.
Папка `infra/docker/airflow/` оставлена как задел под будущую кастомизацию, но текущий стенд использует официальный образ `apache/airflow:2.9.3-python3.11` напрямую из compose.

## Быстрый старт

1. Скопируй пример env:

```bash
cd infra/docker
cp .env.example .env
```

2. Подними стек:

```bash
docker compose --env-file .env up -d --build
```

3. Открой landing page:

```text
http://localhost:18085
```

На landing page есть ссылки на UI и кнопки с параметрами подключения для PostgreSQL, ClickHouse и Greenplum.

## Порты по умолчанию

По умолчанию стенд вынесен в отдельный диапазон портов, чтобы не конфликтовать с другими локальными проектами и системными сервисами.
Если нужно, все внешние порты меняются только через `infra/docker/.env`.

## UI и доступы по умолчанию

Если стек поднят локально, используй `localhost`.
Если стек поднят на сервере, замени `localhost` на IP или домен сервера, например `http://<server-ip>:18080`.

- Airflow UI: `http://localhost:18080` (`admin / admin123`)
- Kafka UI: `http://localhost:18081`
- MinIO Console: `http://localhost:19001` (`minio / minio12345`)
- Spark Master UI: `http://localhost:18088`
- Spark Master RPC: `localhost:17077`
- Kafka broker: `localhost:19092`
- PostgreSQL: `localhost:15432`
- ClickHouse HTTP: `http://localhost:18123`
- ClickHouse native: `localhost:19009`
- Greenplum: `localhost:15433`
- Landing page: `http://localhost:18085`

## Подключение из DBeaver

Если подключаешься к локальному стеку, хост `localhost`.
Если к серверу, используй IP или домен сервера вместо `localhost`.
Те же значения дублируются в landing page по кнопке "Показать подключение".

### PostgreSQL

- Driver: `PostgreSQL`
- Host: `localhost`
- Port: `15432`
- Database: `airflow`
- User: `postgres`
- Password: `postgres123`
- JDBC URL: `jdbc:postgresql://localhost:15432/airflow`

### ClickHouse

- Driver: `ClickHouse`
- Host: `localhost`
- Port: `18123`
- Database: `default`
- User: `clickhouse`
- Password: `clickhouse123`
- JDBC URL: `jdbc:clickhouse://localhost:18123/default`

### Greenplum

- Driver: `PostgreSQL`
- Host: `localhost`
- Port: `15433`
- Database: `postgres`
- User: `gpadmin`
- Password: `greenplum123`
- JDBC URL: `jdbc:postgresql://localhost:15433/postgres`

## Spark submit

Для учебного режима с одним executor используй параметры:

```bash
--num-executors 1
--executor-cores 1
--executor-memory 1g
```

Для базового стенда дополнительные Airflow providers не доустанавливаются на старте.
Если позже понадобятся `SparkSubmitOperator`, S3/Kafka hooks или другие integration-пакеты, их лучше добавлять отдельным шагом с жёсткими constraints под версию Airflow.

## CI/CD

Workflow `.github/workflows/deploy.yml` делает три проверки до деплоя:

- валидирует структуру репозитория;
- проверяет парсинг `docker-compose.yml`;
- компилирует Python DAG-файлы.

После этого workflow копирует проект на сервер и выполняет `docker compose up -d --build`.
