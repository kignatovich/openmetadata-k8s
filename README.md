# openmetadata-k8s

Для подключение Postgresql как основнйо базы для метадаты

openmetadata-helm-charts/charts/openmetadata/values.yaml

```yaml
openmetadata:
  config:
    ...
    database:
      host: <postgresql_endpoint>
      port: 5432
      driverClass: org.postgresql.Driver
      dbScheme: postgresql
      dbUseSSL: true
      databaseName: <openmetadata_database_name>
      auth:
        username: <database_login_user>
        password:
          secretRef: openmetadata-postgresql-secrets
          secretKey: openmetadata-postgresql-password
```


https://docs.open-metadata.org/latest/deployment/kubernetes/on-prem


Подключение эластика.

```yaml
    elasticsearch:
      enabled: true
      host: opensearch
      searchType: opensearch
      port: 9200
      scheme: http
      clusterAlias: ""
      # Value in Bytes
      payLoadSize: 10485760
      connectionTimeoutSecs: 5
      socketTimeoutSecs: 60
      batchSize: 100
      searchIndexMappingLanguage: "EN"
      keepAliveTimeoutSecs: 600
      trustStore:
        enabled: false
        path: ""
        password:
          secretRef: "elasticsearch-truststore-secrets"
          secretKey: "openmetadata-elasticsearch-truststore-password"
      auth:
        enabled: false
        username: "elasticsearch"
        password:
          secretRef: elasticsearch-secrets
          secretKey: openmetadata-elasticsearch-password
```

Подключение внешнего Airflow

Дефолтный конфиг

```yaml
    pipelineServiceClientConfig:
      enabled: true
      className: "org.openmetadata.service.clients.pipeline.airflow.AirflowRESTClient"
      # endpoint url for airflow
      apiEndpoint: http://openmetadata-dependencies-web:8080
      # this will be the api endpoint url of OpenMetadata Server
      metadataApiEndpoint: http://openmetadata:8585/api
      # possible values are "no-ssl", "ignore", "validate"
      verifySsl: "no-ssl"
      hostIp: ""
      ingestionIpInfoEnabled: false
      # healthCheckInterval in seconds
      healthCheckInterval: 300
      # local path in Airflow Pod
      sslCertificatePath: "/no/path"
      auth:
        enabled: true
        username: admin
        password:
          secretRef: airflow-secrets
          secretKey: openmetadata-airflow-password
        trustStorePath: ""
        trustStorePassword:
          secretRef: ""
          secretKey: ""
```
Что нужно заменить

```yaml
pipelineServiceClientConfig:
  enabled: true
  className: "org.openmetadata.service.clients.pipeline.airflow.AirflowRESTClient"

  # URL Airflow REST API (ВАЖНО: это Airflow Webserver, не Flower и не Scheduler)
  apiEndpoint: http://my-airflow.example.com:8080

  # URL OpenMetadata API — он остаётся вашим сервисом
  metadataApiEndpoint: http://openmetadata:8585/api

  verifySsl: "no-ssl"

  auth:
    enabled: true
    username: airflow_user
    password:
      secretRef: airflow-secrets
      secretKey: airflow-password
```
Внимание в Airflow должен быть установлен OpenMetadata Provider

Нужно при билде контейнера предусмотреть установку провайдера
```bash
pip install apache-airflow-providers-openmetadata
```
Создание пользователя для метадаты(можно использовать стандартного админа)

```
airflow users create \
  --username airflow_user \
  --password StrongPass123 \
  --firstname System \
  --lastname User \
  --role Admin \
  --email airflow@example.com
```

Создание секрета 

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: airflow-secrets
type: Opaque
stringData:
  airflow-password: "StrongPass123"
```

OpenMetadata UI — указываем "External Airflow"

После обновления чарта:

Зайти в UI → Settings → Ingestion.
1. Airflow Service → Edit
2. Выбрать External Airflow
3. Указать адрес Airflow:
4. http://my-airflow.example.com:8080
5. Протестировать подключение.

