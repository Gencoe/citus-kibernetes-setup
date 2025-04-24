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

### 2. Namestitev citus-cluster z Helm

```bash
helm install citus-cluster .
```

Preverjanje ali delujejo pods

```bash
kubectl get pods
```

---

### 3. Registracija worker vozlišči z koordinatorja

Vhod v Coordinator pod.

```bash
kubectl exec -it <ime-koordinatorja> -- bash
psql -U postgres
```

Ime Koordinatorja lahko se preveri z to skripto.

```bash
kubectl get pods
```

PgSQL skripta za ustvarjanje podatkovno bazo in citus extension

```sql
CREATE DATABASE mydb;
\c mydb
CREATE EXTENSION citus;
```

PgSQL skripta za nastavitev coordinator host

```sql
SELECT citus_set_coordinator_host('coordinator.default.svc.cluster.local', 5432);
```

PgSQL skripta za dodajanje worker vozlišče

```sql
SELECT citus_add_node('worker.default.svc.cluster.local', 5432)
/*ali*/
SELECT * FROM master_add_node('worker.default.svc.cluster.local', 5432);
```

PgSQL skripta za ustvaranje tabelo in distribucijo (sharding)

```sql
CREATE TABLE t_shard (
id          serial,
shard_key   int,
n           int,
placeholder char(100) DEFAULT 'a');

SELECT create_distributed_table('t_shard', 'shard_key');
```

PgSQL skripta za polnenje tabelo

```sql
INSERT INTO t_shard (shard_key, n) 
SELECT 	id % 16, random()*100000 
FROM 		generate_series(1, 5000000) AS id;
```