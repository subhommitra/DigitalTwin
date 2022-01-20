# This repo contains Helm charts for deploying Postgres Operator, Strimzi Operator, and Keycloak Operator


## Filling in the configuration details

A `conf.yaml` file contains passwords and service names that must be filled in prior to deploying using Helm.
```
# Passwords for keycloakOperator, postgresOperator, and strimzi must be same since we are using a single DB user
keycloakOperator:
  keycloak_db:
    stringData:
      POSTGRES_EXTERNAL_ADDRESS: <database service name>.<database service namespace>.svc.cluster.local
      POSTGRES_PASSWORD: <password for "ngb" user>

  custom_resource:
    credentials:
      ADMIN_PASSWORD: <set the password for keycloak console>
      ADMIN_USERNAME: <set the username for keycloak console>

postgresOperator:
  teamId: <set the team ID for postgres cluster>
  # clusterSvcName: name must begin with "<teamId>-"
  clusterSvcName: <set the database service name for postgres cluster>
  dbPassword: <set the password for "ngb" user>

strimzi:
  kafkaConnector:
    hostname: <database service name>
    password: <password for "ngb" user>

  imageCredentials:
    username: <username for ibn40/digitaltwin dockerhub>
    password: <password for ibn40/digitaltwin dockerhub>
```
- `postgresOperator.dbPassword` is used to **set** the password for the *ngb* user in Postgres. Keycloak and Strimzi (and other Scorpio components) utilise the same Postgres and the same Postgres user. Therefore, `keycloakOperator.keycloak_db.stringData.POSTGRES_PASSWORD` and `strimzi.kafkaConnector.password` need to populated with the same value.
- `postgresOperator.clusterSvcName` and `teamId` is used to **set** the Kubernetes Service name for the Postgres cluster. Note that Postgres Operator enforces that the service name is prefixed with the team ID and a dash. For instance, if `teamId` is set to "acid", a valid `clusterSvcName` will be "acid-foo" or "acid-bar" or "acid-foo-bar". The `clusterSvcName` value is also required by `strimzi.kafkaConnector.hostname` and `keycloakOperator.keycloak_db.stringData.POSTGRES_EXTERNAL.ADDRESS`, the latter needing a FQDN and therefore must be suffixed by the namespace and `svc.cluster.local`
- `keycloak.custom_resource.credentials` is used to **set** the username and password to access the Keycloak console.
- `strimzi.imageCredentials` needs to be configured with the username and password to the **ibn40/digitaltwin** repository in DockerHub. **Since these are sensitive credentials, it is recommended to set them using command line arguments as shown in the "Installation" section instead of writing them in the conf.yaml.**


## ibn40 Docker Hub images

ScoprioBroker components are pulled from `ibn40/digitaltwin` repository in Docker Hub. Kafka Connect, a part of Strimzi, is pulled using the tag `connect_tgz`.


## Installation

Please ensure your `kubectl` is configured to use the credentials to your Kubernetes cluster by default. Generally this would mean adding your credentials to `~/.kube/config` file.

Please ensure the namespace exists before executing this command. You can create a new namespace using `kubectl create ns <namespace>`. It is recommended to use `kubens` to prevent specifying the namespace manually on every command.

```
helm install <release-name> . -f conf.yaml -n <namespace> --set strimzi.imageCredentials.username=<insert dockerhub username> --set strimzi.imageCredentials.password=<insert dockerhub password>
```


## Accessing Keycloak console via GUI

You will need to add the following to your `hosts` file to access the Keycloak GUI using your browser:
```
85.215.213.132 keycloak.local
```
The IP and the hostname can be retrieved from the output of `kubectl get ingress -n <namespace>`.


## Uninstallation

Services deployed using this chart can be removed using Helm `uninstall`
```
helm uninstall <release-name> -n <namespace>
```