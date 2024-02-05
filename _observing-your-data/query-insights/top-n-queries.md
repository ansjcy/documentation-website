---
layout: default
title: Top n queries
parent: Query insights
nav_order: 65
---

# Top n queries

Monitoring top n queries in query insights can help you gain real-time insights into the top queries with high latency in a certain window (for example, the last hour). 

## Getting started

To enable top n queries monitoring, configure the following [dynamic settings]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/#dynamic-settings):

- `search.insights.top_queries.latency.enabled`: Set to `true` to [enable top n query monitoring](#enabling-top-n-queries).
- `search.insights.top_queries.latency.window_size`: [Configure the window size](#configuring-window-size). 
- `search.insights.top_queries.latency.top_n_size`: [Specify the n value](#configuring-top-n-size).

It's important to exercise caution when enabling this feature because it can consume system resources.
{: .important}


For detailed information about enabling and configuring top n queries, see the following sections.

## Enabling top n queries 

After installing the `query-insights` plugin, you can enable the top n queries feature (which is disabled by default) by using the following dynamic setting. This setting enables the corresponding query insights collectors and aggregators in the running cluster:

```json
PUT _cluster/settings
{
  "persistent" : {
    "search.insights.top_queries.latency.enabled" : true
  }
}
```
{% include copy-curl.html %}

## Configuring window size

You can configure the window size for the top N queries by latency with `search.insights.top_queries.latency.window_size`. For example, a cluster with the following configuration will collect top n queries in a 60-minute window:

```json
PUT _cluster/settings
{
  "persistent" : {
    "search.insights.top_queries.latency.window_size" : "60m"
  }
}
```
{% include copy-curl.html %}

## Configuring top n size

You can configure the top n size using `search.insights.top_queries.latency.top_n_size`. For example, a cluster with the following configuration will collect the top 10 queries in the specified window size:

```
PUT _cluster/settings
{
  "persistent" : {
    "search.insights.top_queries.latency.top_n_size" : 10
  }
}
```
{% include copy-curl.html %}

## Monitoring top n queries 

You can use the Insights API endpoint to obtain top n queries by latency:

```json
GET /_insights/top_queries
```
{% include copy-curl.html %}

Specify the type of metric you're interested in to filter the response by metric type (latency is the only supported type as of 2.12):

```json
GET /_insights/top_queries?type=latency
```
{% include copy-curl.html %}