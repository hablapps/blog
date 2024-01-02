---
layout: post
title: "KX Insights: data analytics made simple"
author: Cristian Pérez Corral, Christian Aberturas López
image: /img/butterfly-ship.jpg
toc: true
---


# KX Insights: data analytics made simple

This post serves as a concise exploration of the versatile capabilities within **KX Insights**, showcasing its role as the premier cloud-based platform for swift time series analysis. Leveraging the robust capabilities of kdb+ vector databases, this tool stands out not only for its exceptional speed but also for empowering users, even those without advanced technical expertise, to seamlessly develop data ingestion processes, query engines, and visualizations. The sections will cover:

- The [Use Case](#use-case): We will delve into a specific use case by employing an illustrative example using telecommunication network data. The simulated data tailored for this use case ensures that readers can easily follow the content at their own pace.

- Simple Pipeline and Encounter with the Query Engine: The initial technical segment, titled "[*Was it always this easy?*](#was-it-always-this-easy)" will guide you through establishing a straightforward data pipeline. It explores the steps involved in gathering key stream data and demonstrates how to leverage KX Insights' capabilities for insightful analysis. This section aims to highlight the simplicity of working with the tool.

- Pipeline Bifurcation: The subsequent technical segment, labeled "[*The hardest; the easier*](#the-hardest-the-easiest)," takes a deeper dive. It introduces an ad-hoc approach to make the pipeline more intricate, showcasing the tool's adaptability and advanced capabilities. This section aims to reveal the tool's capacity to handle complex scenarios and provide solutions even when faced with challenges.

> ℹ️ *It's essential to note that the data presented is specifically crafted for this illustrative example and does not originate from real telecommunication sources. You will be provided with this simulated data to facilitate your engagement with the post.*

As we progress through these sections, you will gain a comprehensive understanding of how KX Insights transforms intricate time series analysis tasks into manageable and insightful processes, making it an invaluable asset for both seasoned professionals and those new to the field.

For our development journey, we've opted to leverage the use case outlined earlier for two compelling reasons: the data's temporal nature, allowing for the utilization of time series analysis tools, and its high density. By doing so, we aim not only to demonstrate the prowess of KX Insights but also to shed light on the underlying machinery powering it—the kdb+ database.

Assuming a Kafka server has already been deployed to store and manage the incoming data from the telco network, we'll subscribe to the corresponding topic to seamlessly capture the data.

This post will guide you through the intricacies of constructing a data pipeline in KX Insights, encompassing key stages such as defining the database schema, streaming data ingestion, and visualizing and querying data. Organizing our discussion around the aforementioned use case, we'll first develop a simplified pipeline to capture the foundational aspects of the data later visualized. Subsequently, we'll delve into refining our initial pipeline, exploring the extensive possibilities offered by this versatile tool. Let the journey begin!

## Use Case
Let's suppose we are a telecommunications (telco) company immersed in the dynamic world of high-density real-time telco data. In that case, we would want to empower our Data Team with a robust system for in-depth data analysis and insightful conclusions. To do so, we will create comprehensive dashboards that provide a visual representation of key metrics. We have identified specific elements that will be pivotal in enhancing this operational understanding.

### User Distribution Heatmap:
Our objective is to gain insights into the geographical distribution of our users in real-time. The User Distribution Heatmap within KX Insights will vividly illustrate the concentration of our current users, allowing us to identify high-demand areas promptly.

### Network Performance Analysis (Line Graph):
Understanding the impact of network volume on speed is critical for ensuring a seamless user experience. Through KX Insights, we plan to deploy a Line Graph that dynamically illustrates how network volume influences speed, enabling us to optimize our network performance proactively.

### User Location Tracker:
The ability to locate users within a specific geography is essential for targeted interventions and optimizations. KX Insights will provide real-time information on the geographical whereabouts of our users, facilitating strategic decision-making.

### Failure Analysis TreeMap:
Identifying and resolving issues promptly is key to maintaining a high-quality service. A TreeMap, powered by KX Insights, will pinpoint users experiencing difficulties with their connection. This enables us to swiftly address issues and ensure a seamless communication experience for our users.

Our choice of KX Insights is driven not only by the nature of our data, characterized by a vast volume of real-time information, but also by the platform's exceptional capabilities in constructing functional pipelines, executing complex queries, and delivering compelling visualizations. With KX Insights, we are equipped to harness the full potential of our telco data, providing our Data Team with the tools they need to navigate and derive actionable insights from this dynamic landscape.

Now, let's consider the scenario we have plotted with this use case. Let's suppose we would need to construct a similar architecture using only kdb+/q, and other KX tools. While we have a plethora of tools at our disposal, the question arises: How challenging would this endeavor be? Here, we succinctly outline some of the crucial steps that must be undertaken:

1. **Build a Kafka Consumer:**
   Constructing a Kafka consumer entails delving into the [API](https://github.com/KxSystems/kafka), understanding its intricacies, and developing the necessary infrastructure to facilitate seamless data consumption.

2. **Orchestrate the Pipeline:**
   Docking the pipeline involves implementing all the transformations essential for processing real-time data. This step requires careful consideration and meticulous execution.

3. **Develop Graphic Tools:**
   While the [_Grammar of Graphics_](https://code.kx.com/analyst/libraries/grammar-of-graphics/) package is a powerful tool, we would still need to create a system for visualizing the aggregated data. We could use [KX Dashboards](https://code.kx.com/dashboards/) to power this process, but still we would have two different applications, with the IPC technical specifications it could lead to.

Beyond these tasks, there's the challenge of working with an event handler for streaming data, which is not always straightforward. Additionally, packaging all these components into a black-box-like application is necessary to empower the entire data team to work with it, regardless of their technical expertise. Moreover, generalizing this solution for any conceivable use case poses an even more significant undertaking.

##  Was it always this easy?

We will develop a solution using [KX Insights Enterprise](https://kx.com/products/kdb-insights-enterprise/). This solution wraps the complexity in a user-friendly web interface, eliminating the need for the data team to possess intricate technical knowledge. Now, anyone on the team can seamlessly work with the data, thanks to the intuitive interface provided by KX Insights Enterprise.

To walk all the reader through this tool as easy as possible, we will follow the recommended steps of KX Insights, that can be seen in the following image:

![]({{ site.baseurl }}/assets/2023/12/19/overview.png)

 ### Deploying the database
 
Once we have accessed the databases tab, we can configure it to suit our specific needs. We will leave the technical specifications (memory, storage, etc.) at their default settings, thus avoiding the technical details.

Our database (which we will name db-telco) will have the `main` table. This one is where real-time telephony data will be stored (and from where the heatmap, tracker and volume views will capture the data), and it will have 10 columns, which are: 
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

![]({{ site.baseurl }}/assets/2023/12/19/schema_main.png)

After creating the schemas for the table, we save and click on deploy, and with that, we have the database up and running!


### Data slide: the pipeline
When it comes to data ingestion, pipelines are employed. The tools provided by KX Insights Enterprise for the creation, analysis, and testing of pipelines are extensive and straightforward to use, as we will see below. Constructing the pipelines using the web interface (i.e, going to _Import Data_ in the Overview window) is pretty straightforward, and gives us a pretty basic pipeline that will have four fundamental parts, which can then be further complicated or supplemented with others, namely:

- **Reader:** The reader for Insights pipelines is highly versatile, allowing data ingestion from multiple sources, as depicted in below image.
- **Decoder:** The decoder, responsible for transforming data received through the reader into different formats (JSON, CSV, etc.) into a q structure.
- **Apply Schema:** Shapes the incoming data through ingestion. It can be any type of data, tabular, or an array.
- **Writer:** This last component, as its name suggests, is responsible for writing the received data into a table in a previously created database.

Let's create a pipeline for the ingestion of telephone data, provided by a Kafka service. To achieve this, having chosen Kafka as the data source (selecting the corresponding broker and topic), JSON as the decoder, the schema of the `main` table seen earlier, and writing to that table, we end up with the following:

![]({{ site.baseurl }}/assets/2023/12/19/basic_ppline.png)

We will be writing in a database stored in the KX Insights architecture, but this is not the only place we can store the captured data. Some of the others are a Kafka Topic, Amazon S3... If you want to go deeper in the pipeline operators, you can follow the [documentation](https://code.kx.com/insights/1.8/enterprise/ingest/pipeline/operators/index.html).

We will need to know is how is the data stored, in order to query it efectively. KX Insights supports three types of databases: **RDB, IDB and HDB**. Each one of then store a type of data:
- RDB stores only real time data, and just a little amount of timelapse.
- IDB stores data as intervals from the real time, to time to time whe pipeline will send a signal indicating either the start or the end of a new interval and then all the data will be stored in IDB as batches.
- HDB stores the historic data. Once enough time has passed since the data was stored in IDB, it moves directly to be stored in HDB.

Then, the streaming data will go through  **RDB → IDB → HDB**.

As we are doing a real-time process, we are going to work mainly with RDB, as we will see briefly.

If you want to know more about this, please check the [documentation](https://code.kx.com/insights/1.8/microservices/database/storage/index.html#stream-partitioning-idb-eoi).

Before having a functional pipeline, a couple of things need to be done.

As we are getting JSON-format data, we will have to decode it. The data arriving through the Decoder is in dictionary format (as is the mainly approach for JSON data), but the Apply Schema expects tables. We need to transform the data. Therefore, let's use the dropdown on the left, looking for the Map cell. Map applies the map() function to the incoming data, transforming all data in the stream. In this case, as the output of the Decoder is a dictionary, and a table is expected, the only function to apply to the data is the q enlist.

> ⚠️*This step must be done because the captured data is in JSON format. If we would have CSV-like streaming data, we wouldn't need to transform it to tables: as the usual interpretation of CSV is table-like.*

![]({{ site.baseurl }}/assets/2023/12/19/basic_ppline_map.png)

We are capturing a lot of data in real time. To make the data loading easier, we will add an additional step to load it in batches.

> ⚠️*Due to the high data density that can be received, it's better to introduce an intermediate step before writing, an Apply function. This function will progressively move the data to writing more "slowly," ensuring no issues with real-time database writing. This is a recommended practice whose purpose is to stop components from getting overloaded, which could lead to the processes getting OOM killed.*

![]({{ site.baseurl }}/assets/2023/12/19/basic_ppline_apply.png)

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

### Enjoying the views

Now that we have stored the data, what about analysing it? This section allows us to create dashboards with the data we are ingesting through the pipeline and analyze it in several ways! The interface is pretty similar to the one our colleague [Óscar](https://www.habla.dev/blog/2023/10/03/Exploring-KX-Dashboards.html) explained in this post, so we are skipping most of the setup part. Nevertheless, there are two things that may have to be explained:

### Stream Connectors
As in the dashboards post, when you declare a view in KX Insights, you have to create a data source that will ingest data from a database. As said before, Insights have three different default databases: RDB, IDB, and HDB. As we are analyzing real-time data, most of the time we will ingest from RDB, as seen below (image from heatmap datasource).

![]({{ site.baseurl }}/assets/2023/12/19/views_sources.png)

Also, as in the query engine, you can just use the Insights API to query the data from any of those databases, as you can see in the following image:

![]({{ site.baseurl }}/assets/2023/12/19/views_api.png)

### Heatmap
First, we have created a heatmap that updates in real-time. This innovative heatmap dynamically illustrates user density across in this case Chicago. This live update feature ensures that the data displayed is always current, providing an accurate and up-to-the-minute view of network performance. This information enables us to optimize network coverage and capacity where it's most needed.

![]({{ site.baseurl }}/assets/2023/12/19/heatmap_gr.png)

### Speed
We have also created a dashboard where we plot both the upload and download speeds of different connections in real-time. This gives us information about how our network is working in general terms. In essence, this speed tracking dashboard is a key component in our toolkit, enabling us to continuously monitor and improve the overall health and performance of our network.

![]({{ site.baseurl }}/assets/2023/12/19/volumes.png)

### Tracker
We have created a dashboard that acts as a tracker, as we can follow the locations of the different connections that the user makes. It maps out the geographical journey of users by pinpointing where and when they establish network connections. This level of detailed tracking is invaluable for understanding user mobility and network usage patterns.

![]({{ site.baseurl }}/assets/2023/12/19/tracker_zoomed.png)

Up until now, our data ingestion has revolved around high-density, real-time data utilizing the fundamental modules offered by KX Insights Enterprise. Moreover, we've successfully illuminated the insights derived from this data through the integration of Dashboards. Consequently, the inquiry arises: how can we further enhance this process?

## The hardest; the easiest
We have ingested and queried our data, and everything is working quite well. Why don't we make it a little trickier? In this section we will comment briefly a little upgrade to the pipeline built in the previous one, following the same structure (i.e., database → pipeline → query → dashboard). Shall we begin?


Let's create the `errs` table. This new table will be used for a subsequent grouping to study errors in the network (and assist the most errored users). It will have 3 (we have already seen `ts` and `imsi`):

-   `count0`: This counts the errors that have occurred in the network in the grouping (int type).

![]({{ site.baseurl }}/assets/2023/12/19/schema_errs.png)

Presently, our focus shifts towards storing data in this context. The collection of information concerning the most errored users holds significant utility for promptly addressing their concerns, resolving issues, and maintaining client satisfaction. This entails bifurcating our data stream into two distinct segments: the previously highlighted stream that populates the `main` table, and the one under consideration here. To facilitate error storage, we intend to aggregate the data, aiming for the utmost precision in error-related information.

To aggregate the data, we will use a new module:

- _Timer Window_: This one will store the streaming data by a specified time (in this case, 1 mins).

Once our _Timer Window_ has stored the last minute of data, we will aggregate it using a _Map_. The map function will aggregate by `imsi`, counting the times that the speed of some user is lower than a given threshold.

`0!select count0: count i by imsi, ts from data where uspeed < avg[uspeed] | dspeed < avg[dspeed] `

Once done so, we will store our data in the `errs` table. Using again _Apply Schema_ will work out.

![]({{ site.baseurl }}/assets/2023/12/19/pipelineComplex.png)

As we can see, repeated modules have the same name, with a numeric identification. To change it in order to have it clearer, we can **Right Click** → **Rename Node**, and write down the desired name for each node. Then, we would have our pipeline finished, as something like this:

![]({{ site.baseurl }}/assets/2023/12/19/pipelineComplexFinal.png)

Having established our pipeline for loading the `errs` table with data, the subsequent step involves verification to ensure its functionality. To accomplish this, it is necessary to return to the query engine and modify our code statement accordingly, querying from the newly created table to assess its operational status.

![]({{ site.baseurl }}/assets/2023/12/19/query_errs.png)

And _voilà_, we have our errored users stored!

To visualize the errors, we have created a treemap where we represent the users who have experienced the most errors in the last few minutes. By categorizing and visualizing the errors encountered by our users, this treemap enables us to quickly identify and address the most common and pressing issues. This dashboards does not use the RDB, but the IDB in order to aggregate through more time, so we are always aware of the last minutes state of user experience, allowing for swift and proactive interventions.

![]({{ site.baseurl }}/assets/2023/12/19/errors.png)


## Conclusions
Throughout our exploration, it has become abundantly clear that KX Insights Enterprise stands out as an exceedingly convenient and proficient tool for crafting bespoke data analysis services. Its capabilities extend far beyond mere simplification – it substantially streamlines the complexities inherent in pipeline construction, data querying, and visualization. Remarkably, this tool empowers us to undertake such endeavors without the extensive technical expertise traditionally associated with similar projects. Consequently, we now find ourselves in possession of a thoroughly comprehensive and fully operational real-time data service.

## Acknowledgements
Acknowledgments to Alfonso Campo for the creation of the simulated data and to Stephen Meany for the unwavering technical support provided.

## User Guide

Should you wish to gain a more profound understanding of this tool, we strongly recommend following the latest KX Academy demonstrations pertaining to the tool. While certain aspects covered in this post may coincide with these demonstrations, leveraging all accessible resources will enhance your ability to construct more extensive data-oriented systems.

- *Build and backtest scalable trading strategies using real time and historical tick data*, https://code.kx.com/insights/1.8/enterprise/recipes/finance.html.
- *Manufacturing Tutorial: Apply deep neural networks to streaming IoT data using MQTT and Tensorflow*, https://code.kx.com/insights/1.8/enterprise/recipes/manufacturing.html.
- *Integrate Kafka and kdb+ for real-time telemetry*, https://code.kx.com/insights/1.8/enterprise/recipes/kafkaread.html.
- *Drag and Drop Application Design with kdb Insights Enterprise, Eoin Killeen*, https://kx.com/videos/drag-and-drop-application-design-with-kdb-insights-enterprise-at-community-meetup-belfast/

Should you desire to delve further into the technical aspects, you have the option to construct your own data ingestion process utilizing KX Insights, bypassing reliance on the web interface. The guidance provided in this post w:ill assist you in navigating this undertaking
- *Getting Started with kdb Insights*, https://www.treliant.com/knowledge-center/getting-started-with-kdb-insights/.