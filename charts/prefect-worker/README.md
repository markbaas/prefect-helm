# prefect-worker

## Overview

[Workers](https://docs.prefect.io/latest/concepts/work-pools/#worker-overview) are lightweight polling services that retrieve scheduled runs from a work pool and execute them.

Workers each have a type corresponding to the execution environment to which they will submit flow runs. Workers are only able to join work pools that match their type. As a result, when deployments are assigned to a work pool, you know in which execution environment scheduled flow runs for that deployment will run.

## Installing the Chart

### Prerequisites

1. Add the Prefect Helm repository to your Helm client:

    ```bash
    helm repo add prefect https://prefecthq.github.io/prefect-helm
    helm repo update
    ```

2. Create a new namespace in your Kubernetes cluster to deploy the Prefect worker in:

    ```bash
    kubectl create namespace prefect
    ```

### Configuring a Worker for Prefect Cloud

1. Create a Kubernetes secret for a Prefect Cloud API key

    First create a file named `api-key.yaml` with the following contents:

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: prefect-api-key
      namespace: prefect
    type: Opaque
    data:
      key:  <base64-encoded-api-key>
    ```

    Replace `<base64-encoded-api-key>` with your Prefect Cloud API key encoded in base64. The helm chart looks for a secret of this name and schema, this can be overridden in the `values.yaml`.

    You can use the following command to generate the base64-encoded value:

    ```bash
    echo -n "your-prefect-cloud-api-key" | base64
    ```

    Then apply the `api-key.yaml` file to create the Kubernetes secret:

    ```bash
    kubectl apply -f api-key.yaml
    ```

    Alternatively, you can create the Kubernetes secret via the cli with the following command. In this case, Kubernetes will take care of base64 encoding the value on your behalf:

    ```bash
    kubectl create secret generic prefect-api-key --from-literal=key=pnu_xxxx
    ```

2. Configure the Prefect worker values

    Create a `values.yaml` file to customize the Prefect worker configuration. Add the following contents to the file:

    ```yaml
    worker:
      cloudApiConfig:
        accountId: <target account ID>
        workspaceId: <target workspace ID>
      config:
        workPool: <target work pool name>
    ```

    These settings will ensure that the worker connects to the proper account, workspace, and work pool.
    View your Account ID and Workspace ID in your browser URL when logged into Prefect Cloud. For example: `https://app.prefect.cloud/account/abc-my-account-id-is-here/workspaces/123-my-workspace-id-is-here`

### Configuring a Worker for Self Hosted Cloud

1. Create a Kubernetes secret for a Prefect Self Hosted Cloud API key

    First create a file named `api-key.yaml` with the following contents:

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: prefect-api-key
      namespace: prefect
    type: Opaque
    data:
      key:  <base64-encoded-api-key>
    ```

    Replace `<base64-encoded-api-key>` with your Prefect Self Hosted Cloud API key encoded in base64. The helm chart looks for a secret of this name and schema, this can be overridden in the `values.yaml`.

    You can use the following command to generate the base64-encoded value:

    ```bash
    echo -n "your-prefect-self-hosted-cloud-api-key" | base64
    ```

    Then apply the `api-key.yaml` file to create the Kubernetes secret:

    ```bash
    kubectl apply -f api-key.yaml
    ```

    Alternatively, you can create the Kubernetes secret via the cli with the following command. In this case, Kubernetes will take care of base64 encoding the value on your behalf:

    ```bash
    kubectl create secret generic prefect-api-key --from-literal=key=pnu_xxxx
    ```

2. Configure the Prefect worker values

    Create a `values.yaml` file to customize the Prefect worker configuration. Add the following contents to the file:

    ```yaml
    worker:
      apiConfig: selfHosted
      config:
        workPool: <target work pool name>
      selfHostedApiConfig:
        apiUrl: "https://<DNS of Self Hosted Cloud API>"
        accountId: <target account ID>
        workspaceId: <target workspace ID>
        uiUrl: "https://<DNS of Self Hosted Cloud UI>"
    ```

    These settings will ensure that the worker connects to the proper account, workspace, and work pool.
    View your Account ID and Workspace ID in your browser URL when logged into Prefect Cloud. For example: `https://self-hosted-prefect.company/account/abc-my-account-id-is-here/workspaces/123-my-workspace-id-is-here`

### Configuring a Worker for Prefect Server

1. Configure the Prefect worker values

    Create a `values.yaml` file to customize the Prefect worker configuration. Add the following contents to the file:

    ```yaml
    worker:
      apiConfig: server
      config:
        workPool: <target work pool name>
      serverApiConfig:
        apiUrl: <dns or ip address of the prefect-server pod here>
    ```

    These settings will ensure the worker connects with the local deployment of Prefect Server.
    If the Prefect Server pod is deployed in the same cluster, you can use the local Kubernetes DNS address to connect to it: `prefect-server.<namespace>.svc.cluster.local`

### Installing & Verifying Deployment of the Prefect Worker

1. Install the Prefect worker using Helm

    ```bash
    helm install prefect-worker prefect/prefect-worker --namespace=prefect -f values.yaml
    ```

2. Verify the deployment

    Check the status of your Prefect worker deployment:

    ```bash
    kubectl get pods -n prefect

    NAME                              READY   STATUS    RESTARTS       AGE
    prefect-worker-658f89bc49-jglvt   1/1     Running   0              25m
    ```

    You should see the Prefect worker pod running

## FAQ

### Deploying multiple workers to a single namespace

If you want to run more than one worker in a single Kubernetes namespace, you will need to specify the `fullnameOveride` parameter at install time of one of the workers.

```yaml
fullnameOverride: prefect-worker-2
```

If you want the workers to share a service account, add the following to your `values.yaml`:

```yaml
fullnameOverride: prefect-worker-2
serviceAccount:
  create: false
  name: "prefect-worker"
```

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| jamiezieziula | <jamie@prefect.io> |  |
| jimid27 | <jimi@prefect.io> |  |
| parkedwards | <edward@prefect.io> |  |

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://charts.bitnami.com/bitnami | common | 2.13.2 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| commonAnnotations | object | `{}` | annotations to add to all deployed objects |
| commonLabels | object | `{}` | labels to add to all deployed objects |
| fullnameOverride | string | `"prefect-worker"` | fully override common.names.fullname |
| nameOverride | string | `""` | partially overrides common.names.name |
| namespaceOverride | string | `""` | fully override common.names.namespace |
| role.extraPermissions | list | `[]` | array with extra permissions to add to the worker role |
| serviceAccount.annotations | object | `{}` | additional service account annotations (evaluated as a template) |
| serviceAccount.create | bool | `true` | specifies whether a ServiceAccount should be created |
| serviceAccount.name | string | `""` | the name of the ServiceAccount to use. if not set and create is true, a name is generated using the common.names.fullname template |
| worker.affinity | object | `{}` | affinity for worker pods assignment |
| worker.apiConfig | string | `"cloud"` | one of 'cloud', 'selfHosted', or 'server' |
| worker.cloudApiConfig.accountId | string | `""` | prefect account ID |
| worker.cloudApiConfig.apiKeySecret.key | string | `"key"` | prefect API secret key |
| worker.cloudApiConfig.apiKeySecret.name | string | `"prefect-api-key"` | prefect API secret name |
| worker.cloudApiConfig.cloudUrl | string | `"https://api.prefect.cloud/api"` | prefect cloud API url; the full URL is constructed as https://cloudUrl/accounts/accountId/workspaces/workspaceId |
| worker.cloudApiConfig.workspaceId | string | `""` | prefect workspace ID |
| worker.clusterUid | string | `""` | unique cluster identifier, if none is provided this value will be infered at time of helm install |
| worker.config.http2 | bool | `true` | connect using HTTP/2 if the server supports it (experimental) |
| worker.config.limit | string | `nil` | Maximum number of flow runs to start simultaneously (default: unlimited) |
| worker.config.prefetchSeconds | int | `10` | when querying for runs, how many seconds in the future can they be scheduled |
| worker.config.queryInterval | int | `5` | how often the worker will query for runs |
| worker.config.type | string | `"kubernetes"` | specify the worker type |
| worker.config.workPool | string | `""` | the work pool that your started worker will poll. |
| worker.config.workQueues | list | `[]` | one or more work queue names for the worker to pull from. if not provided, the worker will pull from all work queues in the work pool |
| worker.containerSecurityContext.allowPrivilegeEscalation | bool | `false` | set worker containers' security context allowPrivilegeEscalation |
| worker.containerSecurityContext.readOnlyRootFilesystem | bool | `true` | set worker containers' security context readOnlyRootFilesystem |
| worker.containerSecurityContext.runAsNonRoot | bool | `true` | set worker containers' security context runAsNonRoot |
| worker.containerSecurityContext.runAsUser | int | `1001` | set worker containers' security context runAsUser |
| worker.extraContainers | list | `[]` | additional sidecar containers |
| worker.extraEnvVars | list | `[]` | array with extra environment variables to add to worker nodes |
| worker.extraEnvVarsCM | string | `""` | name of existing ConfigMap containing extra env vars to add to worker nodes |
| worker.extraEnvVarsSecret | string | `""` | name of existing Secret containing extra env vars to add to worker nodes |
| worker.extraVolumeMounts | list | `[]` | array with extra volumeMounts for the worker pod |
| worker.extraVolumes | list | `[]` | array with extra volumes for the worker pod |
| worker.image.debug | bool | `false` | enable worker image debug mode |
| worker.image.prefectTag | string | `"2-python3.11-kubernetes"` | prefect image tag (immutable tags are recommended) |
| worker.image.pullPolicy | string | `"IfNotPresent"` | worker image pull policy |
| worker.image.pullSecrets | list | `[]` | worker image pull secrets |
| worker.image.repository | string | `"prefecthq/prefect"` | worker image repository |
| worker.livenessProbe.config.failureThreshold | int | `3` | The number of consecutive failures allowed before considering the probe as failed. |
| worker.livenessProbe.config.initialDelaySeconds | int | `10` | The number of seconds to wait before starting the first probe. |
| worker.livenessProbe.config.periodSeconds | int | `10` | The number of seconds to wait between consecutive probes. |
| worker.livenessProbe.config.successThreshold | int | `1` | The minimum consecutive successes required to consider the probe successful. |
| worker.livenessProbe.config.timeoutSeconds | int | `5` | The number of seconds to wait for a probe response before considering it as failed. |
| worker.livenessProbe.enabled | bool | `false` |  |
| worker.nodeSelector | object | `{}` | node labels for worker pods assignment |
| worker.podAnnotations | object | `{}` | extra annotations for worker pod |
| worker.podLabels | object | `{}` | extra labels for worker pod |
| worker.podSecurityContext.fsGroup | int | `1001` | set worker pod's security context fsGroup |
| worker.podSecurityContext.runAsNonRoot | bool | `true` | set worker pod's security context runAsNonRoot |
| worker.podSecurityContext.runAsUser | int | `1001` | set worker pod's security context runAsUser |
| worker.priorityClassName | string | `""` | priority class name to use for the worker pods; if the priority class is empty or doesn't exist, the worker pods are scheduled without a priority class |
| worker.replicaCount | int | `1` | number of worker replicas to deploy |
| worker.resources.limits | object | `{"cpu":"1000m","memory":"1Gi"}` | the requested limits for the worker container |
| worker.resources.requests | object | `{"cpu":"100m","memory":"256Mi"}` | the requested resources for the worker container |
| worker.selfHostedApiConfig.accountId | string | `""` | prefect account ID |
| worker.selfHostedApiConfig.apiKeySecret.key | string | `"key"` | prefect API secret key |
| worker.selfHostedApiConfig.apiKeySecret.name | string | `"prefect-api-key"` | prefect API secret name |
| worker.selfHostedApiConfig.apiUrl | string | `""` | prefect API url (PREFECT_API_URL) |
| worker.selfHostedApiConfig.uiUrl | string | `""` | self hosted UI url |
| worker.selfHostedApiConfig.workspaceId | string | `""` | prefect workspace ID |
| worker.serverApiConfig.apiUrl | string | `""` | prefect API url (PREFECT_API_URL) |
| worker.serverApiConfig.uiUrl | string | `"http://localhost:4200"` | prefect UI url |
| worker.tolerations | list | `[]` | tolerations for worker pods assignment |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.11.2](https://github.com/norwoodj/helm-docs/releases/v1.11.2)
