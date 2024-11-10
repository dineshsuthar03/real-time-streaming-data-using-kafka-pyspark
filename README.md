# Real-Time Data Processing using PySpark Streaming

## Tech Stack
- **Python 3.x**
- **PySpark**
- **Apache Spark**
- **Apache Kafka** (Optional for streaming data)
- **Jupyter Notebook** (Optional for data visualization)
- **EC2 (Ubuntu)** for deployment
- **Netcat** (for simulating real-time data via a socket server)

## Overview
This project demonstrates how to process real-time data using PySpark Streaming on an EC2 instance. The example includes simple anomaly detection on streaming data, which can be extended to more complex data pipelines. The setup uses Apache Kafka for data ingestion (optional) and PySpark for processing and analyzing the data.

## Step-by-Step Guide

### Step 1: Launch an EC2 Instance
1. Sign in to your AWS account and go to the EC2 Dashboard.
2. Click **Launch Instance** and choose an **Ubuntu AMI** (Amazon Machine Image).
3. Choose an instance type (e.g., `t2.micro` for testing).
4. Configure security groups to allow SSH (port 22), Jupyter (port 8888), and socket server (port 9999).
5. Generate or use an existing **Key Pair** to SSH into the instance.
6. Launch the instance.

### Step 2: Connect to Your EC2 Instance
Once your EC2 instance is running, open a terminal (Linux/macOS) or a tool like **PuTTY** (Windows) and run the following command to SSH into the instance:

```bash
ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
Step 3: Install Java
Apache Spark requires Java to run. Install Java 8 or 11:

bash

sudo apt update
sudo apt install openjdk-8-jdk -y
java -version
Step 4: Install Apache Spark
Download and install Apache Spark:

bash

cd ~
wget https://archive.apache.org/dist/spark/spark-3.3.2/spark-3.3.2-bin-hadoop3.2.tgz
tar -xvzf spark-3.3.2-bin-hadoop3.2.tgz
Set up environment variables by adding the following to your ~/.bashrc file:

bash

export SPARK_HOME=~/spark-3.3.2-bin-hadoop3.2
export PATH=$PATH:$SPARK_HOME/bin
export PYSPARK_PYTHON=python3
Reload the .bashrc file to apply the changes:

bash

source ~/.bashrc
Verify Spark installation:

bash

spark-submit --version
Step 5: Install Kafka (Optional)
If using Kafka as a data source, install Kafka:

bash

sudo apt install wget -y
wget https://archive.apache.org/dist/kafka/2.8.0/kafka_2.12-2.8.0.tgz
tar -xvzf kafka_2.12-2.8.0.tgz
cd kafka_2.12-2.8.0
Start Kafka and Zookeeper:

bash

# Start Zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties &

# Start Kafka server
bin/kafka-server-start.sh config/server.properties &
Create a Kafka topic for sensor data:

bash

bin/kafka-topics.sh --create --topic sensor_data --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
Step 6: Install Python and PySpark
Install Python 3 and pip if not already installed:

bash

sudo apt install python3-pip -y
pip3 install pyspark
Step 7: Set Up Your PySpark Streaming Code
Create a new directory for your project and add the PySpark Streaming code:

bash

mkdir ~/realtime_data_processing
cd ~/realtime_data_processing
Create the streaming_app.py file with the following content:

python

from pyspark.sql import SparkSession
from pyspark.streaming import StreamingContext

# Initialize SparkSession
spark = SparkSession.builder \
    .appName("RealTimeDataProcessing") \
    .getOrCreate()

# Set up StreamingContext, with a batch interval of 10 seconds
ssc = StreamingContext(spark.sparkContext, 10)

# Create a DStream for receiving data from a socket (localhost, port 9999)
lines = ssc.socketTextStream("localhost", 9999)

# Simple anomaly detection: If temperature > 100, it's an anomaly
def detect_anomalies(data):
    try:
        temp = float(data)
        if temp > 100:  # Anomaly detection threshold
            return "Anomaly Detected"
        else:
            return "Normal"
    except ValueError:
        return "Invalid Data"

# Process data and detect anomalies
processed_stream = lines.map(detect_anomalies)

# Print results to console
processed_stream.pprint()

# Start the streaming context
ssc.start()
ssc.awaitTermination()
Step 8: Start a Socket Server for Simulating Data (Optional)
To simulate real-time data, start a netcat server on port 9999:

bash

nc -lk 9999
Manually enter data like:

bash

30
45
110  # Anomaly will be detected
80
200  # Anomaly will be detected
Step 9: Run the PySpark Streaming Application
Run the PySpark Streaming application using the following command:

bash

spark-submit streaming_app.py
Step 10: Monitor the Output
The application will continuously process data. You can monitor the output in the terminal, where it will indicate whether the data is normal or contains an anomaly.

Step 11: Scaling and Monitoring
For production use, consider the following:

Use Kafka for high-throughput data ingestion.
Deploy Spark on a cluster to handle larger data volumes.
Set up monitoring tools like Grafana or Prometheus to track real-time processing metrics.
Optional Step 12: Use Jupyter for Visualization
To visualize the data being processed, you can set up a Jupyter notebook on the EC2 instance:

bash

pip3 install notebook
jupyter notebook --ip=0.0.0.0 --port=8888 --no-browser
Then, access Jupyter in your browser by navigating to http://<EC2_PUBLIC_IP>:8888.