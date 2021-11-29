# MongoDB-Backup Helm chart

The MongoDB-Backup Helm chart uses the [Helm](https://helm.sh) package manager
to bootstrap an MongoDB-Backup Cron-Job on a [Kubernetes](http://kubernetes.io)
cluster.

## Prerequisites

- Helm v2 or later
- Kubernetes 1.4+
- (Optional) PersistentVolume (PV) provisioner support in the underlying
  infrastructure

## Install the chart

1. Add the Helm repository:

   ```bash
   helm repo add scandio https://scandio.github.io/helm-charts
   ```

2. Run the following command, providing a name for your release:

   ```bash
   helm upgrade --install my-release scandio/mongodb-backup
   ```

   > **Tip**: `--install` can be shortened to `-i`.

   This command deploys MongoDB-Backup on the Kubernetes cluster using the
   default configuration. To find parameters you can configure during
   installation, see [Configure the chart](#configure-the-chart).

   > **Tip**: To view all Helm chart releases, run `helm list`.

## Uninstall the chart

To uninstall the `my-release` deployment, use the following command:

```bash
helm uninstall my-release
```

This command removes all the Kubernetes components associated with the chart and
deletes the release.

## Configure the chart

The following table lists configurable parameters, their descriptions, and their
default values stored in `values.yaml`.

| Parameter | Description | Default |
|---|---|---|
| image.repository | Image repository url | mongo |
| image.tag | Image tag | 4.4-focal |
| serviceAccount.create | It will create service account | true |
| serviceAccount.annotations | Service account annotations | {} |
| serviceAccount.name | Service account name | "" |
| podAnnotations | Annotations for the pods | {} |
| resources | Resources requests and limits  | {} |
| backup.host | Hostname database | "" |
| backup.username | Username for authentication with the database | "" |
| backup.password | Password for authentication with the database | "" |
| backup.authenticationDatabase | The database to authenticate with | "" |
| backup.schedule | Schedule to run jobs in cron format | `0 0 * * *` |
| backup.startingDeadlineSeconds | Deadline in seconds for starting the job if it misses its scheduled time for any reason | `nil` |
| backup.persistence.enabled | Boolean to enable and disable persistance | false |
| backup.persistence.storageClass | If set to "-", storageClassName: "", which disables dynamic provisioning. If undefined (the default) or set to null, no storageClassName spec is set, choosing the default provisioner.  (gp2 on AWS, standard on GKE, AWS & OpenStack |  |
| backup.persistence.annotations | Annotations for volumeClaimTemplates | nil |
| backup.persistence.accessMode | Access mode for the volume | ReadWriteOnce |
| backup.persistence.size | Storage size | 8Gi |
| backup.gcs | Google Cloud Storage config | `nil`
| backup.azure | Azure Blob Storage config | `nil`
| backup.s3 | Amazon S3 (or compatible) config | `nil`

To configure the chart, do either of the following:

- Specify each parameter using the `--set key=value[,key=value]` argument to
  `helm upgrade --install`. For example:

  ```bash
  helm upgrade --install my-release \
    --set backup.persistence.enabled=true,backup.persistence.size=200Gi \
      scandio/mongodb-backup
  ```

  This command enables persistence and changes the size of the requested data
  volume to 200GB.

- Provide a YAML file that specifies the parameter values while installing the
  chart. For example, use the following command:

  ```bash
  helm upgrade --install my-release -f values.yaml scandio/mongodb-backup
  ```

  > **Tip**: Use the default [values.yaml](values.yaml).

## Back up and restore

Before proceeding, please read
[Back Up and Restore with MongoDB Tools](https://docs.mongodb.com/manual/tutorial/backup-and-restore-tools/).

### Backups

The [`mongodb-backup-cronjob`](./chart/templates/mongodb-backup.yaml) runs on
the configured schedule. One can create a job from the backup cronjob on demand
as follows:

```sh
kubectl create job --from=cronjobs/mongodb-backup mongodb-backup-$(date +%Y%m%d%H%M%S)
```

#### Backup Storage

The backup process consists of an init-container that writes the backup to a
local volume, which is by default an `emptyDir`, shared to the runtime container
which uploads the backup to the configured object store.

In order to avoid filling the node's disk space, it is recommended to set a
sufficient `ephemeral-storage` request or enable persistence, which allocates a
PVC.

Furthermore, if no object store provider is available, one can simply use the
PVC as the final storage destination when `persistence` is enabled.

# Acknowledgements

Nearly all of this code is from https://github.com/influxdata/helm-charts namely
the `influx` Helm-Chart. Without their super awsome configuration I wouldn't
have made the script as it is now.
