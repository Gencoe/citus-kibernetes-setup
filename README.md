# citus-kibernetes-setup

This repository provides a step-by-step guide and scripts to set up a local Citus (PostgreSQL extension for distributed databases) environment using Minikube, Helm, and Docker.

---

## 🚀 Setup Steps

### 1. Start Minikube with Docker Driver

```bash
minikube start --driver=docker
```

---

### 2. Install Citus with Helm

This script installs the `cert-manager` and `citus` via Helm charts.

**File:** `helm/citus-install.sh`

```bash
#!/bin/bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager --create-namespace --namespace cert-manager \
  --version v1.5.4 jetstack/cert-manager --set installCRDs=true --wait --debug

helm repo add prates-charts https://sergioprates.github.io/prates-charts/
helm install citus prates-charts/citus --debug --wait
```

---

### 3. Create Docker Network

Create a network for containers to communicate.

**File:** `docker/create-network.sh`

```bash
#!/bin/bash
docker network create citus-network
```

---

### 4. Start Worker Containers

Start two worker nodes.

**File:** `docker/worker1-start.sh`

```bash
#!/bin/bash
docker run -d --name worker1 --network citus-network -e POSTGRES_PASSWORD=mypassword citusdata/citus
```

**File:** `docker/worker2-start.sh`

```bash
#!/bin/bash
docker run -d --name worker2 --network citus-network -e POSTGRES_PASSWORD=mypassword citusdata/citus
```

---

### 5. Start Coordinator Container

Run the Citus coordinator container.

**File:** `docker/coordinator-start.sh`

```bash
#!/bin/bash
docker rm -f citus 2>/dev/null
docker run -d --name coordinator --network citus-network -p 5500:5432 -e POSTGRES_PASSWORD=mypassword citusdata/citus
```

---

### 6. Initialize Database

Access the coordinator with:

```bash
docker exec -it coordinator psql -U postgres
```

Then run the following SQL script to create and shard a table.

**File:** `sql/init-db.sql`

```sql
-- Connect to coordinator container with:
-- docker exec -it coordinator psql -U postgres

-- Optional: Add worker nodes manually if not automatically discovered
-- SELECT * from citus_add_node('worker1', 5432);
-- SELECT * from citus_add_node('worker2', 5432);

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT,
    email TEXT
);

SELECT create_distributed_table('users', 'id');
```
