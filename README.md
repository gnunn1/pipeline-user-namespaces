### Introduction

This is a simple demonstration of using user namespaces in Pipelines to support test containers.

### Prerequisites

* OpenShift Cluster 4.21+
* OpenShift Pipelines 1.24+

### Deploy Example

Deploy the demo using the following command, note this will deploy some 
cluster-scoped resources so you must have the ability to do so:

```
oc apply -k .
```

### Run Pipeline

To run the pipeline, use the provided pipelinerun-user-namespaces.yaml file:

```
oc create -f pipelinerun-user-namespaces.yaml
```
