---
layout: page
title: Getting Started
permalink: /gettingstarted/
---

Asssuming you have the demo installed, if not, dont worry, you can get all the details on installation from the install page. Once installed, follow the instructions below to do a test run of the environment.


1.  Once logged in, select the project **emergency-response-demo**

2.  In the list of applications, click the link for
    **emergency-console**
    
    ![launch emergency console](/images/launch-emergency-console.png)

3.  This link takes you to the login screen for the Emergency Web
    Console
    
    ![emergency console login
    screen](/images/emergency-console-login-screen.png)

4.  Click the **Register Now\!** button. This is for registering as a
    volunteer responder.

5.  Enter the relevant information for your account. You can select any
    test data.
    
    1.  Make note of the fields for Boat Capacity and Medical Support.
    
    2.  You can specify how many people you can carry in your boat.
        Also, you can indicate if you provide medical / first-aid
        support.
        
        ![register now responder](/images/register-now-responder.png)

6.  Click **Register**

# Explore the Application

Once you are successfully registered, you will see the main application
screen.

![main application screen](/images/main-application-screen.png)

  - This screen has the following links:
    
      - **Dashboard**: The overall view of all Incidents, Responders and
        Missions
    
      - **Mission**: The view for an individual responder which shows
        their current mission, including the router to the Incident and
        onward route to the shelter
    
      - **Incidents**: A list of all incidents
    
      - **Github**: Link to the Github repo

## Dashboard

1.  Select the link for **Dashboard**
    
      - This screen shows an overall view of all Incidents, Responders
        and Missions. At the moment the screen is empty, but in the
        following sections we will add data to the application.
        
        ![dashboard empty](/images/dashboard-empty.png)

### Incident Status

The Incident Status section tracks the data for number of incidents
requested, assigned, picked up and rescued. These values update in
real-time based on application events.

### Responder Utilization

The Responder Utilization section monitors the total number of
responders, active and idle responders. This section is also updated in
real-time based on application events.

### Map

The map shows the location of the incidents, responders and their
associated routes.

## Mission

When a Responder is assigned an Incident, a Mission is created. The
Mission defines where the Responder needs to go to collect the victims
of the Incident (the Way Point) and what shelter the victims should be
dropped off at (the Target Location). The mission also has details of
the responders location history.

1.  Select the link for **Dashboard**
    
      - This screen shows the view for an individual responder which
        shows their current mission, including the router to the
        Incident and onward route to the shelter.

2.  Add yourself to the map as a responder
    
    1.  Click any location on the map.
        
        ![add as responder to the
        map](/images/add-as-responder-to-the-map.png)
    
    2.  Click your boat icon. It will show the details of your boat
        profile.
        
        ![responder boat details](/images/responder-boat-details.png)
    
    3.  Click the **Available** button.
        
          - You are now registered as an **Available** responder.

3.  Click the **Dashboard** link
    
      - In the **Responder Utilization** section, verify that there are
        1 total responders. This is based on your recent action.
        
        ![1 total responders](/images/1-total-responders.png)

## Incidents

An incident is a request for help from an individual (or group of
individuals) that are in need of rescue. Details of an Incident include
the location (Lat, Long), the number of people stranded and whether
medical assistance is required.

1.  Click the **Incidents** link
    
      - This screen shows a list of incidents. At the moment, this
        screen is empty, but we will create incidents in the next
        section.
        
        ![incidents empty](/images/incidents-empty.png)

# Disaster Simulator

1.  Move back to the **OpenShift Web Console** window

2.  In the list of applications, click the link for
    **disaster-simulator**
    
    ![launch disaster simulator](/images/launch-disaster-simulator.png)

3.  This takes you to the Disaster Simulator web console.

## Create Incidents

1.  In the section for **Create Incidents**, move to the field for
    **Number of Incidents** and enter `50`.

2.  Click **Submit**
    
    ![create incidents](/images/create-incidents.png)

## Create Responders

1.  In the section for **Create Responders**, move to the field for
    **Number of Responders** and enter `3`.

2.  Click **Submit**
    
    ![create responders](/images/create-responders.png)

3.  Move back to the **Emergency Response Demo Web Console** window

4.  Click the **Dashboard** link.

5.  Confirm that you have incidents and responders.
    
      - You will see activity as the responders are assigned to
        missions. The responders will start moving to rescue the
        stranded victims.
        
        ![er main dashboard](/images/er-main-dashboard.png)

# View Your Mission

By this time, your boat should have been assigned to a mission.

1.  Click the **Mission** link.
    
      - You will see your boat moving towards an incident.
    
      - Once your boat makes it to the incident location, click the
        **Picked Up** button.
    
      - This confirms that you have picked up the passengers and your
        boat will proceed to the shelter.
        
        ![mission picked up](/images/mission-picked-up.png)

# View Incidents

You can view a list of all incidents and check their status.

1.  Click the **Incidents** link.
    
    ![view all incidents](/images/view-all-incidents.png)

# Process Automation

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

Let’s view the process diagram for an incident.

1.  Click the **Dashboard** link.

2.  Click an Incident on the map.
    
      - This will show a pop-up for the incident.
        
        ![incident popup](/images/incident-popup.png)

3.  Click the link for **Process Diagram**
    
      - This will open new tab to view the Process Diagram for this
        incident.
        
        ![view process diagram](/images/view-process-diagram.png)

4.  Review the process diagram for this incident.

# Clean Up

Let’s clean up our application by clearing the incidents, responders and
missions.

1.  Move back to the **Disaster Simulator** web console.

2.  Click the buttons to clear application data
    
    1.  Click **Clear incidents**
    
    2.  Click **Clear responders**
    
    3.  Click **Clear missions**
        
        ![clear incidents responders
        missions](/images/clear-incidents-responders-missions.png)

3.  Move back to the **Emergency Response Demo Web Console** window

4.  Click the **Dashboard** link.

5.  Confirm that all of application data is cleared.

Congratulations\! You have completed this lab. Please move on to the
next lab.
