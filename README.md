# BigData-SentimentAnalysis-StreamSets

Below are the tools used for implementing the use case "Twitter Sentiment Analysis using StreamSets on Big Data Cluster":

1. Cloudera Cluster 6.0.
2. StreamSets data collector
2. Apache Kafka which shipped with CDH 6.0.
3. Apche Spark which shipped with CDH 6.0.
4. Natural Language toolkit(NLTK)
5. Stanford CoreNlp for sentiment analysis

here i will focus to describe StreamSets and the pipeline which is used to implement the sentiment analysis use case.

**What is StreamSets Data Collector** 

StreamSets Data Collector is used to to efficiently build, test, run and maintain dataflow pipelines connecting a variety  
of batch and streaming data sources and compute platforms. you can say it is a ETL tool.

**What is a pipeline** 

A pipeline consists of stages that represent the origin and destination of the pipeline, and any additional processing  
that you want to perform.Data Collector pipelines require minimal schema specification and uniquely detect and handle data 
drift.You can view real-time statistics about your data, inspect data as it passes through the pipeline, or take a close       
look at a snapshot of data.

**Data collection challenges if we don't use SDC like tool**

1. Custom coding :- In traditinal manual pipelines, we need to write custom code which is very cumbersome and error prone.
2. Lengthy development :- In tradional manuual pipeline, building a pipeline, testing it and then depoying it takes huge time which can delay the relevent information to 
                          stakeholders
3. Brittle pipelines :- Hand coding pipelines are not capable to handle frequest changes.

**Features of SDC(StreamSets Data Collector)**

1. Quickly building pipeline.
2. Drag and drop connectors for batch and streaming source/destinations.
3. Minimal schema specification needed which ultimately speed up the pipeline development.
4. It can detect data drift and propogates the changes into the target data automatically.
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

In the above pipeline image, you can see the set of stages with step mentioned to show the flow of data.

Pipeline starts and perform the following steps for each batch of messages:

Step 1 : "Ingest Tweets Stream" stage takes the stream of tweets from tweeter using tweeter's stream API.

Step 2 : If the tweet's language is englist then it is moved to step 3.1, otherwise it is moved to trash(step 3.2).

Step 3.1: Only required tweeter fields are selected from the tweet's json.

Step 4: Tweet's nested json is flattened.

Step 5: Tweet's field data type is converted on the basis of field's data type.

Step 6: Tweet's field data type is converte on the basis of field name.

Step 7: Tweet's text is preprocessed and cleaned using NLTK in "Jython Evaluator". Jython is java implementation of Python.here i have used a jython script in java language to invoke python script which actually uses the NLTK library. i have not found working NLTK library for jython therefore i am forecd to use python(code in python language) from jython(code in java language).

Step 8: The transformed tweet message object is saved in kafka topic using kafka producer in kafka cluster.


**Second pipeline**

![pic2_pipeline1](https://user-images.githubusercontent.com/12975741/55283143-eafb5980-5379-11e9-82f4-f431d21e2303.png)

In the above pipeline image, you can see the set of stages with step mentioned to show the flow of data.

Pipeline starts and perform the following steps for each batch of messages:

Step 1: tweets are taken from kafka topic by kafka consumer.

Step 2: Tweet's field data type is converted by field name as required in Apache spark java code.

Step 3: Sentiment analysis is done for the tweet to know whether it is positive,negative or neutral using stanford core nlp library.

Step 4: The processed data having tweet and its sentiment are saved into RDBMS database.

Step 5: The tweet's text is tokenized(splitted into words).

Step 6: The tweet is saved into RDBMS database.


**Pipeline monitoring and data preview(in preview mode, not in running mode)**

**Note: The pipeline shown below are different as i captured them before i made the final changes to the pipelines.i have added them here just to give an idea of monitoring and error of any pipeline**

You can monitor running pipeline(comple pipeline or each pipeline stage).

![p6](https://user-images.githubusercontent.com/12975741/55286620-98429180-53bb-11e9-9247-0d93b4cc5a9a.png)

Above pipeline shows that 346 tweets have been injested into the pipeline and 346 tweets are loaded into the destination system.

![p2](https://user-images.githubusercontent.com/12975741/55286782-eb1d4880-53bd-11e9-81f7-e59c8f5b7215.png)

Above pipeline shows that what are the good records ,error records ,input records and output records for stage "fields selection".

![p3](https://user-images.githubusercontent.com/12975741/55286786-ee183900-53bd-11e9-9f7b-17cd93f9198c.png)

Above pipeline shows that for the selected stage("stream selector") , input records sent to the stage is 462, output records sent from output 1(Trash) is 106 and output reords sent from output2(field type converter) is 356.

![p4](https://user-images.githubusercontent.com/12975741/55286791-f2445680-53bd-11e9-8bb8-a984f9d6ca18.png)

Above pipeline shows the error records.

![p5](https://user-images.githubusercontent.com/12975741/55286793-f6707400-53bd-11e9-97de-8d2d6e738221.png)

Above pipeline is running in cluster execution mode and it has created muliple worker SDC instances to run the pipeline in cluster.

**First pipeline stage configuration**

Below are the configuration i have used in first pipeline's stages:

![p11](https://user-images.githubusercontent.com/12975741/55287075-2752a800-53c2-11e9-8b5a-11021d87b64b.png)

![p22](https://user-images.githubusercontent.com/12975741/55287079-30437980-53c2-11e9-9733-255fedef7d2f.png)

![p33](https://user-images.githubusercontent.com/12975741/55287085-376a8780-53c2-11e9-9b78-0be9dfc9cd11.png)

![p44](https://user-images.githubusercontent.com/12975741/55287087-3b96a500-53c2-11e9-9a27-f68498efc949.png)

![p55](https://user-images.githubusercontent.com/12975741/55287089-3f2a2c00-53c2-11e9-8fab-62533f3cf9a8.png)

![p66](https://user-images.githubusercontent.com/12975741/55287093-42bdb300-53c2-11e9-92cd-42d03621116f.png)

![p77](https://user-images.githubusercontent.com/12975741/55287094-45200d00-53c2-11e9-91b9-f205fc76bb04.png)

![p88](https://user-images.githubusercontent.com/12975741/55287096-47826700-53c2-11e9-93a2-0f333c788211.png)

![p99](https://user-images.githubusercontent.com/12975741/55287107-5406bf80-53c2-11e9-8fc7-e72453186236.png)

![p111](https://user-images.githubusercontent.com/12975741/55287109-5c5efa80-53c2-11e9-99e5-f2f1e1e52bf5.png)

![p222](https://user-images.githubusercontent.com/12975741/55287110-5ec15480-53c2-11e9-8e78-4c93b29b7099.png)

![p333](https://user-images.githubusercontent.com/12975741/55287112-6254db80-53c2-11e9-8c41-2291b06fb126.png)


**Second pipeline stage configuration**

Below are the configuration i have used in second pipeline's stages:

![p444](https://user-images.githubusercontent.com/12975741/55287113-67198f80-53c2-11e9-89d5-3589fca52e5e.png)

![p555](https://user-images.githubusercontent.com/12975741/55287115-6a148000-53c2-11e9-97c7-279f47f66ed0.png)

![p666](https://user-images.githubusercontent.com/12975741/55287118-74367e80-53c2-11e9-8751-a742f7819820.png)

![p777](https://user-images.githubusercontent.com/12975741/55287120-77316f00-53c2-11e9-968a-5030a94d0ee1.png)

![p888](https://user-images.githubusercontent.com/12975741/55287122-78fb3280-53c2-11e9-8b6a-6866e3fb35df.png)

![p999](https://user-images.githubusercontent.com/12975741/55287126-7e587d00-53c2-11e9-8ca9-ccc5f937ff57.png)

![p1111](https://user-images.githubusercontent.com/12975741/55287129-80bad700-53c2-11e9-93f6-66c4b5116637.png)

WIP: i will update the remaining part later.
