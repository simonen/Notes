---
tags:
  - logstash
  - beats
  - kibana
  - elasticsearch
---
[Elastic Repo](https://www.elastic.co/downloads/beats/filebeat)

`James Turnbull - The Logstash Book`

This docs is based on ELK stack v5.6 and Java 8 which are outdated.

`/etc/yum.repos.d/elastic.repo`
```
[elasticsearch-5.x]
name=Elasticsearch repository for 5.x packages
baseurl=https://artifacts.elastic.co/packages/5.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

Packages:
`java-1.8.0-openjdk-1.8.0.362.b09-4.el9.x86_64`

Elastic.co maintains Beats, Logstash and Elasticsearch and Kibana

- **Analysis**: The study of constituent parts and their interrelationships in making up the whole
Correlation: The act of detecting relationships between data. 

Data collection -> Filtering "wheat from the chaff" -> correlation of the remaining data in order to provide accurate analysis

- Stage 1: List out all applications, devices and hosts and where they log.
- Stage 2: Bring together all that information and work out what the important messages are
- Stage 3: Implement log correlation and analysis 

Pipeline:
* **Input plugins** handle data ingestion. 
* **Codec plugins** are used to change the data representation of an event or stream filter. 
* **Filters** allow for processing of events before sending the output. 
* **Outputs** write the outputs of the Logstash to a stash or storage like `ElasticSearch`. 

#### Beats

Repo: `[elasticsearch-5.x]`

Package:
`filebeat-5.6.16-1.x86_64`

Configuration file:
`/etc/filebeat.yml`

Beats are lightweight forwarders of logs. metrics, network packet data, and Windows events.
Lightweight - means they just ship data to a remote location, keeping track of what has been shipped.

`/etc/filebeat/filebeat.yml`
```
filebeat.prospectors:

- input_type: log
    - /var/log/*.log
    - /var/log/messages
  tags: ["security", "network"]
  fields:
    env: production
- input_type: log
  paths:
    - /var/log/audit/audit.log
  document_type: auditd

output.logstash:
  hosts: ["db1.ohio.cc:5044"]
  ssl.certificate_authorities: ["/etc/pki/tls/certs/cacert.pem"]
  ssl.certificate: "/etc/pki/tls/certs/headoffice.ohio.cc.crt"
  ssl.key: "/etc/pki/tls/certs/headoffice.ohio.cc.key"
```
#### Logstash

Repo: `[elasticsearch-5.x]`

Package:
`logstash-5.6.16-1.noarch`

Configuration file: 
- `/etc/logstash/logstash.yml`
- `/etc/logstash/conf.d/.conf`

`Logstash`: A tool that can transform logs into data that can be indexed and tagged and then stored (or shipped again) to make discovery easier. Has a variety of inputs and outputs that can write to `Elasticsearch`. 

The `logstash` user must have access to the private key. If this is a single-node or non-critical environment, you can try to **delete the node metadata** to force Elasticsearch to recreate it:

```
sudo mv /var/lib/elasticsearch/nodes /var/lib/elasticsearch/nodes.bak
```

Minimal `Logstash` configuration. This will output incoming beats messages to STDOUT

`/etc/logstash/conf.d/general.conf`
```
input {
  beats {
    port => 5044
    ssl => true
    ssl_certificate => "/etc/pki/tls/certs/db1.ohio.cc.cert"
    ssl_key => "/etc/pki/tls/certs/db1.ohio.cc.key"
    ssl_certificate_authorities => ["/etc/pki/tls/certs/cacert.pem"]
    ssl_verify_mode => "force_peer"
  }
}
output {
  stdout { codec => rubydebug }
  }
```

Start the service

```
systemctl enable --now logstash.service
```

Query the Logstash API to test the server

``` bash
curl -XGET 'http://localhost:9600/?pretty'
```

Filters can be added to transform the incoming messages. The `grok` function will turn 
`msg=audit(1729534201.545:187)` into `audit_epoch: 1729534201`, `audit_counter: 187`

`/etc/logstash/conf.d/general.conf`
```
filter {
  if [type] == "auditd" {
    kv { }   # this will turn auditd document_type entries in key: value pairs
    
    grok {
      match => { "msg" => "audit\(%{NUMBER:audit_epoch}:%           {NUMBER:audit_counter}\):" }
    }

    mutate {
      rename => {
        "ses" => "session_id"
      }
    }
  }
}
```

To output data to Elasticsearch

```
output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
#   index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
  }
}

```

If no index format is specified, it defaults to `logstash-%{+YYYY.MM.dd}`
#### Elasticsearch for Log Stashing

Repo:
`[elasticsearch-5.x]`

Package:
`elasticsearch-5.6.16-1.noarch`

- `Index`: Logical namespace for data. Made up of documents, which are equivalent of relational database rows. Each index has a mapping that defines the types in the index and other index settings.
- `type`: A type is a the type of document, like a `user` or `log`, and is used by the API as a filter.
- `document`: JSON object. Has a `type` and `ID`. Made up of one or more key/value pairs

Each document is stored in one primary `shard`. Shards are distributed among the nodes in the Elasticsearch cluster. For distributed systems, it is always good to deploy in odd numbers.

Elasticsearch listens on http port 9200

Check if ElasticSearch has data

``` bash
curl -X GET "http://localhost:9200/_cat/indices?v"
```

```
health status index               uuid                   pri rep 
yellow open   logstash-2024.10.21 5e224TP0R9KpCRO6sS7K1g   5   1
```

#### Kibana

Package
`kibana-5.6.16-1.x86_64`

Port
5601

Minimal configuration

`/etc/kibana/kibana.yml`
```
server.port: 5601
server.host: "192.168.137.12"
```

Add an `index pattern` in Management tab

Query Elasticsearch for index patters. Example index: `logstash-2024.10.21`, so index pattern should be `logstash-*`
