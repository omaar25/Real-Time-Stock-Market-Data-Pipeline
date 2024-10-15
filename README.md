# Real-Time Stock Market Data Pipeline

## Project Overview
The goal of this project is to build a **real-time data pipeline** that streams stock market data from a CSV file, sends it through **Kafka** running on an **EC2 instance**, and stores it in an **AWS S3 bucket** for analysis using **AWS Glue** and **Athena**. The pipeline involves:
- **Producer**: Generating real-time stock data.
- **Consumer**: Writing the data to S3.

### Pipeline Stages:
1. **Generate Real-Time Data** from CSV using Python
2. **Kafka Producer**  sends data to Kafka on EC2
3. **Kafka Consumer** sends data to S3
4. **AWS Glue** crawler to catalog data
5. **Query data** using AWS Athena

## Tools & Technologies Used
- **Python 3.13.0**: For data generation and streaming via Kafka.
- **Apache Kafka**: For real-time data streaming.
- **AWS EC2**: To host Kafka brokers.
- **AWS S3**: For storing the consumer output.
- **AWS Glue**: To create a data catalog from S3.
- **AWS Athena**: To query the cataloged data.

## Step-by-Step Implementation

### 1. Set Up Kafka on AWS EC2
**Prerequisites**: An EC2 instance running Amazon Linux 2 and Java installed.

**SSH into EC2:**
```
ssh -i "your-ec2-key.pem" ec2-user@<EC2-Public-IP>
```
**Install Java:**
```
sudo yum install java-1.8.0-openjdk
java -version
```
**Download and Extract Kafka:**
```wget https://downloads.apache.org/kafka/3.3.1/kafka_2.12-3.3.1.tgz
tar -xvf kafka_2.12-3.3.1.tgz
```

**Start ZooKeeper (Kafka's coordination service):**

```
cd kafka_2.12-3.3.1
```
```
bin/zookeeper-server-start.sh config/zookeeper.properties
```

**Start Kafka Broker: In a new terminal, SSH into the EC2 instance again and run:**

```
export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M"
```
```
cd kafka_2.12-3.3.1
```
```
bin/kafka-server-start.sh config/server.properties
```

**Configure Kafka to use Public IP: Edit server.properties to make Kafka accessible from your local machine:**
```
sudo nano config/server.properties
```

**Change advertised.listeners to use the public IP of the EC2 instance:**
```
advertised.listeners=PLAINTEXT://<EC2-Public-IP>:9092
```
**Create Kafka Topic:**

```
bin/kafka-topics.sh --create --topic stock_market --bootstrap-server <EC2-Public-IP>:9092 --replication-factor 1 --p
```

### 2. Producer: Send Stock Market Data to Kafka

**Install Kafka Python Library:**

```
pip install kafka-python
```

**Python Script to Stream Data:**

go to ```Producer Kafka.ipynb```
This script reads data from the CSV, picks a random row, and sends it to the Kafka topic stock_market on the EC2 instance.

### 3. Consumer: Read Data from Kafka and Store in AWS S3

**Install Kafka and S3FileSystem:**

```
pip install kafka-python s3fs
```
To Start the Kafka Producer:
```
bin/kafka-console-producer.sh --topic stock_market --bootstrap-server <EC2-Public-IP>:9092
```

**Configure AWS CLI with IAM User Credentials**
```
aws configure
```
You'll be prompted to input your IAM user credentials:

1. **AWS Access Key ID:** Your IAM access key
2. **AWS Secret Access Key:** Your IAM secret key
3. **Default region:** The region where your S3 bucket is located (e.g., us-east-1)
4. **Default output format:** Set this to json
   
**Python Script to Consume Data and Save to S3:**

go to ```Consumer Kafka.ipynb```
This script consumes messages from the Kafka topic demo_test and writes them as JSON files to an S3 bucket. Make sure the EC2 instance has the necessary IAM role to write to S3.

To Start the Kafka Consumer:
```
bin/kafka-console-consumer.sh --topic stock_market --bootstrap-server <EC2-Public-IP>:9092
```

### 4. AWS Glue Crawler

**Set Up Glue Crawler:**

* Create a crawler in AWS Glue that scans the S3 bucket where Kafka consumer is writing the data.
* The crawler will create a schema and store it in a data catalog.

## 5. Query Data with Athena:
Once the data is cataloged by Glue, use AWS Athena to query the data.
You can run SQL queries on the data stored in S3 using Athena's query editor.


### Acknowledgments

I would like to express my gratitude to [Darshil Parmar] for their guidance throughout the development of this project. 

You can find more of his work on:
- [YouTube Channel](https://www.youtube.com/@DarshilParmar)
- [LinkedIn Profile](https://www.linkedin.com/in/darshil-parmar/)


