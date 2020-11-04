- [1. Overview](#1-overview)
  - [1.1. Audience](#11-audience)
  - [1.2. Goal](#12-goal)
- [2. Pre-reqs](#2-pre-reqs)
  - [2.1. Skills](#21-skills)
  - [2.2. Tools](#22-tools)
- [3. Code](#3-code)
  - [3.1. Access](#31-access)
  - [3.2. IDE Tabs](#32-ide-tabs)
- [4. Review Questions](#4-review-questions)

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
6. Foundation knowledge of Microprofile specification along with SmallRye implementation
   
## 2.2. Tools
1. IDE

# 3. Code

## 3.1. Access

`````
https://github.com/Emergency-Response-Demo/responder-simulator-quarkus
`````

## 3.2. IDE Tabs
1. **src/main/resource/application.properties**
2. **etc/application.properties**
3. **InfinispanKeyValueStore.java**
4. **TopologyProducer.java**
5. **ResponderService.java**
6. **Simulator.java**

# 4. Review Questions
1. What is the purpose of the delay ?
