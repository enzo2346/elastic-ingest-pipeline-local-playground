## Ingest Pipeline playground

This is a playground for testing ingest pipelines in Elasticsearch. It includes a simple HTTP server that listens on port 8080 and returns a fixed response. You can use this server to test your ingest pipeline by sending requests to it.

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed on your machine.

## Getting Started

1. Clone this repository:

```shell
git clone https://github.com/homelab-2025/elk-local-playground.git
```

2. Navigate to the project dir and start the Docker containers:

```shell
docker-compose up -d
```

Then wait for the containers to start up, it might take few minutes

3. Then you can log into Kibana at http://localhost:5601 and start creating the following ingest pipeline:

-  Go to "Stack Management" > "Ingest Node Pipelines" and click "Create pipeline".
-  Name the pipeline "apache_ingest_pipeline" without any processor and save it.

4. Now you can test the pipeline by sending requests to the HTTP server. You can use the following command to send requests every 5 seconds:

```bash
while true; do curl http://localhost:8080; sleep 5; done
```

5. You can then create a data view in Kibana to visualize the data being ingested. Go to "Stack Management" > "Data Views" and create a new data view that matches the index pattern of your ingest pipeline (e.g., "apache-*"). Once the data view is created, you can use it to explore the ingested data in Kibana's Discover, Visualize, and Dashboard sections.

You are now set up to test your ingest pipeline with the HTTP server. You can modify the ingest pipeline to include processors such as grok, date, or geoip to parse and enrich the incoming data.

Here are some additional steps you can take to try out the grok processor and enhance your ingest pipeline:

- Here is a list of common grok patterns you can use in your ingest pipeline: https://github.com/logstash-plugins/logstash-patterns-core/blob/main/patterns/ecs-v1/grok-patterns
- To test your grok patterns, you can use the Grok Debugger in Kibana: http://localhost:5601/app/dev_tools#/grokdebugger
- To test your pipeline you can add document in `Stack Management / Ingest Pipeline / Edit pipeline / Add documents` as follows:

```json
[
  {
    "_source": {
      "message": "172.25.0.1 - - [[08/Feb/2026:12:42:47 +0000]] \"GET / HTTP/1.1\" 200 191"
    }
  }
]
```

- Here is an example of a grok pattern you can use to parse the generated log messages from the HTTP server:

```
[
  {
    "drop": {
      "if": "ctx.message.contains('agent')"
    }
  },
  {
    "grok": {
      "field": "message",
      "patterns": [
        "%{IPORHOST:client.ip} %{USER:ident} %{USER:auth} \\[\\[%{HTTPDATE:apache.time}\\]\\] \"%{WORD:method} %{URIPATHPARAM:request_uri} HTTP/%{NUMBER:httpversion}\" %{NUMBER:response} %{NUMBER:bytes}"
      ]
    }
  }
]
```
