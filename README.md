# PostgreSQL test application

This is a sample application that uses a PostgreSQL database and consumes the DB configuration via a Kubernetes ConfigMap and Secret.

This allows the configuration to be changed easily based on environment the application is deployed in.

For example, the ConfigMap and Secret when using an AWS RDS instance are as follows
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  annotations:
    kasten.io/config: dataservice
  name: dbconfig
data:
  dataservice.type: postgres
  postgres.manager: awsrds
  postgres.host: dbendpoint.adsadasa.us-west-2.rds.amazonaws.com
  postgres.databases: mypgsqldb
  postgres.user: postgres
  postgres.secret: dbcreds # name of K8s secret in the same namespace

---

apiVersion: v1
kind: Secret
metadata:
  name: dbcreds
type: Opaque
data:
  password: <BASE64 encoded password>
  ```

## Docker Image

The application is automatically built and published as a Docker image to GitHub Container Registry when changes are pushed to the main branch.

### Using the Docker Image

Pull the image:
```bash
docker pull ghcr.io/tribock/pgtest:latest
```

Run the container with PostgreSQL environment variables:
```bash
docker run -p 8080:8080 \
  -e PG_HOST=your-postgres-host \
  -e PG_DBNAME=your-database \
  -e PG_USER=your-username \
  -e PG_PASSWORD=your-password \
  -e PG_SSL=disable \
  ghcr.io/tribock/pgtest:latest
```

### Environment Variables

The application requires the following environment variables:
- `PG_HOST`: PostgreSQL host address
- `PG_DBNAME`: Database name
- `PG_USER`: Database username
- `PG_PASSWORD`: Database password
- `PG_SSL`: SSL mode (disable, require, verify-ca, verify-full)

## Build/Package application
```bash
make clean
make container
make push
```

## Deployment into Kubernetes
```bash
# Set namespace to deploy into
export NAMESPACE=pgtestrds
kubectl create namespace ${NAMESPACE}
kubectl apply -f deploy/. --namespace ${NAMESPACE}
```

## Testing the application
Use `kubectl proxy` to connect to the service in the cluster
```
kubectl proxy&
```
### Get Service and Database Information
```bash
http://127.0.0.1:8001/api/v1/namespaces/pgtestrds/services/pgtestapp:8080/proxy/
```

### Count rows
```bash
http://127.0.0.1:8001/api/v1/namespaces/pgtestrds/services/pgtestapp:8080/proxy/count
```

### Insert a new row
```bash
http://127.0.0.1:8001/api/v1/namespaces/pgtestrds/services/pgtestapp:8080/proxy/insert
```


