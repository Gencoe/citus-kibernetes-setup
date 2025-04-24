# citus-kibernetes-setup

Ta repozitorija ponuja vodnik po koraki za ustvarjanje lokalno Citus okolje z Minikube, Helm in Docker.

---

## Predpogoji

- [Docker](https://docs.docker.com/get-docker/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Helm](https://helm.sh/docs/intro/install/)

## Setup koraki

### 1. Zagon minikubea z docker driver

```bash
minikube start --driver=docker
```

---

### 2. Namestitev Citus z Helm

Ta skripta namesti `cert-manager` in `citus` skozi Helm charts.

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager --create-namespace --namespace cert-manager \
  --version v1.5.4 jetstack/cert-manager --set installCRDs=true --wait --debug

helm repo add prates-charts https://sergioprates.github.io/prates-charts/
helm install citus prates-charts/citus --debug --wait
```

---

### 3. Docker Omrežje

Omrezje za komunikacijo med kontejnerji.

```bash
docker network create citus-network
```

---

### 4. Worker vozlišči

Kreiranje worker vozlišči.

```bash
docker run -d --name worker1 --network citus-network -e POSTGRES_PASSWORD=mypassword citusdata/citus
```

```bash
docker run -d --name worker2 --network citus-network -e POSTGRES_PASSWORD=mypassword citusdata/citus
```

---

### 5. Coordinaor Kontejner

Zagon koordinatorja.

**File:** `docker/coordinator-start.sh`

```bash
docker rm -f citus 2>/dev/null
docker run -d --name coordinator --network citus-network -p 5500:5432 -e POSTGRES_PASSWORD=mypassword citusdata/citus
```

---

### 6. Inicijalizacija podatkovne baze

Access the coordinator with:

```bash
docker exec -it coordinator psql -U postgres
```

Vzpostavljanje konekcijo z coordinator kontejnerja

```bash
docker exec -it coordinator psql -U postgres
```

SQL skripta za kreiranje in ločitev tabelo

```sql
-- ročno dodajanje worker vozlišči
-- SELECT * from citus_add_node('worker1', 5432);
-- SELECT * from citus_add_node('worker2', 5432);

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT,
    email TEXT
);

SELECT create_distributed_table('users', 'id');
```
