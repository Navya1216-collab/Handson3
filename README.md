
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
Bali is predominantly a Hindu country. Bali is known for its elaborate, traditional dancing. The dancing is inspired by its Hindi beliefs. Most of the dancing portrays tales of good versus evil.mvn clean package

   ```

## Expected output: 
 ```bash
its	2
Bali	2
dancing	2
good	1
traditional	1
Hindu	1
country.	1
evil.mvn	1
package	1
tales	1
versus	1
for	1
Hindi	1
inspired	1
known	1
dancing.	1
The	1
beliefs.	1
clean	1
the	1
Most	1
elaborate,	1
portrays	1
predominantly	1

   ```
## Challenges Faced & Solutions

### 1) `mvn` or Java not found on host
**Symptoms:** `mvn : The term 'mvn' is not recognized…` or `JAVA_HOME` not set.  
**Cause:** Maven/JDK not installed or PATH/JAVA_HOME not configured.  
**Fix (Windows PowerShell):**
```powershell
# Verify Java
java -version
# Set JAVA_HOME temporarily (example path)
$env:JAVA_HOME="C:\Program Files\Java\jdk-17"
$env:Path="$env:JAVA_HOMEin;$env:Path"

# Verify Maven
mvn -v
```
Install JDK + Maven if missing; then re-run `mvn clean package`.

---

### 2) Wrong container name in `docker cp` / `docker exec`
**Symptoms:** `No such container: resourcemanager`  
**Cause:** Service name differs from README or compose file.  
**Fix:** Run `docker ps --format "{{.Names}}"` and use the exact name (e.g., `hadoop-resourcemanager-1`) in:
```bash
docker cp target/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar <container>:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
docker exec -it <container> /bin/bash
```

---

### 3) JAR not found inside container
**Symptoms:** `hadoop jar ... file not found`  
**Cause:** Copied to the wrong path or build didn’t create the JAR.  
**Fix:** After `mvn clean package`, confirm locally:
```bash
ls target/*.jar
```
Then copy with the full, correct path and re-check inside the container:
```bash
docker cp target/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar <container>:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
docker exec -it <container> bash -lc 'ls -l /opt/hadoop-3.2.1/share/hadoop/mapreduce/*.jar'
```

---

### 4) `ClassNotFoundException` for main class
**Symptoms:** `ClassNotFoundException: com.example.controller.Controller`  
**Cause:** Wrong main-class path or the JAR isn’t “fat/uber” (missing dependencies).  
**Fix:** Ensure the package/class name matches your source. Consider building a shaded JAR (Maven Shade Plugin) with the correct `Main-Class` in the manifest, or provide the full class path when running:  
```bash
hadoop jar WordCountUsingHadoop-0.0.1-SNAPSHOT.jar com.example.controller.Controller /input/data/input.txt /output1
```

---

### 5) HDFS path vs local path confusion
**Symptoms:** `No such file or directory` when running `hdfs dfs -put` or reading input.  
**Cause:** Using local filesystem paths where HDFS paths are expected (or vice versa).  
**Fix:**
```bash
# Inside the container
hdfs dfs -mkdir -p /input/data
hdfs dfs -put ./input.txt /input/data
# Run with HDFS paths only:
hadoop jar ... /input/data/input.txt /output1
```

---

### 6) “Output directory already exists”
**Symptoms:** `FileAlreadyExistsException: /output1`  
**Cause:** HDFS output folder from a previous run still exists.  
**Fix:** Remove or choose a fresh output path:
```bash
hdfs dfs -rm -r -f /output1
# or pick a new one like /output2
```

---

### 7) Tokenization & punctuation inflating “weird” words
**Symptoms:** Output shows tokens like `country.`, `dancing.`, `elaborate,`.  
**Cause:** `StringTokenizer` (or simple split) leaves punctuation attached.  
**Fix:** Normalize input in mapper:
- Lowercase words.
- Strip punctuation via regex before emitting:
```java
String cleaned = token.toLowerCase().replaceAll("[^a-z0-9]+", "");
if (cleaned.length() >= 3) { /* emit */ }
```

---

### 8) Accidental build text in dataset (e.g., “mvn clean package”)
**Symptoms:** Output contains words like `evil.mvn`, `clean`, `package`.  
**Cause:** The sample input includes a CLI line; mapper counts it.  
**Fix:** Remove build logs from the input file or add a filter in mapper to skip lines starting with common shell prompts/commands.

---

### 9) Word-length filter mismatch with expectations
**Symptoms:** 2-letter words missing (by design), or graders expect different counts.  
**Cause:** Mapper intentionally ignores words with length < 3.  
**Fix:** Make the rule explicit in README and code, or adjust the threshold to match the expected output.

---

### 10) Case sensitivity causing duplicates (`The` vs `the`)
**Symptoms:** Separate counts for the same word in different cases.  
**Cause:** No normalization to lowercase.  
**Fix:** Convert tokens to lowercase before emitting.

---

### 11) HDFS permission or user issues
**Symptoms:** `Permission denied` for `hdfs dfs` commands.  
**Cause:** Running as a user without HDFS permissions.  
**Fix:** Use the Hadoop user configured in your image (often `hadoop`/`root` inside the container), or `hdfs dfs -chmod -R 777 /` for classroom demos (not for production).

---

### 12) YARN / memory errors
**Symptoms:** Container killed by YARN due to memory limits.  
**Cause:** Defaults too low for your job.  
**Fix:** Increase memory in `mapred-site.xml` / `yarn-site.xml` (in the image or mounted configs), e.g.:
```xml
<property><name>mapreduce.map.memory.mb</name><value>1024</value></property>
<property><name>mapreduce.reduce.memory.mb</name><value>1024</value></property>
<property><name>yarn.scheduler.maximum-allocation-mb</name><value>2048</value></property>
```
Recreate containers after changes.

---

### 13) Wrong working directory inside the container
**Symptoms:** `ls` shows no files you just copied; `hdfs dfs -put ./input.txt` fails.  
**Cause:** Not in the directory where you copied files.  
**Fix:**
```bash
cd /opt/hadoop-3.2.1/share/hadoop/mapreduce/
ls -l
```
Then run the `hdfs dfs` commands from there (or give absolute paths).

---

### 14) Slow or failed image pulls behind a proxy
**Symptoms:** Timeouts pulling `hadoop`, `java`, or base images.  
**Cause:** Corporate/ISP proxy.  
**Fix:** Configure Docker Desktop proxy settings (Settings → Resources → Proxies) and retry `docker compose up -d`.

---

### 15) Getting results out of the container cleanly
**Symptoms:** Confusion copying HDFS output to host.  
**Fix:** Three steps:
```bash
# 1) Inside container: copy from HDFS to container FS
hdfs dfs -get /output1 /opt/hadoop-3.2.1/share/hadoop/mapreduce/

# 2) Exit container
exit

# 3) From host: copy from container to host
docker cp <container>:/opt/hadoop-3.2.1/share/hadoop/mapreduce/output1/ ./out/
```

---

## Quick Start (TL;DR)
```bash
docker compose up -d
mvn clean package
docker cp target/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar hadoop-resourcemanager-1:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
docker cp input.txt hadoop-resourcemanager-1:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
docker exec -it hadoop-resourcemanager-1 /bin/bash -lc 'cd /opt/hadoop-3.2.1/share/hadoop/mapreduce/ && hdfs dfs -mkdir -p /input/data && hdfs dfs -put -f ./input.txt /input/data/ && hdfs dfs -rm -r -f /output1 && hadoop jar WordCountUsingHadoop-0.0.1-SNAPSHOT.jar com.example.controller.Controller /input/data/input.txt /output1 && hdfs dfs -get /output1 ./output1'
docker cp hadoop-resourcemanager-1:/opt/hadoop-3.2.1/share/hadoop/mapreduce/output1 ./out
docker compose down -v
```
