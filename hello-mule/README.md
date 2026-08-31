# hello-mule

A small Mule 4 REST API for learning MuleSoft and Maven. It exposes one endpoint:

- `GET /hello`

The endpoint returns a JSON greeting and does not require authentication or credentials.

## Project structure

```text
hello-mule/
|-- pom.xml                         Maven project and Mule build configuration
|-- mule-artifact.json              Mule application metadata
|-- README.md                       Project documentation
|-- src/main/mule/hello-mule.xml    HTTP flow and DataWeave transformation
`-- src/main/resources/
    `-- application.properties      Local HTTP port configuration
```

## Build

From the `hello-mule` directory, run:

```shell
mvn clean package
```

The Mule Maven Plugin packages the application into a deployable Mule application archive in `target/`. The `clean` phase removes the previous build output, and the `package` phase validates and packages the application.

The first build may take several minutes because Maven downloads the Mule Maven Plugin and connector dependencies from configured Maven repositories.

## Run locally

A Mule application normally runs in Anypoint Studio or with the Mule runtime. The included `application.properties` sets the local listener port to `8081`. When launching the packaged application with a Mule runtime, use the default property or override it with another value, for example:

```shell
mule -M-Dhttp.port=8081 target/hello-mule-1.0.0-mule-application.jar
```

The exact launcher command can vary by Mule runtime installation. No CloudHub deployment configuration is included yet.

## Test the API

Once the application is running, request:

```shell
curl http://localhost:8081/hello
```

Expected response:

```json
{
  "message": "Hello from MuleSoft!",
  "application": "hello-mule"
}
```

The GitHub Actions workflow in `.github/workflows/java-ci.yml` packages the app, publishes it to Anypoint Exchange, and deploys `hello-mule-saikiran` to CloudHub 2.0.

## Observability (Splunk and Grafana)

Each `GET /hello` call logs the request and, when enabled, ships a JSON event to:

- **Splunk** HTTP Event Collector
- **Grafana Loki** (Grafana reads Loki)

Exports are **off by default** so CloudHub deploy stays green without Splunk/Grafana accounts. They do not block `/hello` if an export fails.

### Local Grafana + Loki

From the repository root:

```shell
docker compose -f observability/docker-compose.yml up -d
```

- Grafana: http://localhost:3000 (user `admin` / password `admin`)
- Loki: http://localhost:3100

Enable Mule exports in `hello-mule/src/main/resources/application.properties`:

```properties
splunk.enabled=false
loki.enabled=true
loki.url=http://localhost:3100/loki/api/v1/push
```

Then call `GET /hello` and open the **hello-mule observability** dashboard in Grafana. Query: `{app="hello-mule"}`.

### Local Splunk

Splunk is heavy. Start it only if you have enough RAM:

```shell
docker compose -f observability/docker-compose.yml --profile splunk up -d
```

- Splunk UI: http://localhost:8000 (user `admin` / password `HelloMule123`)
- HEC: `http://localhost:8088/services/collector/event` with token `hello-mule-local-hec`

Set:

```properties
splunk.enabled=true
splunk.hec.url=http://localhost:8088/services/collector/event
splunk.hec.token=hello-mule-local-hec
```

### CloudHub 2.0

CloudHub cannot reach `localhost`. Use Splunk Cloud HEC and a public Loki/Grafana Cloud URL, then set GitHub Actions env (or CloudHub properties):

- `SPLUNK_ENABLED=true`
- `SPLUNK_HEC_URL`
- `SPLUNK_HEC_TOKEN`
- `LOKI_ENABLED=true`
- `LOKI_URL`

There is also `GET /health` for a simple up-check.

## Runtime and Java

This project targets Mule Runtime **4.6 LTS** and Java **17** on CloudHub 2.0.
