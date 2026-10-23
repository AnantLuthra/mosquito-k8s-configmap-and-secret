# Mosquitto on Kubernetes: ConfigMap and Secret Volumes

This project demonstrates two ways to deploy the Eclipse Mosquitto MQTT broker on Kubernetes:

1. A basic Deployment with no configuration or Secret volumes mounted.
2. A Deployment that mounts a Mosquitto configuration file from a `ConfigMap` and a Secret file from a Kubernetes `Secret`.

It is intended as a learning example for Kubernetes Deployments, ConfigMaps, Secrets, and volume mounts.

## Files

| File | Purpose |
| --- | --- |
| `mosquito-no-volumes.yaml` | Example baseline Deployment. It runs Mosquitto without mounting data into the configuration or Secret paths. |
| `config-file.yaml` | Creates the `mosquitto-config-file` ConfigMap containing `mosquitto.conf`. |
| `secret-file.yaml` | Creates the `mosquitto-secret-file` Secret containing a sample Secret value. |
| `mosq-deployment.yaml` | Deployment that mounts the ConfigMap at `/mosquitto/config` and the Secret at `/mosquitto/secret`. |
| `sample-secret-certifi.yaml` | Example certificate Secret manifest with placeholder data. |

## Prerequisites

- A Kubernetes cluster
- `kubectl` installed and configured for the target cluster

## Deploy the baseline example

The baseline manifest demonstrates a normal Mosquitto Deployment without custom volume mounts:

```bash
kubectl apply -f mosquito-no-volumes.yaml
kubectl get deployment mosquitto
kubectl get pods -l app=mosquitto
```

## Deploy with ConfigMap and Secret mounts

Apply the ConfigMap and Secret before applying the Deployment that references them:

```bash
kubectl apply -f config-file.yaml
kubectl apply -f secret-file.yaml
kubectl apply -f mosq-deployment.yaml
```

Verify the resources:

```bash
kubectl get configmap mosquitto-config-file
kubectl get secret mosquitto-secret-file
kubectl get deployment mosquitto
kubectl describe pod -l app=mosquitto
```

The mounted resources are available inside the container at:

- ConfigMap: `/mosquitto/config`
- Secret: `/mosquitto/secret`

The Secret volume is mounted as read-only.

## Configuration

The ConfigMap currently provides a `mosquitto.conf` file with:

- Logging directed to standard output
- All log types enabled
- Log timestamps enabled
- A listener configured on port `9001`

The Deployment declares container port `1883`, so the listener and declared port should be aligned if this manifest is used beyond the demonstration. The repository does not currently define a Kubernetes `Service`; therefore, the broker is not exposed through a stable cluster address.

## Important notes

- `mosquito-no-volumes.yaml` is intentionally a baseline example. It shows that a normal Deployment does not automatically mount the ConfigMap and Secret paths.
- `mosq-deployment.yaml` uses the same Deployment name, `mosquitto`, as the baseline manifest. Applying it updates/replaces the existing Deployment configuration.
- The value in `secret-file.yaml` is an example only. Do not commit real passwords, certificates, or private keys to a public repository.
- Kubernetes Secret values in the manifest are base64-encoded, not encrypted. Use an external secret manager or a secure deployment workflow for production workloads.
- `sample-secret-certifi.yaml` contains placeholder certificate data and is not a usable certificate.

## Remove the resources

```bash
kubectl delete -f mosq-deployment.yaml
kubectl delete -f secret-file.yaml
kubectl delete -f config-file.yaml
```

If the baseline Deployment is currently applied, remove it with:

```bash
kubectl delete -f mosquito-no-volumes.yaml
```
