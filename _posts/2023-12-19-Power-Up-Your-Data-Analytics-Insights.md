---
layout: post
title: "Power-up your data analytics: KX Insights"
author: Cristian Pérez Corral, Christian Aberturas López
image: /img/butterfly-ship.jpg
toc: true
---


# KX Insights: data analytics made simple

KX Insights Enterprise is the fastest cloud-based platform for time series analysis, leveraging the power of kdb+ vector databases. But it's not just that; throughout this post, we will demonstrate how, without requiring advanced technical knowledge, you can develop data ingestion, query engine, and visualization—all thanks to this tool. Will you join us?

## Use Case
As a telecommunications (telco) company immersed in the dynamic world of high-density real-time telco data, we are on a mission to empower our Data Team with a robust system for in-depth data analysis and insightful conclusions. Our vision entails the creation of comprehensive dashboards that provide a visual representation of key metrics. To bring this vision to life, we have identified specific elements that will be pivotal in enhancing our operational understanding.

### User Distribution Heatmap:
Our objective is to gain insights into the geographical distribution of our users in real-time. The User Distribution Heatmap within KX Insights will vividly illustrate the concentration of our current users, allowing us to identify high-demand areas promptly.

### Network Performance Analysis (Line Graph):
Understanding the impact of network volume on speed is critical for ensuring a seamless user experience. Through KX Insights, we plan to deploy a Line Graph that dynamically illustrates how network volume influences speed, enabling us to optimize our network performance proactively.

### User Location Tracker:
The ability to locate users within a specific geography is essential for targeted interventions and optimizations. KX Insights will provide real-time information on the geographical whereabouts of our users, facilitating strategic decision-making.

### Failure Analysis TreeMap:
Identifying and resolving issues promptly is key to maintaining a high-quality service. A TreeMap, powered by KX Insights, will pinpoint users experiencing difficulties with their connection. This enables us to swiftly address issues and ensure a seamless communication experience for our users.

Our choice of KX Insights is driven not only by the nature of our data, characterized by a vast volume of real-time information, but also by the platform's exceptional capabilities in constructing functional pipelines, executing complex queries, and delivering compelling visualizations. With KX Insights, we are equipped to harness the full potential of our telco data, providing our Data Team with the tools they need to navigate and derive actionable insights from this dynamic landscape.
## Overview

Before delving into the analysis, it's essential to navigate through some preliminary steps, including the creation of the database and data ingestion processes.

For our development journey, we've opted to leverage the use case outlined earlier for two compelling reasons: the data's temporal nature, allowing for the utilization of time series analysis tools, and its high density. By doing so, we aim not only to demonstrate the prowess of KX Insights but also to shed light on the underlying machinery powering it—the kdb+ database.

Assuming a Kafka server has already been deployed to store and manage the incoming data from the telco network, we'll subscribe to the corresponding topic to seamlessly capture the data.

This post will guide you through the intricacies of constructing a data pipeline in KX Insights, encompassing key stages such as defining the database schema, streaming data ingestion, and visualizing and querying data. Organizing our discussion around the aforementioned use case, we'll first develop a simplified pipeline to capture the foundational aspects of the data later visualized. Subsequently, we'll delve into refining our initial pipeline, exploring the extensive possibilities offered by this versatile tool. Let the journey begin!

## Technical skillset
As we will show through this post, you don't need much technical knowledge to implement a functional pipeline, database and visualization system with KX Insights, more than just a little kwowledge on q programming language. What you will be grateful to know is how is the data stored, in order to query it efectively. KX Insights supports three types of databases: **RDB, IDB and HDB**. Each one of then store a type of data:
- RDB stores only real time data, and just a little amount of timelapse.
- IDB stores data as intervals from the real time, to time to time whe pipeline will send a signal indicating either the start or the end of a new interval and then all the data will be stored in IDB as batches.
- HDB stores the historic data. Once enough time has passed since the data was stored in IDB, it moves directly to be stored in HDB.

Then, the streaming data will go through  **RDB → IDB → HDB**.

If you want to know more about this, please check the [docs](https://code.kx.com/insights/1.8/microservices/database/storage/index.html#stream-partitioning-idb-eoi).
##  Was it always this easy?

Now, let's consider the scenario where we need to construct a similar architecture using only kdb+/q. While we have a plethora of tools at our disposal, the question arises: How challenging would this endeavor be? Here, we succinctly outline some of the crucial steps that must be undertaken:

1. **Build a Kafka Consumer:**
   Constructing a Kafka consumer entails delving into the [API](https://github.com/KxSystems/kafka), understanding its intricacies, and developing the necessary infrastructure to facilitate seamless data consumption.

2. **Orchestrate the Pipeline:**
   Docking the pipeline involves implementing all the transformations essential for processing real-time data. This step requires careful consideration and meticulous execution.

3. **Develop Graphic Tools:**
   While the [_Grammar of Graphics_](https://code.kx.com/analyst/libraries/grammar-of-graphics/) package is a powerful tool, we would still need to create a system for visualizing the aggregated data. This involves developing graphic tools tailored to our specific requirements.

Beyond these tasks, there's the challenge of working with an event handler for streaming data, which is not always straightforward. Additionally, packaging all these components into a black-box-like application is necessary to empower the entire data team to work with it, regardless of their technical expertise. Moreover, generalizing this solution for any conceivable use case poses an even more significant undertaking.

Enter [KX Insights Enterprise](https://kx.com/products/kdb-insights-enterprise/). This solution wraps the complexity in a user-friendly web interface, eliminating the need for the data team to possess intricate technical knowledge. Now, anyone on the team can seamlessly work with the data, thanks to the intuitive interface provided by KX Insights Enterprise.

To walk all the reader through this tool as easy as possible, we will follow the recommended steps of KX Insights, that can be seen in the following image:

![]({{ site.baseurl }}/assets/2023/12/19/overview.png)
 ### Deploying the databases
Once we have accessed the databases tab, we can configure it to suit our specific needs. We will leave the technical specifications (memory, storage, etc.) at their default settings, thus avoiding the technical details.

Our database (which we will name db-telco) will have two tables: `main` and `errs`. The first one is where real-time telephony data will be stored (and from where the heatmap, tracker and volume views will capture the data), and the second one will be used for a subsequent grouping to study errors in the network (and assist the most errored users). The `main` table will have 10 columns, which are:

-   `ts`: The timestamp of the network trace for the record (timestamp type).
-   `cellID`: The ID of the cell on which the trace is being transmitted (symbol type).
-   `phase`: This field could be avoided.
-   `imsi`: The ID of the SIM card participating in the trace (string type).
-   `imei`: The ID of the device participating in the trace (string type).
-   `dspeed`: The download speed of the network at this moment, cell, and event (float type).
-   `uspeed`: The upload speed of the network at this moment, cell, and event (float type).
-   `lat`: The latitude of the device from which the event is taking place (float type).
-   `lng`: The longitude of the device from which the event is taking place (float type).
-   `cellDistance`: The distance to the cell of the device (float type).

![Schema Main](assets/2023/12/19/schema_main.png)

And the `errs` table has 3 (we have already seen `ts` and `imsi`):

-   `count0`: This counts the errors that have occurred in the network in the grouping (int type).

![Schema Errs](assets/2023/12/19/schema_errs.png)

After creating the schemas for the tables, we save and click on deploy, and with that, we have the database up and running!


### Data slide: the pipeline
When it comes to data ingestion, pipelines are employed. The tools provided by KX Insights Enterprise for the creation, analysis, and testing of pipelines are extensive and straightforward to use, as we will see below. The more basic pipeline have four fundamental parts, which can then be further complicated or supplemented with others, namely:

- **Reader:** The reader for Insights pipelines is highly versatile, allowing data ingestion from multiple sources, as depicted in below image.
- **Decoder:** The decoder, responsible for transforming data received through the reader into different formats (JSON, CSV, etc.) into a q structure.
- **Apply Schema:** Shapes the incoming data through ingestion. It can be any type of data, tabular, or an array.
- **Writer:** This last component, as its name suggests, is responsible for writing the received data into a table in a previously created database.

Let's create a pipeline for the ingestion of telephone data, provided by a Kafka service. To achieve this, having chosen Kafka as the data source (selecting the corresponding broker and topic), JSON as the decoder, the schema of the `main` table seen earlier, and writing to that table, we end up with the following:

![Basic Pipeline](assets/2023/12/19/basic_ppline.png)

Before having a functional pipeline, a couple of things need to be done.

As we are getting JSON-format data, we will have to decode it. The data arriving through the Decoder is in dictionary format (as is the mainly approach for JSON data), but the Apply Schema expects tables. We need to transform the data. Therefore, let's use the dropdown on the left, looking for the Map cell. Map applies the map() function to the incoming data, transforming all data in the stream. In this case, as the output of the Decoder is a dictionary, and a table is expected, the only function to apply to the data is the q enlist.

> **This step must be done because the captured data is in JSON format. If we would have CSV-like streaming data, we wouldn't need to transform it to tables: as the usual interpretation of CSV is table-like.**

![Pipeline Map Step](assets/2023/12/19/basic_ppline_map.png)

We are capturing a lot of data in real time. To make the data loading easier, we will add an additional step to load it in batches.

> **Due to the high data density that can be received, it's better to introduce an intermediate step before writing, an Apply function. This function will progressively move the data to writing more "slowly," ensuring no issues with real-time database writing. This is a recommended practice whose purpose is to stop components from getting overloaded, which could lead to the processes getting OOM killed.**

![Pipeline Apply Step](assets/2023/12/19/basic_ppline_apply.png)

The magic of Insights lies precisely in the fact that, with a minimal amount of code, we have managed to deploy a real-time massive data ingestion pipeline. You don't always need to be technical to work with data!


### Querying the data

Once the pipeline is built, we can use the query engine to examine the data! To do so, let's navigate to the "Query" section, where we will encounter something like this:

![]({{ site.baseurl }}/assets/2023/12/19/query_basic.png)

Once here, we have three options for querying the data: using the API, employing SQL, or utilizing q. By using q, we can perform various transformations with simple commands, so let's proceed with it. As we observed in the pipeline, we are storing the data in the `main` table and querying from it.

![]({{ site.baseurl }}/assets/2023/12/19/query1.png)

We can even do a little analysis of the speed of the network data using the "Visual" window below, as follows:

![]({{ site.baseurl }}/assets/2023/12/19/query2.png)

And also, using PyKX we can check the different statistics of the table as easy as working with Pandas!

![]({{ site.baseurl }}/assets/2023/12/19/query3.png)


## The hardest; the easiest
Okay, so far, we have ingested and queried our data, and everything is working quite well. Why don't we make it a little trickier? Let's complicate our pipeline.

As a reminder, we have created two tables: `main` and `errs`. Now, we are going to save some data in the second one. Gathering information about the most errored users could be useful when intending to address their issues promptly, solve problems, and retain all clients. To achieve this, we will split our data stream into two different parts: the one mentioned earlier that fills the `main` table and the one we are going to discuss here. To store the errors, we are going to aggregate the data, intending to have the most accurate information about the errors.

To do so we will add a few more blocks to our pipeline. A new one has appeared, and that is:

- _Timer Window_: This one will store the streaming data by a specified time (in this case, 1 mins).

Once our _Timer Window_ has stored the 2 last minutes of data, we will aggregate it using a _Map_. The map function will aggregate by `imsi`, counting the times that the speed of some user is lower than a given threshold.

`0!select count0: count i by imsi, ts from data where uspeed < avg[uspeed] | dspeed < avg[dspeed] `

Once done so, we will store our data in the `errs` table. Using again _Apply Schema_ will work out.

With this, we would have something like this:

![]({{ site.baseurl }}/assets/2023/12/19/pipelineComplex.png)

As we can see, repeated modules have the same name, with a numeric identification. To change it in order to have it clearer, we can **Right Click** → **Rename Node**, and write down the desired name for each node. Then, we would have our pipeline finished, as something like this:

![]({{ site.baseurl }}/assets/2023/12/19/pipelineComplexFinal.png)

Now that we have set up our pipeline to load the `errs` table with data, it is time to check it out, and see if it is actually working. To do so, we just have to get back to the query engine and do so, but now, changing our code statement so we query from the new table.

![]({{ site.baseurl }}/assets/2023/12/19/query_errs.png)

And _voilà_, we have our errored users stored!
## Views
Last but not least, we have the Views section. This section allows us to create dashboards with the data we are ingesting through the pipeline and analyze it in several ways! The interface is pretty similar to the one our colleague [Óscar](https://www.habla.dev/blog/2023/10/03/Exploring-KX-Dashboards.html) explained in this post, so we are skipping most of the setup part. Nevertheless, there are two things that may have to be explained:

### Stream Connectors
As in the dashboards post, when you declare a view in KX Insights, you have to create a data source that will ingest data from a database. As said before, Insights have three different default databases: RDB, IDB, and HDB. As we are analyzing real-time data, most of the time we will ingest from RDB, as seen below (image from heatmap datasource).

![Views Sources](assets/2023/12/19/views_sources.png)

Also, as in the query engine, you can just use the Insights API to query the data from any of those databases, as you can see in the following image:

![Views API](assets/2023/12/19/views_api.png)

### Heatmap
First, we have created a heatmap that updates in real-time. This innovative heatmap dynamically illustrates user density across in this case Chicago. This live update feature ensures that the data displayed is always current, providing an accurate and up-to-the-minute view of network performance. This information enables us to optimize network coverage and capacity where it's most needed.

![Heatmap](assets/2023/12/19/heatmap_gr.png)

### Speed
We have also created a dashboard where we plot both the upload and download speeds of different connections in real-time. This gives us information about how our network is working in general terms. In essence, this speed tracking dashboard is a key component in our toolkit, enabling us to continuously monitor and improve the overall health and performance of our network.

![Volumes](assets/2023/12/19/volumes.png)

### Tracker
We have created a dashboard that acts as a tracker, as we can follow the locations of the different connections that the user makes. It maps out the geographical journey of users by pinpointing where and when they establish network connections. This level of detailed tracking is invaluable for understanding user mobility and network usage patterns.

![Tracker Zoomed](assets/2023/12/19/tracker_zoomed.png)

### Errored Users
We have created a treemap where we represent the users who have experienced the most errors in the last few minutes. By categorizing and visualizing the errors encountered by our users, this treemap enables us to quickly identify and address the most common and pressing issues. The real-time nature of the dashboard ensures that we are always aware of the current state of user experience, allowing for swift and proactive interventions.

![Errors](assets/2023/12/19/errors.png)


## Conclusions
Throughout our exploration, it has become abundantly clear that KX Insights Enterprise stands out as an exceedingly convenient and proficient tool for crafting bespoke data analysis services. Its capabilities extend far beyond mere simplification – it substantially streamlines the complexities inherent in pipeline construction, data querying, and visualization. Remarkably, this tool empowers us to undertake such endeavors without the extensive technical expertise traditionally associated with similar projects. Consequently, we now find ourselves in possession of a thoroughly comprehensive and fully operational real-time data service.

## Acknowledgements
Thanks to Stephen Meany for the constant technical support and Alfonso Campo for the data generation.

## Bibliography

- *The fastest and most efficient data analytics engine in the cloud.*, https://kx.com/products/kdb-insights-enterprise/.
- *Run machine learning model to create stock predictions in real-time*, https://code.kx.com/insights/1.8/enterprise/recipes/finance_ml.html.
- *Build and backtest scalable trading strategies using real time and historical tick data*, https://code.kx.com/insights/1.8/enterprise/recipes/finance.html.
- *Manufacturing Tutorial: Apply deep neural networks to streaming IoT data using MQTT and Tensorflow*, https://code.kx.com/insights/1.8/enterprise/recipes/manufacturing.html.
- *Integrate Kafka and kdb+ for real-time telemetry*, https://code.kx.com/insights/1.8/enterprise/recipes/kafkaread.html.
- *Drag and Drop Application Design with kdb Insights Enterprise, Eoin Killeen*, https://kx.com/videos/drag-and-drop-application-design-with-kdb-insights-enterprise-at-community-meetup-belfast/