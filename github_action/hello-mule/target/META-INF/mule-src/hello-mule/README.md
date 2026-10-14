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

## Runtime and Java

This project targets Mule Runtime `4.12.2` and declares Java `17` compatibility in its Maven properties. The minimum Mule version in `mule-artifact.json` is `4.12.0`, so the application can run on the 4.12 runtime line.

CloudHub 2.0 client credentials are intentionally not configured. Add deployment configuration later through secure GitHub Actions secrets and the appropriate Mule Maven Plugin deployment settings.

## GitHub Actions and CloudHub 2.0

The workflow in `.github/workflows/java-ci.yml` builds the Mule application for pull requests and pushes. It deploys to CloudHub 2.0 only for pushes to `main` or a manual workflow run. Pull requests do not deploy.

Before deploying, create a GitHub environment named `cloudhub-production`. Add these environment **secrets**:

- `ANYPOINT_CLIENT_ID`
- `ANYPOINT_CLIENT_SECRET`

Add these environment **variables**:

- `ANYPOINT_URI` (usually `https://anypoint.mulesoft.com`)
- `ANYPOINT_BUSINESS_GROUP_ID`
- `ANYPOINT_ENVIRONMENT`
- `ANYPOINT_TARGET`
- `ANYPOINT_APPLICATION_NAME`
- `ANYPOINT_REPLICAS` (for example, `1`)
- `ANYPOINT_VCORES` (for example, `0.1`)

The Connected App used by the secrets must have permission to deploy applications in the selected business group and environment. GitHub Actions passes these values to Maven at deployment time; they are not stored in the repository.
