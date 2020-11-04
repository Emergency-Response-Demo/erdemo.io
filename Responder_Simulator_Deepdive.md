- [1. Overview](#1-overview)
  - [1.1. Audience](#11-audience)
  - [1.2. Goal](#12-goal)
- [2. Pre-reqs](#2-pre-reqs)
  - [2.1. Skills](#21-skills)
  - [2.2. Tools](#22-tools)
  - [2.3. Code](#23-code)
- [3. Review environment properties](#3-review-environment-properties)
  - [3.1. Kafka Channels](#31-kafka-channels)
  - [3.2. Data Grid Clients](#32-data-grid-clients)
  - [3.3. Kafka Streams Client](#33-kafka-streams-client)
  - [3.4. Responder Simulator](#34-responder-simulator)
- [4. Review Java source files](#4-review-java-source-files)
  - [4.1. InfinispanKeyValueStore.java](#41-infinispankeyvaluestorejava)
  - [4.2. Review ResponderLocationRepository.java](#42-review-responderlocationrepositoryjava)
  - [4.3. TopologyProducer.java](#43-topologyproducerjava)
  - [4.4. ResponderService.java](#44-responderservicejava)
  - [4.5. Simulator.java](#45-simulatorjava)
- [5. Test Your Understanding](#5-test-your-understanding)

**Responder Simulator Deep Dive**

# 1. Overview
The Emergency Response Demo application requires the movement of *responders*.  For the purpose of the demo, these responders along with their movement (to the pick-up point and to the evacuation center) are simulated.  The ER-Demo's [responder-simulator](https://github.com/Emergency-Response-Demo/responder-simulator-quarkus) is responsible for moving responders around the map during missions.

## 1.1. Audience
- Business Application developers and architects

## 1.2. Goal
The purpose of the document is to provide a technical deep-dive into the *Responder Simulator* of the Emergency Response Demo application.  Via review of this single project, you will gain an *advanced* understanding of how various components of the Red Hat Application Services portfolio interact to provide meaningful, reactive and scalability business services in the cloud.

# 2. Pre-reqs
## 2.1. Skills
1. Familiarity with [terminology](gettingstarted.md#13-scenario--terminology) used in the Red Hat's Emergency Response Demo application
2. Foundational knowledge of Red Hat Data Grid
3. Foundational knowledge of Red Hat AMQ Streams / Apache Kafka
4. Foundational knowledge of Apache Kafka Streams
5. Foundational knowledge of Red Hat Build of Quarkus
6. Foundational knowledge of the Microprofile specification along with SmallRye implementation
   
## 2.2. Tools
You will need an Java IDE.
Red Hat recommends one of the following:

   - VSCode with the following plugins:
     - Language Support for Java by Red Hat
     - Quarkus
     - Tools for MicroProfile
   - Red Hat CodeReady Studio (based on Eclipse)
   - Red Hat CodeReady Workspaces (based on Eclipse Che)

## 2.3. Code

The source code that will be in review is found at the following URL:

`````
https://github.com/Emergency-Response-Demo/responder-simulator-quarkus
`````

Clone this project and load it into your favorite IDE for Java development.

NOTE:  Red Hat recommends the following IDEs:
  - 
   
# 3. Review environment properties
At runtime, environment variables set in the *responder-simulator* container will be injected from two properties files that complement each other.

In your IDE, open the following two files:

- *src/main/resources/application.properties*
  This file is parsed at build-time by the Quarkus compiler.  These properties (along with any additional are included in the *responder-simulator* container.
- *etc/application.properties* files.
  This file is not parsed at build-time.  Instead, the properties in this file reflect equivalent properties that are loaded into a config map (when ER-Demo is provisioned) and are read at boot time by Quarkus.

## 3.1. Kafka Channels
1. Notice that the first set of properties define *incoming* and *outgoing* Microprofile *channels* for communication with Red Hat AMQ Streams

  - src/main/resources/application.properties
    `````
    # Configure the Kafka sources
    mp.messaging.incoming.mission-event.connector=smallrye-kafka
    mp.messaging.incoming.mission-event.key.deserializer=org.apache.kafka.common.serialization.StringDeserializer
    mp.messaging.incoming.mission-event.value.deserializer=org.apache.kafka.common.serialization.StringDeserializer
    mp.messaging.incoming.mission-event.request.timeout.ms=30000
    mp.messaging.incoming.mission-event.enable.auto.commit=false

    mp.messaging.outgoing.responder-location-update.connector=smallrye-kafka
    mp.messaging.outgoing.responder-location-update.key.serializer=org.apache.kafka.common.serialization.StringSerializer
    mp.messaging.outgoing.responder-location-update.value.serializer=org.apache.kafka.common.serialization.StringSerializer
    mp.messaging.outgoing.responder-location-update.session.timeout.ms=6000
    mp.messaging.outgoing.responder-location-update.acks=1
    `````
  - etc/application.properties
    `````
    mp.messaging.incoming.mission-event.topic=topic-mission-event
    mp.messaging.incoming.mission-event.group.id=responder-simulator-quarkus

    mp.messaging.outgoing.responder-location-update.topic=topic-responder-location-update
    `````

2. The responder-simulator consumes *MissionStartedEvent* messages from the *topic-mission-event* topic.  Subsequently, as the mission progresses, it produces a JSON message to the *topic-responder-location-update* topic that is similar to the following:
   `````
   {
   "responderId": "21",
   "missionId": "a6af2762-2ffc-4484-94bd-8ad2111d721c",
   "incidentId": "96287834-811f-4108-98bf-cc4ed21881f5",
   "status": "MOVING",
   "lat": 34.0187,
   "lon": -77.8986,
   "human": false,
   "continue": true
   }
   `````

## 3.2. Data Grid Clients

1. In support of the *responder-simulator* service, Data Grid is used as an in-memory grid for the following:
   - responder state
   - responder-simulator state
  
    Follow-on sections of this document will elaborate.

2. As part of the Emergency Response Demo provisioning process, Red Hat Data Grid is installed.  The responder-simulator is a client of Red Hat Data Grid.
   
3. Make note of the following DataGrid client related properties:
   - etc/application.properties
     `````
     quarkus.infinispan-client.server-list={{ datagrid_server_list }}
     quarkus.infinispan-client.auth-username={{ datagrid_username }}
     quarkus.infinispan-client.auth-password={{ datagrid_password }}

     # Name of Data Grid cache for responder state
     infinispan.streams.store=responder-store

     # Name of Data Grid cache for responder-simulator state
     infinispan.cache.responder-simulator=responder-simulator
     `````

## 3.3. Kafka Streams Client

1. In the responder-simulator service, Kafka Streams captures responder related state and exposes this data to assist in the generation of simulated responder *moves*.  Follow-on sections of this document will elaborate.
2. As part of the Emergency Response Demo provisioning process, the Kafka Streams component of Red Hat AMQ Streams is installed .  The responder-simulator is a client of this Kafka Streams component.
3. Make note of the following Kafka Streams related properties:
   - etc/application.properties
     `````
     quarkus.kafka-streams.bootstrap-servers={{ kafka_bootstrap_address }}
     quarkus.kafka-streams.application-id=responder-simulator-kstreams
     quarkus.kafka-streams.topics=topic-responder-event

     kafka.topic.responder-event=topic-responder-event
     `````

## 3.4. Responder Simulator

`````
simulator.delay=10000
simulator.distance.base=1500.0
simulator.distance.variation=0.2
`````

# 4. Review Java source files

## 4.1. InfinispanKeyValueStore.java

The purpose of this ApplicationScope object is to make Responder state available to the Responder Simulator.
The Responder state is stored in Red Hat Data Grid.
This class manages the lifecycle of that Responder data in Data Grid.

This Java class implements *org.apache.kafka.streams.state.KeyValueStore*.
It is invoked by the *TopologyProducer* object when responder state change events are consumed and an Apache Kafka Streams *ktable* is created from these events.

## 4.2. Review ResponderLocationRepository.java

The purpose of this ApplicationScope object is to make Responder Location state available to the Responder Simulator.
Responder Location state is stored in Red Hat Data Grid.
This class manages the lifecycle of this Responder Location state in Data Grid.


## 4.3. TopologyProducer.java
## 4.4. ResponderService.java
## 4.5. Simulator.java

This ApplicationScope Java object tranforms MissionCreated events into a ResponderLocation object that subsequently is persisted in Red Hat Data Grid via the ResponderLocationRepository object.

# 5. Test Your Understanding
1. What is the status of the *outgoing* message ?
