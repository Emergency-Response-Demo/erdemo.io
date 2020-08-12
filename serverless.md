---
layout: page
title: Serverless
subtitle: Knative Serving and Eventing
---

# 1. Introduction

Knative extends facilities in Openshift and Kubernetes to provide a scalable environment for deploying services that scale up and down based on demand. 
It also provides an eventing mechanism to provide decoupled messaging to applications running inside an Openshift or Kubernetes cluster.

Knative consists of the following components:

* **Serving - Request-driven compute that can scale to zero**
  Knative Serving is ideal for running application services inside OpenShift by providing a more simplified deployment syntax with automated scale-to-zero and scale-out <span style="color:blue">based on HTTP load </span>. The Knative platform will manage your service’s deployments, revisions, networking and scaling. Knative Serving exposes your service via an HTTP URL. As the Knative Serving Service has the built in ability to automatically scale down to zero when not in use, it is apt to call it as “serverless service”. Knative Serving is mostly adapted for stateless applications with minimal or no dependencies on other systems.


* **Eventing - Management and delivery of events to and from Knative services**
  Knative Eventing designed to decouple event sources and event consumers. Knative Eventing services are loosely coupled and can be developed and deployed independently. Event producers and event consumers are independent. Any producer (or source), can generate events before there are active event consumers that are listening. Any event consumer can express interest in an event or class of events, before there are producers that are creating those events. Cross-service interoperability is provided by adhering to the CloudEvents specification that is developed by the Cloud Native Computing Foundation Serverless working Group.


In the Emergency Response Demo application, Knative is installed by default.  In addition, both Knative Serving and Knative Eventing is implemented as described in the following sections:

# 2. Knative Serving:  **Find-My-Relative Service**

```
$ ansible-playbook \
    -i inventories/inventory playbooks/install.yml \
    -e project_admin=user1 \
    -e deploy_from=source \
    -e provision_find_service=true
```

   
# 3. Knative Eventing: **Mission Analytics Service**  


[07_1_Serverless_Lab](https://github.com/redhat-gpe/cnd_advanced_v2/blob/master/modules/07_Serverless/07_1_Serverless_Lab.adoc)
