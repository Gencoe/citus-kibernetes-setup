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

Kreiranje in prvi zagon worker vozlišči.

```bash
docker run -d --name worker1 --network citus-network -e POSTGRES_PASSWORD=mypassword citusdata/citus
docker run -d --name worker2 --network citus-network -e POSTGRES_PASSWORD=mypassword citusdata/citus
```

Vsaki naslednji zagon worker vozlišči.

```bash
docker start worker1
docker start worker2
```

---

### 5. Zagon PostgreSQL z Citus

PostgreSQL se zažene s Citus na vrat 5500

```bash
docker run -d --name citus -p 5500:5432 -e POSTGRES_PASSWORD=mypassword citusdata/citus
```

Vsaki naslednji zagon citusa

```bash
docker start citus
```

---

### 6. Inicijalizacija podatkovne baze

Vzpostavljanje konekcijo z coordinator kontejnerja.

```bash
docker exec -it citus psql -U postgres
```

PgSQL skripta za kreiranje in ločenje (sharding) tabelo.

```sql
CREATE TABLE t_shard (
id          serial,
shard_key   int,
n           int,
placeholder char(100) DEFAULT 'a');

SELECT create_distributed_table('t_shard', 'shard_key');
```

---

### Zaslonski posnetki
