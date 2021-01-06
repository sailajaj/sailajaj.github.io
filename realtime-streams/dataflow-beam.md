Stream processing with Dataflow Model and Apache Beam
When dealing with big data and challenges around data aggregations ,with the constraints of data correctness, we found a compelling answer in Apache Beam. The reality of most data pipelines today are :
1. Infinite streams of data and the unpredictability of receiving these data points in sequence.
2. Large amounts of data that can only be processed in chunks (or windows)
3. Data correctness and accuracy is important; self correcting and sensors to reprocess data needs to be intrinsic to the system; minimize manual intervention
Traditional Batch Processing
Data aggregation of large volumes has typically been done using batch processing. As shown in the diagram below, batch jobs typically work with fixed intervals for window sizes, aggregating out-of-order data by its processing time. With distributed data sources feeding the pipeline, there is no guarantee the data will come in sequence. With that, the aggregations in batch jobs are more sensitive to the processing time rather than event time leading to difficulty in controlling data correctness with time. 
Batch jobs typically follow the Map->Reduce pattern where the reductions start with small intervals and further reductions(or aggregations) of these smaller chunks happen with time. In this model, small drops in data are accepted within the selected threshold. Large delays in data have to be reprocessed with the entire dependency chain being rerun for all the past and current intervals. This requires manual up-keep; with human and system resources checking and redoing the data aggregation and rebuilding our composite data.
Understanding what we need from data
 Understanding what we need from our data aggregations is an important first step as there are many variants based on the type of data one is working with. A few are listed below:
Batch with Fixed Windows
Streaming analysis with allowed lateness
Session analysis
Corrections on very late data
Typically Alerts/Trend analysis processes are less sensitive to delays unless there is a significant delay in data.
Whereas, aggregations/unique counts analysis need to be accurate with time. These processes need to sense delays, correct past data and follow through to re-report or re-run dependent scripts.
Why Apache Beam
There are many articles that talk about Beam and its usage. Here is one to get started LINK. What Beam gave us:
Flexibility: Beam as a runner opens us to many streaming analysis platforms independent of cloud. With many cloud options out there, a solution that is cloud agnostic is safer.
Correctness: Using Event-time and the windowing options (Fixed/Sliding/Session) we can self-correct our aggregations/counts with time. 
Modular : Streaming aggregations pipeline can be used for batch processing.  To correct past data we use the same streaming pipeline, removing the possibility of incorrect data being inserted based on the run.
Accountability : Moving away from SQL centric solutions allows us to bring a test environment to include a controlled and audit-able release process.

The best of Beam
PCollection - immutable container for elements;
PTransform - operation that consumes one PCollection and produces another PCollection;
No batch or streaming pipelines, only bounded and unbounded PCollections;
Pipeline and PTransforms are serialized and run on a variety of runners, including Cloud Dataflow.
Using Event time (instead of processing time) brings alignment to data and eventual correctness.

Using event time instead of processing timeUsing Windowing to control late data and build sensors

Sliding Windows with allowed latenessControlling the environment
There are many factors that one has to think about when building this system. Highlighting some of the constraints we ran into with continued adaptation of the framework.
1. Footprint: How much data can we afford to keep in memory. If the data footprint is small, we can keep more data in memory and have a larger allowed lateness window and consume longer data delays. However, this makes managing downstream dependent chain more complicated.
2. Heuristics: What is my optimal window size? Optimal window size determines the chunks in which data is present in its aggregated form in the DB and also determines the number of chunks that have to be  re-processed on delays. These window sizes have to remain consistent for downstream processes. If 95% of data, on average, is coming within a window, is that an optimal/accepted size?  
3. Modular: Separating alerts from data aggregation allows the flexibility for alerts to operate independent of data in DB. 
4. Cost effectiveness: If the window size is large then perhaps adding data completeness sensors from streams and running batch aggregations from DB raw data may be faster and cheaper.
5. Sensors: Sense long delays in data using sensors and rerun dependent jobs with DAG dependency checks.
6. Parallel processing: 
Release code without disturbing the data flow
We cannot update the process unless we have a proper way to 
1. Drain the existing in memory for the current window and push the data to storage.
2. Adjust the watermark to reprocess the data post restart of the process.
3. Pause Alert triggers during release. 
Ensure corrected data is the one in use
Streamed content in DB should update with the latest update time. Downstream processes should retrieve data for the latest timestamp for a given window. 
Conclusions
With the unpredictable nature of when the data arrival, the growing acceptance of big data and with the call for data correctness, Apache Beam leads us to a decently viable solution. Even with Beam doing the heavy lifting, there is much to be considered and evaluated. This article is highlighting some of our concerns and struggles.
