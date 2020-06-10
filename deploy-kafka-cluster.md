# Deploy Kafka Cluster using AMQ Stream (Strimzi)

This section shows how to deploy Kafka cluster on OCP 4

## Prerequisite
You need cluster admin role to Deploy Kafka Operator

## Step 1: Install AMQ Stream Operator on your Namespace

1. Go to Administrator > Operator > OperatorHub. search AMQ Stream Operator
2. Deploy Operator to your namespace

## Step 2: Install Kafka cluster on your Namespace
1. Go To Developer > Topology
2. Right click on topology > add new application from catalog > choose AMQ Stream > Create Kafka cluster.
> Note: use default configuration eg: cluster name : my-cluster. This operator will create 3 pods of Zookeeper and 3 pods of Kafka brokers 

## Optional - Step 3: Deploy Quarkus Application for Kafka Client
1. Go To Developer > Topology
2. Right click on topology > add new application from container images > quay.io/efeluzy/tutorial-kafka-quarkus:v2

