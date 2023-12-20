---
layout: post
title: "Power-up your data analytics: KX Insights"
author: Cristian Pérez Corral, Christian Aberturas López
image: /img/butterfly-ship.jpg
toc: true
---


# Power-up your data analytics: KX Insights

KX Insights Enterprise is the fastest cloud-based platform for time series analysis, leveraging the power of kdb+ vector databases. But it's not just that; throughout this post, we will demonstrate how, without requiring advanced technical knowledge, you can develop data ingestion, query engine, and visualization—all thanks to this tool. Will you join us?

## Overview

Let's get, before the analysis, through some preliminary steps, such as creating the database or data ingestion.

For the development, we will use a specific use case—a telecommunications network. It has been chosen for two reasons: the data is temporal (thus allowing the use of time series analysis tools) and high density. With this, we will not only showcase the power of Insights but also the underlying machinery: the kdb+ database.


Once we enter our instance of Insights Enterprise, we encounter the following screen.

![]({{ site.baseurl }}/assets/2023/12/19/overview.png)

The parts that will compose this post are those visible on the screen, addressing each one individually for development.
## Databases


Once we have accessed the databases tab, we can configure it to suit our specific needs. We will leave the technical specifications (memory, storage, etc.) at their default settings, thus avoiding the technical details.

Our database (which we will name db-telco) will have two tables: `main` and `errors`. The first one is where real-time telephony data will be stored, and the second one will be used for a subsequent grouping to study errors in the network. The `main` table will have 10 columns, which are:

-   `ts`: The timestamp of the network trace for the record (timestamp type).
-   `cellID`: The ID of the cell on which the trace is being transmitted (symbol type).
-   `phase`: I DON'T REMEMBER WHAT PHASE WAS.
-   `imsi`: The ID of the SIM card participating in the trace (string type).
-   `imei`: The ID of the device participating in the trace (string type).
-   `dspeed`: The download speed of the network at this moment, cell, and event (float type).
-   `uspeed`: The upload speed of the network at this moment, cell, and event (float type).
-   `lat`: The latitude of the device from which the event is taking place (float type).
-   `lng`: The longitude of the device from which the event is taking place (float type).
-   `cellDistance`: The distance to the cell of the device (float type).

![]({{ site.baseurl }}/assets/2023/12/19/schema_main.png)

And the `errs` table has 3 (we have already seen `ts` and `imsi`):

-   `count0`: This counts the errors that have occurred in the network in the grouping (int type).

![]({{ site.baseurl }}/assets/2023/12/19/schema_errs.png)

After creating the schemas for the tables, we save and click on deploy, and with that, we have the database up and running!

##  Pipelines (I)

When it comes to data ingestion, pipelines are employed. The tools provided by KX Insights Enterprise for the creation, analysis, and testing of pipelines are extensive and straightforward to use, as we will see below. The more basic pipeline have four fundamental parts, which can then be further complicated or supplemented with others, namely:

- Reader: The reader for Insights pipelines is highly versatile, allowing data ingestion from multiple sources, as depicted in Figure 1. (MAKE REFERENCE HERE SOMEHOW)
- Decoder: The decoder, responsible for transforming data received through the reader into different formats (JSON, CSV, etc.) into a q structure.
- Apply Schema: Shapes the incoming data through ingestion. It can be any type of data, tabular, or an array.
- Writer: This last component, as its name suggests, is responsible for writing the received data into a table in a previously created database.
Let's create a pipeline for the ingestion of telephone data, provided by a Kafka service. To achieve this, having chosen Kafka as the data source (selecting the corresponding broker and topic), JSON as the decoder, the schema of the `main` table seen earlier, and writing to that table, we end up with the following:

![]({{ site.baseurl }}/assets/2023/12/19/basic_ppline.png)

Before having a functional pipeline, a couple of things need to be done.

The data arriving through the Decoder is in dictionary format (the way JSON is cast), but the Apply Schema expects tables. Therefore, let's use the dropdown on the left, looking for the Map cell. Map applies the map() function to the incoming data, transforming all data in the stream. In this case, as the output of the Decoder is a dictionary, and a table is expected, the only function to apply to the data is the q enlist.

![]({{ site.baseurl }}/assets/2023/12/19/basic_ppline_map.png)

> :warning: **due to the high data density that can be received, it's better to introduce an intermediate step before writing, an Apply function. This function will progressively move the data to writing more "slowly," ensuring no issues with real-time database writing.**

![]({{ site.baseurl }}/assets/2023/12/19/basic_ppline_apply.png)

The magic of Insights lies precisely in the fact that, with a minimal amount of code, we have managed to deploy a real-time massive data ingestion pipeline. You don't always need to be technical to work with data!


## Query (I)

Once the pipeline is built, we can use the query engine to examine the data! To do so, let's navigate to the "Query" section, where we will encounter something like this:

![]({{ site.baseurl }}/assets/2023/12/19/query_basic.png)

Once here, we have three options for querying the data: using the API, employing SQL, or utilizing q. By using q, we can perform various transformations with simple commands, so let's proceed with it. As we observed in the pipeline, we are storing the data in the `main` table and querying from it.

![]({{ site.baseurl }}/assets/2023/12/19/query1.png)

We are receiving the data without any issues! Even the schema of the current table is the one defined in the Databases section!

## Pipelines (II)
Okay, so far, we have ingested and queried our data, and everything is working quite well. Why don't we make it a little trickier? Let's complicate our pipeline.

As a reminder, we have created two tables: `main` and `errs`. Now, we are going to save some data in the second one. Gathering information about the most errored users could be useful when intending to address their issues promptly, solve problems, and retain all clients. To achieve this, we will split our data stream into two different parts: the one mentioned earlier that fills the `main` table and the one we are going to discuss here. To store the errors, we are going to aggregate the data, intending to have the most accurate information about the errors.

To do so we will add a few more blocks to our pipeline. A new one has appeared, and that is:

- _Timer Window_: This one will store the streaming data by a specified time (in this case, 2 mins).

Once our _Timer Window_ has stored the 2 last minutes of data, we will aggregate it using a _Map_. The map function will aggregate by `imsi`, counting the times that the speed of some user is lower than a given threshold.

`0!select ts, count0: count i by imsi from data where uspeed < avg[uspeed]-sdev uspeed`

Once done so, we will store our data in the `errs` table. Using again _Apply Schema_ will work out.

![]({{ site.baseurl }}/assets/2023/12/19/pipelineComplex.png)
## Query (II)
Now that we have set up our pipeline to load the `errs` table with data, it is time to check it out, and see if it is actually working. To do so, we just have to get back to the query engine and do so, but now, changing our code statement so we query from the new table.

![]({{ site.baseurl }}/assets/2023/12/19/query_errs.png)

And _voilà_, we have our errored users stored!
## Views
Last but no least, we have the Views section. This section allows us to create dashboards with the data we are ingesting through the pipeline, and analyze it in several ways! The interface is pretty similiar as the one our colleague [Óscar](https://www.habla.dev/blog/2023/10/03/Exploring-KX-Dashboards.html) explained in this post, so we are skipping the setup part. Instead, we are going to show some possibilities to analyze this data.

### Heatmap
First, we have created a heatmap that updates in real-time. This innovative heatmap dynamically illustrates user density across in this case Chicago. This live update feature ensures that the data displayed is always current, providing an accurate and up-to-the-minute view of network performance. This information enables us to optimize network coverage and capacity where it's most needed.

![]({{ site.baseurl }}/assets/2023/12/19/heatmap_gr.png)

### Speed
We have also created a dashboard where we plot both the upload and download speeds of different connections in real-time. This gives us information about how our network is working in general terms. In essence, this speed tracking dashboard is a key component in our toolkit, enabling us to continuously monitor and improve the overall health and performance of our network.

![]({{ site.baseurl }}/assets/2023/12/19/volumes.png)

### Tracker
We have created a dashboard that acts as a tracker, as we can follow the locations of the different connections that the user makes. It maps out the geographical journey of users by pinpointing where and when they establish network connections. This level of detailed tracking is invaluable for understanding user mobility and network usage patterns.

![]({{ site.baseurl }}/assets/2023/12/19/tracker_zoomed.png)

### Errored users
We have created a treemap where we represent the users who have experienced the most errors in the last few minutes. By categorizing and visualizing the errors encountered by our users, this treemap enables us to quickly identify and address the most common and pressing issues. The real-time nature of the dashboard ensures that we are always aware of the current state of user experience, allowing for swift and proactive interventions.

![]({{ site.baseurl }}/assets/2023/12/19/errors.png)