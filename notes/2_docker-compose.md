This is a **Docker Compose configuration**: it describes *which services we want, how they communicate, what configuration they need, and what data should persist*.

---

## `services:`

* **Purpose:** Defines all the **containers/services** that Docker Compose should create and run.
* Everything underneath (`kafka`, `postgres`, `airflow-webserver`, etc.) is a separate service/container definition.
* **Remember:** "What components make up my application?"

---

# 1. Zookeeper

```yaml
zookeeper:
```

* **Purpose:** Runs Apache ZooKeeper, which Kafka versions like ours use to manage **Kafka cluster metadata and coordination**.
* In this project, Kafka depends on ZooKeeper to operate.

### `image`

```yaml
image: confluentinc/cp-zookeeper:7.4.0
```

* **Purpose:** Specifies the Docker image used to create the container.
* `confluentinc/cp-zookeeper` = ZooKeeper image from Confluent.
* `7.4.0` = image version.
* **Remember:** "Which pre-built software should Docker run?"

### `environment`

```yaml
environment:
```

* **Purpose:** Passes environment variables/configuration into the container.

### `ZOOKEEPER_CLIENT_PORT: 2181`

* **Purpose:** Tells ZooKeeper which port clients (such as Kafka) use to connect.
* `2181` is ZooKeeper's standard client port.

### `ZOOKEEPER_TICK_TIME: 2000`

* **Purpose:** Sets ZooKeeper's basic timing unit to **2000 ms (2 seconds)**.
* Used internally for coordination/heartbeat timing.

### `ports`

```yaml
ports:
  - "2181:2181"
```

* **Purpose:** Maps a port from our **host machine → container**.
* Format: `"HOST:CONTAINER"`
* `2181:2181` means host port 2181 connects to container port 2181.
* **Remember:** "Make this container port accessible from outside the container."

---

# 2. Kafka

```yaml
kafka:
```

* **Purpose:** Runs the Kafka broker that handles **producing, storing, and consuming events/messages**.

### `container_name`

```yaml
container_name: kafka
```

* **Purpose:** Gives the container an explicit name.
* Useful when we want to refer to the container directly.
* **Not required**; Docker can generate a name automatically.

### `depends_on`

```yaml
depends_on:
  - zookeeper
```

* **Purpose:** Tells Docker Compose that Kafka depends on ZooKeeper.
* Docker starts ZooKeeper before Kafka.
* **Important:** It controls startup order, **not whether ZooKeeper is actually ready**.

### Kafka ports

```yaml
ports:
  - "9092:9092"
  - "29092:29092"
```

These provide **two ways to connect to Kafka**:

* `9092` → Kafka communication inside the Docker network.
* `29092` → Kafka access from our host machine.

This is why we have two listeners later.

---

### `KAFKA_BROKER_ID: 1`

* **Purpose:** Gives this Kafka broker a unique ID.
* `1` means this is broker #1.
* In a multi-broker cluster, each broker needs a different ID.

### `KAFKA_ZOOKEEPER_CONNECT`

```yaml
KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
```

* **Purpose:** Tells Kafka where ZooKeeper is.
* `zookeeper` = Docker Compose service name.
* `2181` = ZooKeeper port.

Think:

> Kafka → "Connect to the container called `zookeeper` on port `2181`."

---

### `KAFKA_LISTENERS`

```yaml
KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,PLAINTEXT_HOST://0.0.0.0:29092
```

* **Purpose:** Defines **where Kafka listens for incoming connections**.
* `0.0.0.0` means "listen on all network interfaces."

we have two listeners:

```text
PLAINTEXT       → port 9092
PLAINTEXT_HOST  → port 29092
```

---

### `KAFKA_ADVERTISED_LISTENERS`

```yaml
KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://host.docker.internal:29092
```

* **Purpose:** Tells Kafka clients **what address they should use to connect back to Kafka**.

This is extremely important.

Inside Docker:

```text
kafka:9092
```

From our host:

```text
host.docker.internal:29092
```

**Key distinction:**

> `LISTENERS` = where Kafka listens.
> `ADVERTISED_LISTENERS` = what address Kafka tells clients to use.

---

### `KAFKA_LISTENER_SECURITY_PROTOCOL_MAP`

```yaml
KAFKA_LISTENER_SECURITY_PROTOCOL_MAP:
  PLAINTEXT:PLAINTEXT
  PLAINTEXT_HOST:PLAINTEXT
```

* **Purpose:** Maps each listener name to its security protocol.
* `PLAINTEXT` means **no encryption/authentication**.
* Fine for a local development environment.

---

### `KAFKA_INTER_BROKER_LISTENER_NAME`

```yaml
KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
```

* **Purpose:** Tells Kafka which listener brokers should use to communicate with each other.
* Relevant when we have multiple Kafka brokers.

---

### `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1`

* **Purpose:** Sets the replication factor for Kafka's internal consumer-offset topic.
* `1` = only one copy.
* Fine for a **single-broker development environment**.
* Production clusters normally use higher replication.

### `KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1`

* **Purpose:** Sets replication for Kafka's internal transaction-state topic.
* `1` = one copy.
* Again, suitable for our single-broker setup.

### `KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1`

* **Purpose:** Minimum number of in-sync replicas required for Kafka transactions.
* `1` allows the single broker to operate.

---

# 3. Debezium / Kafka Connect

```yaml
connect:
```

* **Purpose:** Runs Kafka Connect with Debezium.
* Used to **capture changes from databases and stream them into Kafka**.
* This is the **CDC (Change Data Capture)** part of our architecture.

### `image`

```yaml
image: debezium/connect:2.2
```

* Runs Debezium's Kafka Connect image.
* `2.2` = Debezium version.

### `depends_on`

```yaml
depends_on:
  - kafka
  - zookeeper
  - postgres
```

* Ensures Kafka, ZooKeeper, and PostgreSQL are started before Connect.

### `ports`

```yaml
"8083:8083"
```

* Exposes Kafka Connect's REST API.
* we can use this API to create/manage connectors.

---

### `BOOTSTRAP_SERVERS`

```yaml
BOOTSTRAP_SERVERS: 'kafka:9092'
```

* **Purpose:** Tells Kafka Connect where to initially connect to Kafka.
* `kafka:9092` = Kafka service inside Docker.

### `GROUP_ID`

```yaml
GROUP_ID: '1'
```

* **Purpose:** Identifies the Kafka Connect worker group.
* Connect workers with the same group ID can coordinate.

### `CONFIG_STORAGE_TOPIC`

```yaml
CONFIG_STORAGE_TOPIC: connect-configs
```

* **Purpose:** Kafka topic where Kafka Connect stores connector configurations.

### `OFFSET_STORAGE_TOPIC`

```yaml
OFFSET_STORAGE_TOPIC: connect-offsets
```

* **Purpose:** Stores connector offsets/progress.
* Important for knowing **where Debezium left off**.

### `STATUS_STORAGE_TOPIC`

```yaml
STATUS_STORAGE_TOPIC: connect-status
```

* **Purpose:** Stores connector/worker status information.

---

### `KEY_CONVERTER`

```yaml
KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter
```

* **Purpose:** Defines how Kafka message **keys** are converted between Kafka Connect data and JSON.

### `VALUE_CONVERTER`

```yaml
VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
```

* Same idea, but for the **message values**.

### `*_SCHEMAS_ENABLE: false`

```yaml
KEY_CONVERTER_SCHEMAS_ENABLE: 'false'
VALUE_CONVERTER_SCHEMAS_ENABLE: 'false'
```

* **Purpose:** Prevents Kafka Connect from including schema metadata in the JSON output.
* Makes the JSON payload simpler.

---

# 4. PostgreSQL

```yaml
postgres:
```

* **Purpose:** Runs PostgreSQL as the **source database** for our CDC pipeline.

### `POSTGRES_PASSWORD`

```yaml
POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

* **Purpose:** Sets PostgreSQL's password.
* `${POSTGRES_PASSWORD}` means the value comes from an **environment variable**, rather than being hardcoded here.

Same idea for:

```yaml
POSTGRES_USER
POSTGRES_DB
```

---

### PostgreSQL volume

```yaml
volumes:
  - ./docker/postgres/data:/var/lib/postgresql/data
```

* **Purpose:** Persists PostgreSQL data outside the container.
* Left side = our host directory.
* Right side = PostgreSQL's data directory inside the container.

Without persistent storage, deleting the container could delete our database data.

---

### `command`

```yaml
command: >
  postgres -c wal_level=logical
          -c max_wal_senders=10
          -c max_replication_slots=10
```

This is **especially important for our Debezium CDC setup**.

#### `wal_level=logical`

* Enables PostgreSQL **logical replication**.
* Debezium needs this to capture database changes.

#### `max_wal_senders=10`

* Allows PostgreSQL to have up to 10 WAL sender processes/connections.

#### `max_replication_slots=10`

* Allows up to 10 replication slots.
* Debezium uses a replication slot to track changes it still needs to consume.

So this entire `command` exists because:

> **PostgreSQL → Debezium → Kafka CDC requires logical replication.**

---

# 5. MinIO

```yaml
minio:
```

* **Purpose:** Runs MinIO, an **S3-compatible object storage system**.
* In our architecture, it can act as our local **data lake/object storage**.

### `command`

```yaml
server /data --console-address ":9001"
```

* Starts MinIO.
* `/data` = where objects are stored.
* `9001` = MinIO web console.

### `MINIO_ROOT_USER`

* **Purpose:** Admin username.

### `MINIO_ROOT_PASSWORD`

* **Purpose:** Admin password.

### Ports

```yaml
9000:9000
```

* MinIO API/object-storage endpoint.

```yaml
9001:9001
```

* MinIO web UI/console.

### Volume

```yaml
./docker/minio/data:/data
```

* Persists MinIO data on our machine.

---

# 6. Airflow Webserver

```yaml
airflow-webserver:
```

* **Purpose:** Runs the **Airflow web UI/API**.
* This is where we view DAGs, task status, logs, etc.

### `build`

```yaml
build:
  context: .
  dockerfile: dockerfile-airflow.dockerfile
```

Unlike `image:`, we're telling Docker:

> "Build my own Airflow image using this Dockerfile."

`context: .` = current project directory.

---

### `restart: always`

* **Purpose:** Automatically restart the container if it stops/crashes.

### `depends_on`

```yaml
depends_on:
  - airflow-scheduler
  - airflow-postgres
```

* Starts these services before the webserver.

---

### `AIRFLOW__CORE__LOAD_EXAMPLES=False`

* **Purpose:** Prevents Airflow from loading its example DAGs.
* Keeps our Airflow UI clean.

### `AIRFLOW__DATABASE__SQL_ALCHEMY_CONN`

* **Purpose:** Tells Airflow where its **metadata database** is.
* Airflow stores things like DAG/task states, execution history, connections, etc.

---

### Airflow volumes

```yaml
./docker/dags:/opt/airflow/dags
```

* our local DAG files → Airflow's DAG directory.

```yaml
./docker/logs:/opt/airflow/logs
```

* Stores Airflow logs.

```yaml
./docker/plugins:/opt/airflow/plugins
```

* Makes custom Airflow plugins available.

```yaml
./banking_dbt:/opt/airflow/banking_dbt
```

* Makes our dbt project accessible from Airflow.

```yaml
./banking_dbt/.dbt:/home/airflow/.dbt
```

* Gives Airflow access to our dbt configuration.

### `command: webserver`

* **Purpose:** Tells the container to start Airflow's webserver process.

---

# 7. Airflow Scheduler

```yaml
airflow-scheduler:
```

* **Purpose:** Runs the Airflow scheduler.
* The scheduler determines **which tasks/DAG runs need to be executed and when**.

This is different from the webserver:

```text
Webserver  → UI / monitoring
Scheduler  → actually schedules tasks
```

The same `build`, database connection, DAG/log/plugin/dbt volumes are there because the scheduler needs access to the same Airflow environment.

### `command: scheduler`

* Starts the Airflow scheduler process.

---

# 8. Airflow PostgreSQL

```yaml
airflow-postgres:
```

* **Purpose:** PostgreSQL database specifically for **Airflow's metadata**.

This is intentionally separate from:

```text
postgres
```

which is our **banking/source database**.

So we have:

```text
postgres
     ↓
Banking/source data

airflow-postgres
     ↓
Airflow metadata
```

### Why port `5433:5432`?

```yaml
ports:
  - "5433:5432"
```

Because our other PostgreSQL already uses host port `5432`.

So:

```text
Banking PostgreSQL
Host: 5432 → Container: 5432

Airflow PostgreSQL
Host: 5433 → Container: 5432
```

Inside their respective Docker networking, both PostgreSQL containers can still use `5432`.

---

# 9. Named volumes

```yaml
volumes:
  airflow_postgres_data:
  postgres_data:
```

* **Purpose:** Defines Docker-managed persistent volumes.
* `airflow_postgres_data` is actually used by `airflow-postgres`.


---

# 10. Network

```yaml
networks:
  default:
    name: banking-mds-net
```

* **Purpose:** Creates/uses a Docker network named `banking-mds-net`.
* This allows our containers to communicate with each other using **service names**.

For example:

```text
Kafka → zookeeper:2181
Connect → kafka:9092
Connect → postgres:5432
Airflow → airflow-postgres:5432
```

we don't need to know the container IP addresses.

---

# The most important mental model

Instead of memorizing all these properties individually, understand what the Compose file is doing:

```text
                    ┌──────────────┐
                    │  PostgreSQL  │
                    │ Source DB    │
                    └──────┬───────┘
                           │
                         CDC
                           ↓
                    ┌──────────────┐
                    │  Debezium    │
                    │ Kafka Connect│
                    └──────┬───────┘
                           │
                           ↓
                    ┌──────────────┐
                    │    Kafka     │
                    │ Message Bus  │
                    └──────┬───────┘
                           │
             ┌─────────────┴─────────────┐
             ↓                           ↓
         MinIO                         Airflow
      Data Lake                  Orchestration
                                         │
                                         ↓
                                        dbt
                                         │
                                         ↓
                                   Data Warehouse
```

And **this is the key reason these settings exist**:

| Configuration    | Main question it answers                             |
| ---------------- | ---------------------------------------------------- |
| `image`          | **What software should I run?**                      |
| `build`          | **How should I build my own container?**             |
| `environment`    | **What configuration does the software need?**       |
| `ports`          | **How can the host/other systems access it?**        |
| `depends_on`     | **What other services does this service depend on?** |
| `volumes`        | **What data/files should persist or be shared?**     |
| `command`        | **What should the container actually execute?**      |
| `container_name` | **What explicit name should the container have?**    |
| `restart`        | **What should happen if the container stops?**       |
| `networks`       | **How should containers communicate?**               |

### And one important distinction for our notes

Don't treat every line as a **Docker default/template**.

There are three categories here:

**1. Docker Compose syntax**

```text
services
image
build
environment
ports
volumes
depends_on
command
restart
networks
```

These are Compose configuration concepts.

**2. Software-specific configuration**

```text
KAFKA_BROKER_ID
KAFKA_ADVERTISED_LISTENERS
ZOOKEEPER_TICK_TIME
AIRFLOW__CORE__LOAD_EXAMPLES
POSTGRES_PASSWORD
MINIO_ROOT_USER
```

These exist because **Kafka, ZooKeeper, Airflow, PostgreSQL, and MinIO expect them**.

**3. Project-specific decisions**

```text
9092 / 29092
connect-configs
connect-offsets
banking-mds-net
./banking_dbt
AIRFLOW_DB_USER
```

