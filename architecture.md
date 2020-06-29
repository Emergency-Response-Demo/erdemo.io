- [1. Incident Service](#1-incident-service)
- [2. Process Service](#2-process-service)
  - [2.1. ER-Demo Business Process](#21-er-demo-business-process)
  - [2.2. Responsibilities](#22-responsibilities)
  - [2.3. Interaction with other components](#23-interaction-with-other-components)
  - [2.4. Assign Mission rules](#24-assign-mission-rules)
  - [2.5. Architecture](#25-architecture)
- [3. Incident Priority Service](#3-incident-priority-service)
- [4. Responder Service](#4-responder-service)
- [5. Disaster Service](#5-disaster-service)
- [6. Mission Service](#6-mission-service)
- [7. Emergency Web Console](#7-emergency-web-console)
- [8. Demo Simulators](#8-demo-simulators)
  - [8.1. Responder Simulator](#81-responder-simulator)
  - [8.2. Disaster Simulator](#82-disaster-simulator)

The Emergency Response Demo application consists of multiple runtimes and frameworks: Quarkus, Node, Vert.x, JBoss Data Grid, AMQ Streams (Kafka on OpenShift), Prometheus and much more.

Below is a diagram of its application architecture:

![application architecture](/images/application-architecture.png)


The following provides details of each of the core application components of the demo application.

# 1. Incident Service

  - Runtime: Spring Boot

  - Application Services Products / Components: 
    - Red Hat AMQ-Streams

  - Other Components: Postgres DB
  - Source code: https://github.com/Emergency-Response-Demo/incident-service

The Incident Service exposes an API for registering new Incidents and
retrieving information about existing Incidents. An endpoint is also
exposed for resetting Incident state (this is typically used by
simulator services for managing and resetting the demo).

When a new Incident is received, the Incident details are stored in the PostgreSQL
database and a new message is sent out on the *topic-incident-event* Kafka
Topic.

The Service also listens on Kafka to the *topic-incident-command* topic
for updates to Incidents and stores the latest Incident state in its PostgreSQL database.

  - Send: topic-incident-event

  - Listen: topic-incident-command

To view the database schema that supports the *incident-service*, execute the following from the command line:

`````
$ ERDEMO_NS=user1-er-demo    # CHANGE ME (if needed)

$ oc rsh `oc get pod -n $ERDEMO_NS | grep "^postgresql" | grep "Running" | awk '{print $1}'`

$ psql emergency_response_demo

emergency_response_demo=# \d reported_incident
`````



# 2. Process Service

  - Runtime: Spring Boot

  - Application Services Products:
    
      - Red Hat Process Automation Manager (PAM)
      - Red Hat AMQ Streams
    
  - Other Components:
    - *process-service-postgresql*
    - *incident-process-kjar*
    - *cajun-navy-rules*
  - Source code:
    - process-service:  https://github.com/Emergency-Response-Demo/process-service
    - incident-process-kjar: https://github.com/Emergency-Response-Demo/incident-process-kjar
    - cajun-navy-rules: https://github.com/Emergency-Response-Demo/cajun-navy-rules

## 2.1. ER-Demo Business Process
The Process Service manages the overall business process flow of the Emergency Response Demo scenario.  

![application architecture](/images/incident-process.png)

The business process is implemented using the *Business Process Management Notation* [(BPMN2) standard](https://www.omg.org/bpmn/).  BPMN2 is similar to other business process modelling technologies such as Visio.  However, unlike Visio, BPMN2 is a standard that is implemented as XML that can subsequently be parsed, validated, compiled and executed at runtime.

The business process of the Emergency Response demo is best characterized as *Straight-Through*.
It meets the criteria of a *straight-through* business process in that its flow is known upfront at design-time and at runtime its lifecycle starts and completes in an automated manner.  More elaboration of the different types of business processes can be found [here](/Business_Patterns.md). 

This process diagram is version controlled in a project called: *incident-process-kjar* .
It can be viewed in any number of BPMN2 editors such as: *Business Central* (from Red Hat Process Automation Manager), *[bpmn.new](https://bpmn.new/)*, or using the *[BPMN Editor](https://github.com/kiegroup/kogito-tooling/releases)* plugin for Microsoft vscode.
The *incident-process-kjar* project is compiled and loaded as a dependency into the *process-service* project.

## 2.2. Responsibilities 
The process-service is responsible for the following:

* **Microservice Orchestration**
It orchestrates the interactions of the other services of the Emergency Response demo application to complete the end-to-end business scenario: simulation of volunteer responders evacuating communities members to shelter after a hurricane.

* **Management of wait-states**
The end-to-end lifecycle of the business scenario is *long-running*.  It can take seconds, minutes and even hours for some of the steps of the business scenario to complete.  When each of these business process steps are being executed, the business process instance is temporarily placed in a *wait-state*.  In particular, its state is persisted in the corresponding *process-service* PostgreSQL database.  When the step in the business process completes, a signal is sent to the process service to resume the process instance and continue to the next task in the business process.

## 2.3. Interaction with other components
The Process Service primarily consumes and produces Red Hat AMQ Streams messages.
When a new Incident is reported on the *topic-incident-event* topic, the process Service kicks off a new BPM process to manage the new Incident.  When a Responder is shown as available (via the *topic-responder-event* topic), the BPM process is updated to reflect this. As the Mission progresses and additional messages are received on the *topic-mission-event* topic, the BPM process is updated to reflect the latest state.

The Process Service sends out multiple types of messages on various Topics in response to the incident progressing through its lifecycle:

  - **Send**: topic-mission-command, topic-responder-command, topic-incident-command, topic-incident-event

  - **Receive**: topic-incident-event, topic-responder-event, topic-mission-event


## 2.4. Assign Mission rules
One of the *service tasks* in the *incident-process* BPMN2 process definition is called:  *Assign Mission*.  This service task is of type: *BusinessRuleTask*

![Assign Mission](_site/images/../../images/assign_mission_task.png).

When the process instance reaches this service task, it invokes the *technical rules* that are version controlled in the *cajun-navy-rules* project.  These *technical rules* are authored in the *Drools Rule Language* and loaded into the process-service.

## 2.5. Architecture
The *process engine* of Red Hat Process Automation Manager can be deployed and invoked in a variety of manners.
For the purpose of the Emergency Response Demo application, the *process engine* is embedded directly in a SpringBoot microservice application.  The *process-engine* is invoked asynchronously by sending it messages on various AMQ Streams topics.

The AMQ Streams message producers are implemented as custom *work-item-handlers*.
A *workItem* extends the capabilities of the process engine.

Incidentally, this architecture approach most closely resembles the architecture approach implemented in Red Hat's upcoming *cloud-native business automation* project:  [kogito](https://kogito.kie.org/).  With Kogito, the process engine will likely run in Quarkus (instead of SpringBoot) and many of the work-item-handlers needed to inter-operate with AMQ Streams will be auto-generated.

# 3. Incident Priority Service

  - Runtime: Vert.x

  - Middleware Products / Components:
    - Red Hat Decision Manager
    - AMQ Streams

  - Source Code: https://github.com/Emergency-Response-Demo/incident-priority-service

When the process service is unable to assign a responder to an incident, an IncidentAssignmentEvent is sent to an AMQ Streams broker.
The incident priority service consumes these events and raises the priority for each failed assignment.
Uses an embedded rules engine to calculate the priority of an incident and the average priority.
The rules engine uses a stateful rules session.

# 4. Responder Service

  - Runtime: Spring Boot

  - Middleware Products / Components: AMQ-Streams

  - Other Components: Postgres DB

The Responder Service exposes an API for managing Responders, including
registering new Responders, retrieving information about all available
responders and retrieving information about specific responders.
Endpoints are also exposed for removing responders and resetting
responder state (these are typically used by simulator services for
managing and resetting the demo).

When a new Responder is registered, the Responder details are stored in
the database.

The Service also listens on Kafka to the topic test-topic for updates to
Responders and stores the latest Responder state in the Database. If the
update to the responder includes an Incident Id (i.e. if the responder
has been assigned to work on an Incident) the services also sends a new
Kafka message to the topic test-topic.

  - Send: test-topic

  - Listen: test-topic

# 5. Disaster Service

  - Runtime: Vert.x
  
  - Middleware Components: JDG

The Disaster Service exposes an API for managing the disaster's metadata, including: the coordinates and magnification for the center of the disaster; the list of inclusion zones (geopolygons in which incidents and responders should spawn); and the list of shelters and their locations.

Tracking this data dynamically allows the incident commander to change the location of the disaster to match the geography in which the demo is being performed.

# 6. Mission Service

  - Runtime: Vert.x

  - Middleware Products / Components: JDG, AMQ-Streams

  - Other Components: None

The Mission Service exposes an API for managing Missions, including
getting a list of mission keys, getting a specific mission by key,
clearing all missions and getting missions assigned to a specific
responder.

The Mission Service listens on Kafka to the topic-mission-command topic
for details of new or updated missions being created. New Mission
messages trigger a call to MapBox to generate the routes for a mission,
using the responders location as a starting point, the victims location
as a way point and the shelter location as the final destination. The
mission details are then stored in JDG, to service API requests for
Mission details.

The Mission Service sends updates to Kafka on the topic-mission-event
topic in response to mission state change events such as when a mission
is created, when an API request is received (e.g. to complete all
missions). The Mission service also sends updates to Kafka on the
topic-responder-command when missions are completed to indicate that the
Responder is available for a new mission.

  - Send: topic-mission-event, topic-responder-command

  - Receive: topic-mission-command, topic-responder-location-update

# 7. Emergency Web Console

  - Runtime: Node.js, Angular

  - Middleware Products / Components: None

The emergency console is the front end UI for the Demo Solution. It
provides the following main views:

  - Incident Commander Dashboard: The overall view of all Incidents,
    Responders and Missions

  - Responder Interface: The view for an individual responder which
    shows their current mission, including the router to the Incident
    and onward route to the shelter

  - Incidents: A tabular list of all incidents

The console communicates with several of the back end services
(Incident, Mission & Responder) to display real time data via
WebSockets.

# 8. Demo Simulators

The following components are used to control the demo and simulate
events which are needed for the demo, but which can not be sourced from
/ represented in the real world (i.e. Incidents, Responder Bots,
Responder movement around the map).

## 8.1. Responder Simulator

  - Runtime: Vert.x

  - Middleware Components: None

  - Other Components: None

The Responder Simulator is responsible for moving responders (both bots
and humans) around the map during missions. As the demo requires the
movement of personnel to function and since we can not have real people
actually moving many miles for each Mission, this simulator is required
to allow the demo to function.

The Responder simulator listens on the topic-mission-event for details
of active responders that need to be moved on the map. The simulator
them periodically updates (default every 10 seconds) the responders
location (based on the mission route received) to show the responder at
the next location. As the simulator moves responders, it emits messages
on the topic-responder-location-update Topic.

  - Send: topic-mission-event

  - Receive: topic-responder-location-update

## 8.2. Disaster Simulator

  - Runtime: Vert.x

  - Middleware Components: None

  - Other Components: None

The Disaster Simulator is used for managing / coordinating the demo. It
exposes a basic UI which allows a user to add and remove Incidents and
Responders in order to drive the demo forward.

![create incidents](/images/create-incidents.png)

The Disaster Simulator uses HTTP API requests to the Incident Service,
the Responder Service the Mission Service and the Incident Priority
Service in order to manage data creation / deletion.
