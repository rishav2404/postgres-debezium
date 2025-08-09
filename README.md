# PostgreSQL Debezium CDC Setup
<img width="1259" height="696" alt="image" src="https://github.com/user-attachments/assets/8942a431-394e-4563-8345-817fb949e3d6" />

A complete Change Data Capture (CDC) solution using PostgreSQL, Debezium, Kafka, and Schema Registry. This setup captures real-time database changes and streams them to Kafka topics for downstream processing.

## 🏗️ Architecture Overview

```
PostgreSQL → Debezium Connector → Kafka → Schema Registry → Consumer
```

- **PostgreSQL**: Source database with CDC enabled
- **Debezium**: CDC connector that captures database changes
- **Kafka**: Message broker for streaming changes
- **Schema Registry**: Manages Avro schemas for message serialization
- **Kafka Connect**: Framework for Debezium connector


### 1. Start the Infrastructure

```bash
# Start all services
podman compose up -d

# Verify services are running
podman compose ps
```

### 2. Configure PostgreSQL for CDC

Connect to PostgreSQL and set up the required table:

```bash
# Access PostgreSQL container
podman exec -it postgres-debezium-postgres-1 /bin/sh

# Connect to database
psql -U rishav -d rishavdb -W
# Password: rishav

# Create the student table
CREATE TABLE student (
    id int primary key, 
    name varchar
);

# Enable CDC for the table
ALTER TABLE public.student REPLICA IDENTITY FULL;
```

### 3. Deploy Debezium Connector

Register the Debezium connector with Kafka Connect:

```bash
# Deploy the connector
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" \
  127.0.0.1:8083/connectors/ --data "@debezium.json"
```

### 4. Verify CDC is Working

Check the connector status:
```bash
curl http://localhost:8083/connectors/postgres-connector/status
```

## 📊 Testing CDC

### Insert Data
```sql
INSERT INTO student (id, name) VALUES (1, 'Rishav');
```

### Update Data
```sql
UPDATE student SET name='Hobo' WHERE id=1;
```

### Delete Data
```sql
DELETE FROM student WHERE id=1;
```

### Consume CDC Events

```bash
# Consume messages from the beginning
podman run --tty --network postgres-debezium_default \
  confluentinc/cp-kafkacat \
  kafkacat -b kafka:9092 -C \
  -s key=s -s value=avro \
  -r http://schema-registry:8081 \
  -t postgres.public.student \
  -o beginning -e
```
or 

```
podman run --tty --network postgres-debezium_default confluentinc/cp-kafkacat kafkacat -b kafka:9092 -C -s key=s -s value=avro -r http://schema-registry:8081 -t postgres.public.student
```

## 🔧 Configuration Details

### PostgreSQL Configuration
- **Host**: localhost:5432
- **Database**: rishavdb
- **User**: rishav
- **Password**: rishav
- **CDC Enabled**: Yes (REPLICA IDENTITY FULL)

### Kafka Configuration
- **Bootstrap Server**: kafka:9092
- **Schema Registry**: http://schema-registry:8081

### Debezium Connector
- **Connector Name**: postgres-connector
- **Database Server**: postgres
- **Database Port**: 5432
- **Database Name**: rishavdb
- **Table**: public.student

## 📋 Service URLs

| Service | URL | Description |
|---------|-----|-------------|
| PostgreSQL | `localhost:5432` | Database |
| Kafka | `localhost:9092` | Message broker |
| Schema Registry | `localhost:8081` | Schema management |
| Kafka Connect | `localhost:8083` | Connector management |
| Debezium UI | `localhost:8083` | Connector status |

## 🛠️ Troubleshooting

### Common Issues

1. **"Network not found" error**
   ```bash
   # Use the correct network name
   --network postgres-debezium_default
   ```

2. **"Leader not available" error**
   - Wait 30-60 seconds for Kafka to fully initialize
   - Check service status: `podman compose ps`

3. **"Topic not found"**
   - Ensure the table exists and has CDC enabled
   - Check connector status: `curl http://localhost:8083/connectors/postgres-connector/status`

### Debug Commands

```bash
# Check all services
podman compose ps

# View logs
podman compose logs -f [service-name]

# List Kafka topics
podman run --tty --network postgres-debezium_default \
  confluentinc/cp-kafkacat \
  kafkacat -b kafka:9092 -L

# Check connector status
curl http://localhost:8083/connectors/postgres-connector/status
```

## 📚 Additional Resources

- [Debezium Documentation](https://debezium.io/documentation/)
- [Kafka Connect Guide](https://docs.confluent.io/platform/current/connect/index.html)
- [PostgreSQL CDC Setup](https://debezium.io/documentation/reference/stable/connectors/postgresql.html)

## 🧹 Cleanup

To stop all services:
```bash
podman compose down
```

To remove all data:
```bash
podman compose down -v
```
or 
```bash
podman system prune --all --volumes
```



<img width="1651" height="442" alt="image" src="https://github.com/user-attachments/assets/365a7d58-b202-484c-8a9b-5668960fcd01" />
