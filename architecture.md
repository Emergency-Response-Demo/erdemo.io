---
layout: page
title: Architecture
permalink: /architecture/
---

In this application, you’ll notice multiple runtimes and frameworks in play, starting from Node, Vert.x to Kafka, prometheus and much more..  
The idea is to showcase how a polyglot system can be architected. Obviously this is not the only way to create a distributed and hybrid microservices architecture. However it should provide a true hybrid cloud solution, with IT infrastructure that provides consistency, safety, repeatability, reusability, and portability while still helping development teams move fast. The application is based on true polyglot development and it is agile, secure and scalable at every level in the stack.

Below is a diagram of the application architecture.

![application architecture](/images/application-architecture.png)

# Application Components

The following application components work together to provide the core
implementation for the demo solution.

## Mission Service

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

## Process Service

  - Runtime: Spring Boot

  - Middleware Components:
    
      - Process Automation Manager (PAM)
    
      - Decision Manager (DM)

  - Other Components: Postgres DB

The Process Service is responsible for managing the overall process flow
of the system. The Process Service operates purely on Kafka messages and
does not expose any HTTP API - although it does invoke HTTP APIs in the
Responder and Incident Priority Services.

When a new Incident is reported on the topic-incident-event Topic, the
process Service kicks off a new BPM process to manage the new Incident.
When a Responder is shown as available (via the topic-responder-event
Topic), the BPM process is updated to reflect this. As the Mission
progresses and additional messages are received on the
topic-mission-event Topic, the BPM process is updated to reflect the
latest state.

The Process Service sends out multiple types of messages on various
Topics in response to the Incident progressing through the Business
Process.

  - Send: topic-mission-command, topic-responder-command,
    topic-incident-command, topic-incident-event

  - Receive: topic-incident-event, topic-responder-event,
    topic-mission-event

## Incident Service

  - Runtime: Spring Boot

  - Middleware Products / Components: AMQ-Streams

  - Other Components: Postgres DB

The Incident Service exposes an API for registering new Incidents and
retrieving information about existing Incidents. An endpoint is also
exposed for resetting Incident state (this is typically used by
simulator services for managing and resetting the demo).

When a new Incident is received, the Incident details are stored in the
database and a new message is sent out on the topic-incident-event Kafka
Topic.

The Service also listens on Kafka to the topic topic-incident-command
for updates to Incidents and stores the latest Incident state in the
Database.

  - Send: topic-incident-event

  - Listen: topic-incident-command

## Responder Service

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

## Disaster Service

  - Runtime: Vert.x
  
  - Middleware Components: JDG

The Disaster Service exposes an API for managing the disaster's metadata, including: the coordinates and magnification for the center of the disaster; the list of inclusion zones (geopolygons in which incidents and responders should spawn); and the list of shelters and their locations.

Tracking this data dynamically allows the incident commander to change the location of the disaster to match the geography in which the demo is being performed.

## Emergency Web Console

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

# Demo Simulators

The following components are used to control the demo and simulate
events which are needed for the demo, but which can not be sourced from
/ represented in the real world (i.e. Incidents, Responder Bots,
Responder movement around the map).

## Responder Simulator

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

## Disaster Simulator

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
