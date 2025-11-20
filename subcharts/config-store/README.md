# Config Store

A Helm chart for demonstrating using subcharts and deploying the config-store application.

## Setup Instructions

Follow these steps to set up and deploy the config-store project:

### 1. Configure Environment Variables

Create a `.env` file based on the provided example and populate it with your database passwords:

```bash
cp .env.example .env
```

Edit the `.env` file and add your password values:

- Set `postgres-password` for the database root password
- Set `password` for the application user password

### 2. Create Kubernetes Secret

Create a Kubernetes secret from your `.env` file:

```bash
kubectl create secret generic postgres-creds --from-env-file=.env
```

### 3. Install the Chart

Deploy the application using Helm:

For a new installation:

```bash
helm install config-store .
```

Or with a single command for both new installation and upgrades:

```bash
helm upgrade config-store . --install
```

Follow the creation and deployment of pods with `kubectl get pod --watch`. The application pod might restart a few times until the DB pod is up and running, just wait a few moments until everything is up and running.

## Troubleshooting

### 1. PostgreSQL pod stuck in `CreateContainerConfigError`

This is probably due to a missing secret or password key in the present secret. Make sure that:

1. There is a secret named `postgres-creds` present in the same namespace where the chart is deployed.
2. The secret contains both the `password` and `postgres-password` keys.

### 2. Config-store pod continuously restarting

One of the most common reasons for that is the database pod not being available, leading the config-store pod to not initializing correctly. Make sure that the PostgreSQL pod is correctly initialized (might take a few seconds to a couple of minutes after deploying the chart).

Another common reason is the incorrect configuration of the deployment template for the application. Make sure that its values are correctly configured, especially the environment variables that specify database credentials and connection strings.

### 3. Config-store not able to authenticate into DB

This can happen if the chart was installed, uninstalled, and then re-installed again. This is due to the fact that the underlying PVC and PV for the database are **not** removed upon uninstalling the chart. The solution for that is as follows:

1. Uninstall the chart with `helm uninstall config-store`
2. Remove the corresponding PVC with `kubectl delete pvc data-postgresql-db-0`, and make sure that the PV is also deleted (should be the case when using dynamic provisioning of PVs with minikube; otherwise, make sure to delete and re-create the PV manually)
3. Reinstall the chart
