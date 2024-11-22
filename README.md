### **README: API Data Pipeline to InfluxDB**  

---

### **Project Overview**  
This project establishes two robust data pipelines to process and analyze real-time API data.  
- **Path 1**: Stream data from an API → Flume → Kafka → HDFS for batch analysis.  
- **Path 2**: Stream data from an API → Flume → Kafka → InfluxDB → Grafana for time-series visualization.  
  ![stream (1)](https://github.com/user-attachments/assets/341fb245-e479-4788-9ab7-2aac8e05b3d1)
#### **Key Features**  
1. **Data Generation**: Fetching population data from a public API.  
2. **Data Streaming**: Utilizing Apache Flume and Apache Kafka for streaming.  
3. **Data Storage & Visualization**:  
   - HDFS for large-scale batch storage and analysis.  
   - InfluxDB for time-series data analysis, visualized in Grafana
---

### **System Architecture**  
#### **Components**  
1. **API**: Public API as the data source (e.g., Population API).  
2. **Flume**: Collects data and streams it to Kafka.  
3. **Kafka**: Acts as a message broker.  
4. **HDFS**: Stores data for batch processing (Path 1).  
5. **InfluxDB**: Time-series database for real-time data (Path 2).  
6. **Grafana**: Visualizes metrics in dashboards connected to InfluxDB.  

---

### **Prerequisites**  
Ensure the following are installed:  
- Python 3.7+  
- Apache Flume  
- Apache Kafka  
- InfluxDB  
- Grafana  
- Apache Hadoop (HDFS)  
- PySpark  

---

### **Setup Instructions**  

#### **Step 1: Fetch Data from API**  
Use the provided Python script to fetch and save data from the API. Modify the `api_url` and `interval` as needed.  
```python
# Example API URL
api_url = "https://datausa.io/api/data?drilldowns=Nation&measures=Population"

# Fetch and save API data every 60 seconds
fetch_and_save_api_data(api_url, interval=60)
```  

---

#### **Step 2: Configure Flume Agent**  
##### Path 1: Flume → Kafka  
Create a configuration file `to_kafka.conf` with the following setup:  
```properties
# Flume Agent Configuration for Kafka
agent.sources = src_1
agent.sinks = kafka_sink
agent.channels = mem_channel

# Source Configuration
agent.sources.src_1.type = spooldir
agent.sources.src_1.spoolDir = /path/to/local/data
agent.sources.src_1.channels = mem_channel

# Sink Configuration
agent.sinks.kafka_sink.type = org.apache.flume.sink.kafka.KafkaSink
agent.sinks.kafka_sink.kafka.bootstrap.servers = localhost:9092
agent.sinks.kafka_sink.kafka.topic = data_from_flume

# Channel Configuration
agent.channels.mem_channel.type = memory
```

Run the agent:  
```bash
$FLUME_HOME/bin/flume-ng agent --conf conf --conf-file to_kafka.conf --name agent -Dflume.root.logger=DEBUG,console
```  

##### Path 2: Kafka → HDFS  
Create `kafka_hdfs.conf`:  
```properties
# Flume Agent Configuration for HDFS
agent2.sources = source2
agent2.sinks = sink2
agent2.channels = channel2

agent2.sources.source2.type = org.apache.flume.source.kafka.KafkaSource
agent2.sources.source2.kafka.bootstrap.servers = localhost:9092
agent2.sources.source2.kafka.topics = data_from_flume

agent2.sinks.sink2.type = hdfs
agent2.sinks.sink2.hdfs.path = /user/bigdata/data_from_kafka

agent2.channels.channel2.type = memory
```  

Run the agent:  
```bash
$FLUME_HOME/bin/flume-ng agent --conf conf --conf-file kafka_hdfs.conf --name agent2 -Dflume.root.logger=DEBUG,console
```  

---

#### **Step 3: Kafka Setup**  
1. Start Zookeeper and Kafka:  
   ```bash
   bin/zookeeper-server-start.sh config/zookeeper.properties
   bin/kafka-server-start.sh config/server.properties
   ```  
2. Create Kafka topics:  
   ```bash
   bin/kafka-topics.sh --create --topic data_from_flume --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
   ```  
3. Verify topics:  
   ```bash
   bin/kafka-topics.sh --list --bootstrap-server localhost:9092
   ```  

---

#### **Step 4: PySpark to InfluxDB**  
Use the provided PySpark script to consume data from Kafka and write to InfluxDB. Update `influxdb_url`, `bucket`, and `token` as per your InfluxDB setup.  

---

#### **Step 5: Visualization in Grafana**  
1. Connect Grafana to InfluxDB:  
   - Add a new InfluxDB data source.  
   - Provide the database URL and credentials.  
2. Create dashboards to visualize time-series data (e.g., Population trends).  

---

### **Project Structure**  
```plaintext
data_pipeline_project/
├── data_gen/               # Directory for API data storage
├── flume_configs/          # Flume configuration files
│   ├── to_kafka.conf
│   └── kafka_hdfs.conf
├── scripts/                # Python scripts
│   ├── fetch_api_data.py
│   └── kafka_influxdb.py
└── README.md               # Project documentation
```  

---

### **Future Work**  
- Extend to other public APIs for diverse data sources.  
- Implement advanced data transformation using Spark.  
- Integrate with Elasticsearch for full-text search capabilities.  

---

**Contributors**:  
Aya Yahia  
[GitHub](https://github.com/aya-yahia-1november) | [LinkedIn](https://linkedin.com/in/aya-yahia-37522a217)  

