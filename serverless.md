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
  Knative Eventing is designed to decouple event sources and event consumers. Knative Eventing services are loosely coupled and can be developed and deployed independently. Event producers and event consumers are independent. Any producer (or source), can generate events before there are active event consumers that are listening. Any event consumer can express interest in an event or class of events, before there are producers that are creating those events. Cross-service interoperability is provided by adhering to the CloudEvents specification that is developed by the Cloud Native Computing Foundation Serverless working Group.


In the Emergency Response Demo application, both Knative Serving and Knative Eventing are installed by default.
The list of Knative enabled services can be seen as per the following:

`````
$ OCP_USERNAME=user1  # CHANGE ME IF NEEDED

$ oc get kservice -n $OCP_USERNAME-er-demo

NAME            URL                                                    LATESTCREATED         LATESTREADY          READY   REASON
datawarehouse   http://datawarehouse-user1-er-demo.apps.ratwater.xyz   datawarehouse-8sfmk                        False   RevisionMissing
find-service    http://find-service-user1-er-demo.apps.ratwater.xyz    find-service-xff7s    find-service-xff7s   True    
kafdrop         http://kafdrop-user1-er-demo.apps.ratwater.xyz         kafdrop-wn479         kafdrop-wn479        True
`````

Details pertaining to each type of Knative capability are found in the following sections:

# 2. Knative Serving:


1. Verify the successful installation of the knative operator in the *openshift-operators* namespace:
   
    `````
    $oc get subscription serverless-operator -n openshift-operators 

    NAMESPACE              NAME                          PACKAGE                       SOURCE                CHANNEL
    openshift-operators    serverless-operator           serverless-operator           redhat-operators      4.4
    `````

2. Verify the successful installation of Knative Serving:

    `````
    $ oc get knativeserving.operator.knative.dev/knative-serving \
         -n knative-serving \
         --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'

    DependenciesInstalled=True
    DeploymentsAvailable=True
    InstallSucceeded=True
    Ready=True
    `````

3. View *scale-out* capability of knative enabled *kafdrop* service in ER-Demo
   
   3.1 Tail container orchestration *events* in the er-demo namespace
       `````
       $oc get events -w -n $OCP_USERNAME-er-demo
       `````
    Keep this terminal window open.


   3.2 Navigate a browser tab to the output of the following command:
      ````
      echo -en "\n\n$(oc get kservice kafdrop --template='{{ .status.url }}' -n $OCP_USERNAME-er-demo)\n\n"
      ````
   3.3 Notice container orchestration events such as the following as a *kafkdrop* service is spun up from zero:
      `````
      0s          Normal    ScalingReplicaSet   deployment/kafdrop-wn479-deployment                 Scaled up replica set kafdrop-wn479-deployment-687449d4b6 to 1
      0s          Normal    SuccessfulCreate    replicaset/kafdrop-wn479-deployment-687449d4b6      Created pod: kafdrop-wn479-deployment-687449d4b6-p7btj
      0s          Normal    Scheduled           pod/kafdrop-wn479-deployment-687449d4b6-p7btj       Successfully assigned user1-er-demo/kafdrop-wn479-deployment-687449d4b6-p7btj to worker-1.ratwater.xyz
      0s          Normal    AddedInterface      pod/kafdrop-wn479-deployment-687449d4b6-p7btj       Add eth0 [10.128.2.125/23]
      0s          Normal    Pulled              pod/kafdrop-wn479-deployment-687449d4b6-p7btj       Container image "index.docker.io/obsidiandynamics/kafdrop@sha256:b7ba8577ce395b1975b0ed98bb53cb6b13e7d32d5442188da1ce41c0838d1ce9" already present on machine
      0s          Normal    Created             pod/kafdrop-wn479-deployment-687449d4b6-p7btj       Created container user-container
      0s          Normal    Started             pod/kafdrop-wn479-deployment-687449d4b6-p7btj       Started container user-container
      0s          Normal    Pulled              pod/kafdrop-wn479-deployment-687449d4b6-p7btj       Container image "registry.redhat.io/openshift-serverless-1/serving-queue-rhel8@sha256:8d45d901a58ec5da58e97b5e0d38ce22a9318837a13926ef6c21bb5b81f0bcb0" already present on machine
      `````


   
# 3. Knative Eventing:

1. Verify the successful installation of Knative Eventing:
   `````
   $ oc get knativeeventing.operator.knative.dev/knative-eventing \
        -n knative-eventing \
        --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'
  

   InstallSucceeded=True
   Ready=True

   `````

2. Verify the various types of Knative eventing sources: 
   `````
   $ oc api-resources --api-group='sources.knative.dev'

   NAME               APIGROUP              NAMESPACED   KIND
   apiserversources   sources.knative.dev   true         ApiServerSource
   kafkasources       sources.knative.dev   true         KafkaSource
   pingsources        sources.knative.dev   true         PingSource
   sinkbindings       sources.knative.dev   true         SinkBinding
   `````

3. Verify the *datawarehouse-source* (used by the Knative Eventing enabled *datawarehouse* service):
   `````
   $ oc get kafkasources -n $OCP_USERNAME-er-demo

   NAME                   TOPICS                     BOOTSTRAPSERVERS                                         READY   REASON     AGE
   datawarehouse-source   [topic-incident-command]   [kafka-cluster-kafka-bootstrap.user1-er-demo.svc:9092]   False   NotFound   4d3h
   `````



# 4. Appendix
## 4.1. Reference
1. [Knative Tutorial](https://redhat-developer-demos.github.io/knative-tutorial/knative-tutorial-eventing/index.html)
2. [Advanced Cloud Native Development:  07_1_Serverless_Lab](https://github.com/redhat-gpe/cnd_advanced_v2/blob/master/modules/07_Serverless/07_1_Serverless_Lab.adoc)
