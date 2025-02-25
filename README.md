# Web Server Log Analysis Using Apache Hive

## Project Overview

This project uses Apache Hive to handle and analyze web server log data. The goal is to extract information about traffic sources, suspicious activity, traffic patterns, most frequented pages, total requests, and status code distribution.

## Implementation Approach

1. **Setup Hive Environment:** Hive, HDFS, and Hue are installed in a Dockerized environment.
2. **Load Data:** To construct an external Hive table, upload the web server logs in CSV format to HDFS.
3. **Execute Queries:** Use HiveQL to carry out the necessary analysis.
4. **Partitioning:** Improve query efficiency by utilizing status code partitioning.
5. **Export and Report:** Get the results and produce the output in the format that is needed.

6. ## Execution Steps
### 1. Setup Hive and HDFS
```sh
# Start Docker containers
docker-compose up -d

# Access the running Hadoop container
docker exec -it namenode /bin/bash

# Create HDFS directories
hdfs dfs -mkdir -p /user/hive/warehouse/web_logs
```

### 2. Upload the CSV file to HDFS
```sh
# Copy file from local system to HDFS
docker cp /workspaces/webserver-log-analysis-hive-SriLaxmiPrasannaJoginipelli/web_server_logs.csv namenode:/web_server_logs.csv

docker exec -it namenode /bin/bash

hdfs dfs -put /web_server_logs.csv /user/hive/warehouse/web_logs/

```

### 3. Load data into hive

```sql
LOAD DATA INPATH '/user/hive/warehouse/web_logs/web_server_logs.csv' 
INTO TABLE web_server_logs;
```

### 3. Create Hive Table
```sql
CREATE EXTERNAL TABLE IF NOT EXISTS web_server_logs (
    ip STRING,
    `timestamp` STRING,
    url STRING,
    status INT,
    user_agent STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/hive/warehouse/web_logs';
```

### 4. Run Analysis Queries
#### Count Total Web Requests
```sql
SELECT COUNT(*) AS total_requests FROM web_server_logs;
```

#### Analyze Status Codes
```sql
SELECT status, COUNT(*) AS count 
FROM web_server_logs 
GROUP BY status 
ORDER BY count DESC;
```

#### Identify Most Visited Pages
```sql
SELECT url, COUNT(*) AS visit_count 
FROM web_server_logs 
GROUP BY url 
ORDER BY visit_count DESC;
```

#### Traffic Source Analysis
```sql
SELECT user_agent, COUNT(*) AS count 
FROM web_server_logs 
GROUP BY user_agent 
ORDER BY count DESC;
```

#### Detect Suspicious Activity
```sql
SELECT ip, COUNT(*) AS failed_requests 
FROM web_server_logs 
WHERE status IN ('404', '500') 
GROUP BY ip 
HAVING failed_requests > 3;
```

#### Analyze Traffic Trends
```sql
SELECT SUBSTR(`timestamp`, 1, 16) AS minute, COUNT(*) AS request_count 
FROM web_server_logs 
GROUP BY SUBSTR(`timestamp`, 1, 16) 
ORDER BY minute;
```

## Challenges Faced

- **HDFS Upload Issues:** Directory permissions were fixed, and the CSV path was accurate.
- **Hive Table Creation:** corrected naming conventions for columns (e.g., avoiding reserved terms like `timestamp`).

## Query and Output

![image](https://github.com/user-attachments/assets/fbccabff-ee19-422d-8952-9ef5a81200da)

![image](https://github.com/user-attachments/assets/377510d8-d42f-401e-b70e-0967a4751b57)

![image](https://github.com/user-attachments/assets/703dd4e1-2695-4984-9917-04e84750db08)







