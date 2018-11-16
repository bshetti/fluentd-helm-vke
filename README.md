# Kubernetes Logging with Fluentd

This repository is a configuration to send kubernetes cluster and application logs using fluentd
In addition this is generated to work with VKE production cluster using kubernetes v1.10.8-1.

NOTE: This has been tested with AWS Elasticseach in a public configuration for ease of use.
Please use the standard ways to secure elasticsearch per AWS documentation.
Two such options
- elasticsearch deployed in a VPC deployment 
- using Cognito to configure user/password for access into AWS elasticsearch in the public config.

In either case the best way to access them is to use a proxy. We have included a proxy if you secure the ES configuration. Notes below

[Fluentd](http://fluentd.org) is a an extensible __Log Processor__ that comes with full support for Kubernetes:

- Read Kubernetes/Docker log files from the file system or through Systemd Journal
- Enrich logs with Kubernetes metadata
- Deliver logs to third party storage services like Elasticsearch, InfluxDB, HTTP, etc.

This repository contains a set of Yaml files to deploy Fluentd using helm stable/chart for fluentd-elastcisearch

## Getting started

[Fluentd](http://fluentd.org) must be deployed as a DaemonSet, so on that way it will be available on every node of your Kubernetes cluster. To get started run the following commands to create the namespace:

```
$ kubectl create namespace logging
```

Install helm
```
$ helm init
```

#### Fluentd to Elasticsearch

Regardless of using or not using cognito, the following configuration will work for both configurations.
It uses key and shared key to access ES cluster.

1)First run the es-proxy

Change the following parameters in the es-proxy-deployment.yaml file
with your parameters

```
        -name: AWS_ACCESS_KEY_ID
          value: "YOURAWSACCESSKEY"
        - name: AWS_SECRET_ACCESS_KEY
          value: "YOURAWSSECRETACCESSKEY"
        - name: ES_ENDPOINT
          value: "YOURESENDPOINT"
```

Now run:
```
$ kubectl create -f ./output/es-proxy/es-proxy-deployment.yaml
$ kubectl create -f ./output/es-proxy/es-proxy-service.yaml
```

Now run the following:
```
$ helm install --name my-release -f values-es.yaml stable/fluentd-elasticsearch --namespace=logging
```

