# BigData-SentimentAnalysis-StreamSets

Below are the tools used for implementing the use case "Twitter Sentiment Analysis using StreamSets on Big Data Cluster":

1. Cloudera Cluster 6.0.
2. StreamSets data collector
2. Apache Kafka which shipped with CDH 6.0.0.
3. Apche Spark which shipped with CDH 6.0.0.
4. Natural Language toolkit(NLTK)
5. Stanford CoreNlp for sentiment analysis

here i will focus to describe StreamSets and the pipeline which is used to implement the use case.

**What is StreamSets Data Collector** 

StreamSets Data Collector is used to to efficiently build, test, run and maintain dataflow pipelines connecting a variety  
of batch and streaming data sources and compute platforms. you can say it is a ETL tool.

**What is a pipeline** 

A pipeline consists of stages that represent the origin and destination of the pipeline, and any additional processing  
that you want to perform.Data Collector pipelines require minimal schema specification and uniquely detect and handle data 
drift.You can view real-time statistics about your data, inspect data as it passes through the pipeline, or take a close       
look at a snapshot of data.

**Data collection challenges if we don't use SDC like tool**

1. Custom coding :- in traditinal manual pipelines, we need to write custom code which is very cumbersome and error prone.
2. Lengthy development :- in tradional manuual pipeline, building a pipeline, testing it and then depoying it takes huge time which can delay the relevent information to 
                          stakeholders
3. Brittle pipelines :- hand coding pipelines are not capable to handle frequest changes.

**Features of SDC(StreamSets Data Collector)**

1. Quickly building pipeline.
2. Drag and drop connectors for batch and streaming source/destinations.
3. Minimal schema specification needed which ultimately speed up the pipeline development.
4. it can detect data drift and propogates the changes into the target data automatically.
5. To solve your ingest needs, you can use a single Data Collector to run one or more pipelines. Or you might install a series of Data Collectors on multiple nodes.
6. In pipelines that write to Hive or parquet or to PostgreSQL, you can implement a data drift solution that detects drift in incoming data and updates tables in destination systems. 
7. While the pipeline runs, you can monitor the pipeline to verify that the pipeline performs as expected. You can also define metric and data rules and alerts to let you know when certain thresholds are reached.
8. Lightweight Transformation for Consumption-Ready Data
    *   Leverage dozens of built-in processors or design your own.
    *   Trigger custom code when needed.
9. Intelligent Monitoring and Error Detection
    *   Pinpoint problems using fine-grained metrics.
    *   Error detection using triggers and alerts.
    *   Inspect data at any point along a pipeline.
    
**Pipeline concepts and design**
 
Data passes through the pipeline in batches. This is how it works:

The origin creates a batch as it reads data from the origin system or as data arrives from the origin system, noting the offset.The offset   is the location where the origin stops reading.The origin sends the batch when the batch is full or when the batch wait time limit elapses. The batch moves through the pipeline from processor to processor until it reaches pipeline destinations.Destinations write the batch to destination systems, and Data Collector commits the offset internally. Based on the pipeline delivery guarantee,Data Collector either commits the offset as soon as it writes to any destination system or after receiving confirmation of the write from all destination systems.
After the offset commit, the origin stage creates a new batch.Note that this describes general pipeline behavior. Behavior can differ based on the specific pipeline configuration. For example, for the Kafka Consumer, the offset is stored in Kafka or ZooKeeper. And for origin systems that do not store data, such as Omniture and HTTP Client, offsets are not stored because they aren't relevant.

Pipeline runs either in standalone mode or cluster mode.
each pipeline supports Single threaded or multithreaded pipeline.

**Pipeline components(stages)**
   * Origins
   * Destination
   * Processors
   * Executors
   
   **Origin**
   
   An origin stage represents the source for the pipeline. You can use a single origin stage in a pipeline.You can use different origins  
   based on the execution mode of the pipeline: standalone, cluster, or edge. Basically it is used to ingest data into the pipeline from source system.Development origins are used to create test pipeline where you can specify the test data instead of taking data from source system.
   
   **Processors**
   
   A processor stage represents a type of data processing that you want to perform. You can use as many processors in a pipeline as you need.You can use different processors based on the execution mode of the pipeline: standalone, cluster, or edge.
Development Processors are used to create test pipeline.

  **Destination**
  
  A destination stage represents the target for a pipeline. You can use one or more destinations in a pipeline.
  You can use different destinations based on the execution mode of the pipeline: standalone, cluster, or edge. To help create or test pipelines, you can use a development destination.
  
  **Executors**
  
  An executor stage triggers a task when it receives an event. Executors do not write or store events.
  Use executors as part of a dataflow trigger in an event stream to perform event-driven, pipeline-related tasks, such as moving a fully- 
  written file when a destination closes it.
   
   

**Sentiment Analysis Pipeline**

A data pipeline is divided into the following parts:
1. Ingestion(Extract)
2. Transformation
3. Storage(Load)

The sentiment analysis use case has 2 pipelines. 

First pipeline takes stream data from twitter and after performing required transformaion store it into kafka topic.

The second pipeline takes the data from kafka topic and after performing required transformation , it store the data into RDBMS data table. Then using Tableau, we coonect to the RDBMS database and display the insights.

**First pipeline**

![pic1_pipeline1](https://user-images.githubusercontent.com/12975741/55283027-0b75e480-5377-11e9-800c-aab7e2306799.png)

