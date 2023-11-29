# uds-capability-mattermost
Contains both the Mattermost Operator and a Mattermost component

Bigbang [Mattermost Operator](https://repo1.dso.mil/big-bang/product/packages/mattermost-operator) deployed via flux by zarf

Bigbang [Mattermost](https://repo1.dso.mil/big-bang/product/packages/mattermost) deployed via flux by zarf

## Deployment Prerequisites

### Resources
- Minimum compute requirements for single node deployment are at LEAST 64 GB RAM and 32 virtual CPU threads (aws `m6i.8xlarge` instance type should do)
- k3d installed on machine

#### General

- Create `mattermost` namespace
- Label `mattermost` namespace with `istio-injection: enabled`

#### Database

- A Postgres database is running on port `5432` and accessible to the cluster
- This database can be logged into via the username `mattermost`
- This database instance has a psql database created named `mattermostdb`
- The `mattermost` user has read/write access to the above mentioned database
- Create `mattermost-postgres` service in `mattermost` namespace that points to the psql database
- Create `mattermost-postgres` secret in `mattermost` namespace with the keys `DB_CONNECTION_STRING` and `DB_CONNECTION_CHECK_URL` that contains connection the string to the for the psql database. Example connection string `postgres://mattermost:###ZARF_VAR_POSTGRES_DB_PASSWORD###@mattermost-postgres.mattermost.svc.cluster.local:5432/mattermostdb?connect_timeout=10&sslmode=disable`

#### Object Storage

- Create the secret `mattermost-object-store` in the `mattermost` namespace with the following keys:
  - An example for in-cluster Minio can be found in this repository at the path `utils/pkg-deps/mattermost/minio/secret.yaml`
  - Secret needs to contain the `accessKey` and `secretKey` for the object storage.
- Create a bucket called `mattermost-bucket`
- Create `mattermost-object-store` service in `mattermost` namespace that points to the object store url.

## Deploy

### Use zarf to login to the needed registries i.e. registry1.dso.mil

```bash
# Download Zarf
make build/zarf

# Login to the registry
set +o history

# registry1.dso.mil (To access registry1 images needed during build time)
export REGISTRY1_USERNAME="YOUR-USERNAME-HERE"
export REGISTRY1_TOKEN="YOUR-TOKEN-HERE"
echo $REGISTRY1_TOKEN | build/zarf tools registry login registry1.dso.mil --username $REGISTRY1_USERNAME --password-stdin

set -o history
```

### Build and Deploy Everything via Makefile and local package

```bash
# This will run make build/all, make cluster/reset, and make deploy/all. Follow the breadcrumbs in the Makefile to see what and how its doing it.
make all
```

## Declare This Package In Your UDS Bundle
Below is an example of how to use this projects zarf package in your UDS Bundle

```yaml
kind: UDSBundle
metadata:
  name: example-bundle
  description: An Example UDS Bundle
  version: 0.0.1
  architecture: amd64

zarf-packages:
  # Mattermost Operator with a Mattermost instance
  - name: mattermost
    repository: ghcr.io/defenseunicorns/uds-capability/mattermost
    ref: x.x.x
```
