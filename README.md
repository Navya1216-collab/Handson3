
# WordCount-Using-MapReduce-Hadoop

This repository is designed to test MapReduce jobs using a simple word count dataset. In this project we provide a input file and then we create a maaper and reducer logic to count the occurence of each word in the given input. There are sample input and Expected output for the sample input.

## Approach and implementation
1. Mapper Logic: We use StringTokenizer to create tokens from the input file and loop it using while loop to map all the words in the input file with key value pairs. In this mapper, it will not count characters that are smaller than 3.

2. Reducer Logic: Using the output of Mapper logic we increase a variable sum value as we encounter same words and retun them. this way we will get a list of words and the number of times it occured in the input file as output.

## Setup and Execution

### 1. **Start the Hadoop Cluster**

Run the following command to start the Hadoop cluster:

```bash
docker compose up -d
```

### 2. **Build the Code**

Build the code using Maven:

```bash
mvn clean package
```

### 4. **Copy JAR to Docker Container**

Copy the JAR file to the Hadoop ResourceManager container:

```bash
docker cp target/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### 5. **Move Dataset to Docker Container**

Copy the dataset to the Hadoop ResourceManager container:

```bash
docker cp shared-folder/input/data/input.txt resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### 6. **Connect to Docker Container**

Access the Hadoop ResourceManager container:

```bash
docker exec -it resourcemanager /bin/bash
```

Navigate to the Hadoop directory:

```bash
cd /opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### 7. **Set Up HDFS**

Create a folder in HDFS for the input dataset:

```bash
hadoop fs -mkdir -p /input/data
```

Copy the input dataset to the HDFS folder:

```bash
hadoop fs -put ./input.txt /input/data
```

### 8. **Execute the MapReduce Job**

Run your MapReduce job using the following command: Here I got an error saying output already exists so I changed it to output1 instead as destination folder

```bash
hadoop jar /opt/hadoop-3.2.1/share/hadoop/mapreduce/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar com.example.controller.Controller /input/data/input.txt /output1
```

### 9. **View the Output**

To view the output of your MapReduce job, use:

```bash
hadoop fs -cat /output1/*
```

### 10. **Copy Output from HDFS to Local OS**

To copy the output from HDFS to your local machine:

1. Use the following command to copy from HDFS:
    ```bash
    hdfs dfs -get /output1 /opt/hadoop-3.2.1/share/hadoop/mapreduce/
    ```

2. use Docker to copy from the container to your local machine:
   ```bash
   exit 
   ```
    ```bash
    docker cp resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/output1/ shared-folder/output/
    ```
3. Commit and push to your repo so that we can able to see your output


## Sample Input: 
 ```bash
   Hello world
   Hello Hadoop
   Hadoop is powerful
   Hadoop is used for big data
   ```

## Expected output: 
 ```bash
Hadoop 3
Hello 2
used 1
for 1
big 1
data 1
powerful 1
world 1
   ```
