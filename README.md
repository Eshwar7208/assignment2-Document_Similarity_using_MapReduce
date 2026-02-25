# Assignment 2: Document Similarity using MapReduce

**Name:** Eshwar Dudala\
**Student ID:** 801454682

------------------------------------------------------------------------

## Overview

This project implements a Document Similarity computation system using
the Hadoop MapReduce framework. The objective is to calculate the
Jaccard Similarity between every pair of documents in a dataset.

Each input record represents a document in the following format:

    DocID word1 word2 word3 ...

The program processes the dataset and produces similarity scores for all
unique document pairs.

------------------------------------------------------------------------

## Methodology and Design

### 1. Mapper Implementation

The `DocumentSimilarityMapper` class extends:

    Mapper<LongWritable, Text, Text, Text>

### Input

-   **Key:** Byte offset of the line\
-   **Value:** Entire line of text (one document)

### Processing Logic

-   Each line is tokenized using whitespace.
-   The first token is extracted as the Document ID.
-   Remaining tokens are treated as document terms.
-   All terms are converted to lowercase and stored in a
    `HashSet<String>` to remove duplicates.

During the `map()` phase, documents are stored in a
`LinkedHashMap<String, Set<String>>`.

After processing all input records, the `cleanup()` method:

-   Generates all unique document pairs.
-   Computes Jaccard Similarity using:

J(A,B) = \|A ∩ B\| / \|A ∪ B\|

The mapper emits:

    Key   → "DocA, DocB"
    Value → "Similarity: X.XX"

The similarity value is formatted to two decimal places.

------------------------------------------------------------------------

### 2. Reducer Implementation

The `DocumentSimilarityReducer` extends:

    Reducer<Text, Text, Text, Text>

Since similarity computation is completed in the mapper, the reducer
acts as a pass-through component.

------------------------------------------------------------------------

## Data Flow

1.  Input file → Each line represents one document\
2.  Mapper (map phase) → Parses document and stores word sets\
3.  Mapper (cleanup phase) → Generates document pairs and computes
    similarity\
4.  Shuffle & Sort → Hadoop groups results by document pair\
5.  Reducer → Writes final similarity scores\
6.  Output → Tab-separated similarity results in HDFS

Final output format:

    DocA, DocB    Similarity: X.XX

------------------------------------------------------------------------

## Environment Setup and Execution

### Step 1: Start Hadoop Cluster

``` bash
docker compose up -d
```

### Step 2: Build the Project

``` bash
mvn clean package
```

### Step 3: Copy JAR File to Hadoop Container

``` bash
docker cp target/DocumentSimilarity-0.0.1-SNAPSHOT.jar resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### Step 4: Transfer Dataset to Container

``` bash
docker cp shared-folder/input/data/input.txt resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### Step 5: Access the Hadoop Container

``` bash
docker exec -it resourcemanager /bin/bash
cd /opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### Step 6: Prepare HDFS

``` bash
hadoop fs -mkdir -p /input/data
hadoop fs -put ./input.txt /input/data
```

### Step 7: Execute the MapReduce Job

``` bash
hadoop jar /opt/hadoop-3.2.1/share/hadoop/mapreduce/DocumentSimilarity-0.0.1-SNAPSHOT.jar com.example.controller.DocumentSimilarityDriver /input/data/input.txt /output1
```

### Step 8: View Output

``` bash
hadoop fs -cat /output1/*
```

------------------------------------------------------------------------

## Sample Input

    Document1 This is a first document. Hey I am the first document
    Document2 This is the second document
    Document3 Hey I am the third document. How are you

## Sample Output

    Document1, Document2    Similarity: 0.36
    Document1, Document3    Similarity: 0.36
    Document2, Document3    Similarity: 0.08

------------------------------------------------------------------------

## Conclusion

This project demonstrates how Hadoop MapReduce can be used to compute
document similarity using the Jaccard coefficient. The implementation
leverages efficient set operations, structured MapReduce design, and
proper output formatting to ensure accurate results.
