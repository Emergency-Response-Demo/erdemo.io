- [1. Overview](#1-overview)
- [2. Pre-requisites](#2-pre-requisites)
  - [2.1. Local Tooling](#21-local-tooling)
  - [2.2. MapBox Access Token](#22-mapbox-access-token)
  - [2.3. OpenShift](#23-openshift)
    - [2.3.1. Overview](#231-overview)
    - [2.3.2. Minimum Requirements](#232-minimum-requirements)
- [3. Installation](#3-installation)
  - [3.1. Setup](#31-setup)
  - [3.2. Option A:  Pre-built Images](#32-option-a-pre-built-images)
  - [3.3. Option B: CI/CD](#33-option-b-cicd)
  - [3.4. Installation Complete !](#34-installation-complete-)
    - [3.4.1. Sanity Checks](#341-sanity-checks)
    - [3.4.2. Deployment Topology](#342-deployment-topology)
  - [3.5. Troubleshooting A Failed Deployment](#35-troubleshooting-a-failed-deployment)
    - [3.5.1. Ansible Hints](#351-ansible-hints)
    - [3.5.2. Idempotency](#352-idempotency)
  - [3.6. Uninstalling](#36-uninstalling)
- [4. ER-Demo Web Consoles](#4-er-demo-web-consoles)
  - [4.1. **Emergency Response Console**](#41-emergency-response-console)
  - [4.2. **Disaster Simulator**](#42-disaster-simulator)
  - [4.3. **Grafana Dashboards**](#43-grafana-dashboards)
- [5. Appendix](#5-appendix)
  - [5.1. OCP4 from RHPDS](#51-ocp4-from-rhpds)
    - [5.1.1. Overview](#511-overview)
    - [5.1.2. Risks and Challenges](#512-risks-and-challenges)
    - [5.1.3. Order OCP4](#513-order-ocp4)
    - [5.1.4. Confirmation Emails](#514-confirmation-emails)
    - [5.1.5. Access](#515-access)
  - [5.2. Advanced Multi-User Installation](#52-advanced-multi-user-installation)


# 1. Overview
By this time you are excited and want to try out this application.  To do so, you will need to install the application on an OpenShift Container Platform (OCP) 4.* environment.

The approach currently taken is:  **Bring Your Own OpenShift 4 Cluster** .

Once you've acquired your own OpenShift 4, the installation of the ER-Demo application on your OpenShift cluster is done using Ansible.

# 2. Pre-requisites

## 2.1. Local Tooling

To install the Emergency Response application, you will need the following tools on your local machine:

1. **Unix flavor OS with BASH shell**:  ie; Fedora, RHEL, CentOS, Ubuntu, OSX
2. **git**
3. **[oc utility v4.6](https://mirror.openshift.com/pub/openshift-v4/clients/oc/4.6/)**
4. **[Ansible](https://www.redhat.com/en/technologies/management/ansible)**
   
   Installation of the Emergency Response application is tested using the _ansible-playbook_ utility from the _ansible_ package of Fedora 31.  Others in the community have also succeeded in installing the app using ansible on OSX.

## 2.2. MapBox Access Token

The Emergency Response application makes use of a third-party SaaS API called [MapBox](https://www.mapbox.com/).
MapBox APIs provide the Emergency Response application with an optimized route for a responder to travel given pick-up and drop-off locations.  To invoke its APIs, MapBox requires an [access token](https://docs.mapbox.com/help/how-mapbox-works/access-tokens).  For normal use of the Emergency Response application the _free-tier_ account provides ample rate-limits.

![MapBox token](/images/mapbox_token.png)

## 2.3. OpenShift

### 2.3.1. Overview

<span style="color:blue">To host the Emergency Response demo, it is recommended that you use your own OpenShift 4 environment.</span>.

The benefit of utilizing your own OpenShift environment is that you decide if/when to shut it down and the duration of its lifetime.  In addition, if there are any errors in the provisioning process of OpenShift, you will have some ability to troubleshoot the problem.

Otherwise, if you are a Red Hat associate or Red Hat partner, you can order an OpenShift 4 environment from Red Hat's _Partner Demo System_ (RHPDS).  Using RHPDS, the minimum requirements described below are met.  However, there are known risks and challenges.  Details pertaining to these risks as well as accessing an OpenShift 4 environment from RHPDS are found in [the Appendix](##51-ocp4-from-rhpds) of this document.

### 2.3.2. Minimum Requirements
To install the Emergency Response application, you will need a full OpenShift Container Platform environment with the following minimum specs:

1. **OCP Version:**  4.6 
2. **Memory:**    24 GBi allocated to one or more _worker_ node(s)
3. **CPU:** 10 cores allocated to one or more _worker_ nodes
4. **Disk:** 20 Persistent Volumes that support [Read Write Once (RWO)](https://docs.openshift.com/container-platform/4.4/storage/understanding-persistent-storage.html#pv-access-modes_understanding-persistent-storage).
   
   The Emergency Response application does not require Read-Write-Many (RWX).
   Every PV should be at least 5GB in size.

5. **Credentials:**  You will need _cluster-admin_ credentials to your OpenShift environment.
6. **CA signed certificate:** Optional
   
   Preferably, all public routes of your Emergency Response application utilize a SSL certificate signed by a legitimate certificate authority.  ie:  [LetsEncrypt](https://letsencrypt.org/)
7. **Non cluster-admin users**:
   The Emergency Response application will be owned by a non cluster-admin _project administrator_.  In your OpenShift environment, you will need one or more non cluster-admin users to serve this purpose.  In _Code Ready Containers_, the default non cluster-admin user is:  _developer_ .  In other OpenShift clusters, the convention tends to be:  user[1-200].
8. **Pull Secret**:
   Some Linux container images used in the Emergency Response application reside in the following secured image registry:  _registry.redhat.io_.
   Those images will need to be pulled to your OpenShift 4 environment.
   As part of the installation of OCP4, you should have already been prompted to provide your [pull secret](https://cloud.redhat.com/openshift/install/pull-secret) that enables access to various secured registries to include regisry.redhat.io.
9. **OpenShift Monitoring Capabilities** 
   <span style="color:red">WARNING: Avoid the use of CodeReady Containers to host the ER-Demo.  For one, the monitoring capabilities of the demo will not be available.</span>.



# 3. Installation 
Now that you have an OpenShift environment that meets the minimum requirements, you can now layer the Emergency Response application on that OpenShift.  You will do so using an ansible playbook.

## 3.1. Setup

1. Using the oc utility on your local machine, ensure that your are authenticated in your OCP 4 environment as a cluster-admin.
2. Using the git utility on your local machine, clone the _install_ project of the Emergency Response application:
    ```
    git clone https://github.com/Emergency-Response-Demo/install.git

    ```
    This installer uses Ansible. There is a comprehensive list of roles and playbooks that install the application.

3. Change directories into the _ansible_ directory of the cloned project:
   ```
   cd install/ansible
   ```

4. Checkout the latest tag:
   ```
   git checkout 2.9
   ```

5. Copy the _inventory.template_
   ```
   cp inventories/inventory.template inventories/inventory
   ```

6. Using your favorite text editor, open the copied file: _inventories/inventory_
7. Replace the sample MapBox access token (`map_token`) with a real MapBox token.
   ```
   # MapBox API token, see https://docs.mapbox.com/help/how-mapbox-works/access-tokens/
   map_token=pk.egfdgewrthfdiamJyaWRERKLJWRIONEWRwerqeGNjamxqYjA2323czdXBrcW5mbmg0amkifQ.iBEb0APX1Vmo-2934rj
   ```
8. Save the changes.
9.  Diff your inventory file with the inventory template and ensure only the value of the _map_token_ is different:
   ```
   $ diff inventories/inventory inventories/inventory.template 
     3c3
     < map_token=pk.eyJ1IjoiamdyaWRlIiwiYSI6ImNqeGNjamxq5jAxeXczdXBrcW5mbml0amkifQ.iBEb0APX1Vmo_VtsDj-Y3g
     ---
     > map_token=replaceme
   ```
   **NOTE:** If you previously cloned the ER-Demo install ansible and the values in your inventory file are now out of date, be sure to update as per the inventory.template.



You will now execute the ansible to install the Emergency Response application.
**Select one of the following two approaches:  _Pre-built Images_ or _CI/CD_**

## 3.2. Option A:  Pre-built Images
This approach is best suited for those that want to utilize (ie: for a customer demo) the Emergency Response app.  This installation approach does not use Jenkins pipelines.  Instead, the OpenShift _deployments_ for each component of the Emergency Response application are started using pre-built Linux container images pulled from corresponding [public Quay image repositories](https://quay.io/organization/emergencyresponsedemo).  With this approach, the typical duration to build the Emergency Response app is about 20 minutes.  This is the default installation approach.

1. From the _install/ansible_ directory, kick-off the Emergency Response app provisioning:
   ```
   ansible-playbook -i inventories/inventory playbooks/install.yml
   ```
2. After about 20 minutes, you should see ansible log messages similar to the following:
   ```
   PLAY RECAP ********************************************************************************
   localhost : ok=432  changed=240  unreachable=0    failed=0    skipped=253  rescued=0    ignored=0 
   ```


## 3.3. Option B: CI/CD

This approach is best suited for code contributors to the Emergency Response app.  Individual Jenkins pipelines are provided for each component of the app.  Each pipeline builds from source, tests, creates a Linux container image and deploys that image to OpenShift.  The typical duration to build the Emergency Response application from source using these pipelines is about an hour.  This approach is also of value if the focus of a demo to a customer and/or partner is to introduce them to CI/CD best practices of a microservice architected application deployed to OpenShift.

1. From the _install/ansible_ directory, kick-off the Emergency Response app provisioning:
   ```
   ansible-playbook -i inventories/inventory playbooks/install.yml \
                    -e deploy_from=source
   ```
   
2. After about an hour, the provisioning should be complete.

3. You can review any of the CI/CD pipelines that ran as part of this provisioning process:
   1. List all build pipelines:
       ```
       oc get build -n user1-er-tools | grep pipeline

       ...

       user1-incident-service-pipeline-1         JenkinsPipeline            Complete   2 hours ago         
       user1-mission-service-pipeline-1          JenkinsPipeline            Complete   2 hours ago         
       user1-responder-service-pipeline-1        JenkinsPipeline            Complete   2 hours ago         
       user1-assignment-rules-model-pipeline-1   JenkinsPipeline            Complete   2 hours ago  
       ```
   2. From that list, pick any of the builds to get the URL to the corresponding Jenkins pipeline:
      ```
      oc logs user2-incident-service-pipeline-1 -n user1-er-tools

      info: logs available at https://jenkins-user2-er-tools.apps.cluster-denver-8ab6.denver-8ab6.example.opentlc.com/blue/organizations/jenkins/user2-er-tools%2Fuser2-er-tools-user2-mission-service-pipeline/detail/user2-er-tools-user2-mission-service-pipeline/1/
      ```
   3. Open a browser tab and navigate to the provided URL:
      ![](/images/pipeline_example.png)


## 3.4. Installation Complete !

Congratulations on having installed the ER-Demo !

### 3.4.1. Sanity Checks
A few sanity-checks that you can execute prior to getting started with the demo are as follows:

1. Ensure that the *statefulsets* that support the ER-Demo are healthy:

   ```
   $ oc get statefulset -n user1-er-demo

   NAME                      READY   AGE
   datagrid-service          3/3     6d13h
   kafka-cluster-kafka       3/3     6d14h
   kafka-cluster-zookeeper   3/3     6d14h
   ```

2. Ensure that the ER-Demo *deployments* are healthy:

   ```
   $ oc get dc -n user1-er-demo

   NAME                              DESIRED   CURRENT
   dw-postgresql                     1         1      
   postgresql                        1         1      
   process-service-postgresql        1         1      
   user8-datawarehouse               1         1      
   user8-disaster-service            1         1      
   user8-disaster-simulator          1         1      
   user8-emergency-console           1         1      
   user8-incident-priority-service   1         1      
   user8-incident-service            1         1      
   user8-mission-service             1         1      
   user8-process-service             1         1      
   user8-process-viewer              1         1      
   user8-responder-client-app        1         1      
   user8-responder-service           1         1      
   user8-responder-simulator         1         1      

   ```

3. Ensure that all of pods are stable and are not being restarted:
   ```
   $ oc get pods -w -n user1-er-demo
   ```

### 3.4.2. Deployment Topology
A complete topology of all of the components that have been installed can be found [here](/images/project_topology.png).
As you become more familiar with the ER-Demo, consider cross-referencing all the components listed in this diagram with what is actually deployed in your OpenShift cluster.

Also, the ER-Demo [Architecture Guide](/architecture.md) provides details of the various components that make up the ER-Demo.

## 3.5. Troubleshooting A Failed Deployment
Considering that the automated provisioning of ER-Demo consists of many hundreds of discrete steps, errors can occur from time to time.

The good news is that at least you have full cluster-admin access to your OpenShift cluster to troubleshoot.  This is where your OpenShift skills will be of great assistance.

### 3.5.1. Ansible Hints

Normally you can get a ball-park approximation as to where the provisioning failed by studying the ansible output. ie:

````
TASK [../roles/openshift_process_service : deploy postgresql] ****************************************************************************************************************************************************
fatal: [localhost]: FAILED! => {"changed": false, "msg": {"cmd": "oc create -f /tmp/process-service-4LN1V4/postgresql.yml -n user1-er-demo", "results": {}, "returncode": 1, "stderr": "Error from server (AlreadyExists): persistentvolumeclaims \"process-service-postgresql\" already exists\n", "stdout": "service/process-service-postgresql created\ndeploymentconfig.apps.openshift.io/process-service-postgresql created\n"}}

NO MORE HOSTS LEFT ***********************************************************************************************************************************************************************************************

PLAY RECAP *******************************************************************************************************************************************************************************************************
localhost                  : ok=16   changed=8    unreachable=0    failed=1    skipped=1    rescued=0    ignored=0  
````

The above log alludes to a problem provisioning the postgresql database for the _process-service_.
For further troubleshooting assistance, feel free to reach out to the community at:  emer-demo-team at redhat dot com .

### 3.5.2. Idempotency

Also of good news is that the ansible itself is idempotent.
In particular, it is feasible to re-run the ansible more than one time without introducing unintended consequences.

As a short-cut, you may not have to re-run the entire install playbook.
Included are playbooks for each service that make up the ER-Demo.

`````
$ ls playbooks/

assignment_rules_model.yml      emergency_console.yml          incident_service.yml  kafka_lag_exporter.yml  monitoring.yml       responder_client_app.yml
assignment_rules.yml            erd_monitoring.yml             install.yml           kafka_topics.yml        nexus.yml            responder_service.yml
datagrid.yml                    find_service.yml               jenkins.yml           knative.yml             pgadmin4.yml         responder_simulator_service.yml
datawarehouse.yml               group_vars                     kafdrop.yml           library                 postgresql.yml       sso_realm.yml
disaster_service.yml            incident_priority_service.yml  kafka_cluster.yml     mission_service.yml     process_service.yml  sso.yml
disaster_simulator_service.yml  incident_process.yml           kafka_connect.yml     module_utils            process_viewer.yml   strimzi_operator.yml

`````

For example, to re-install just the _process-service_, the following could be executed:

`````
$ ansible-playbook -i inventories/inventory playbooks/process_service.yml -e ACTION=uninstall

$ ansible-playbook -i inventories/inventory playbooks/process_service.yml
`````





## 3.6. Uninstalling

To uninstall:
```
$ ansible-playbook playbooks/install.yml \
                   -e ACTION=uninstall \
                   -e uninstall_cluster_resources=true \
                   -e uninstall_delete_project=true
```

# 4. ER-Demo Web Consoles

Now that installation of the Emergency Response app is complete, you should be able to navigate your browser to the following URLs:

## 4.1. **Emergency Response Console**
 - Navigate to the URL from the following command:

   ```
   echo -en "\nhttps://$(oc get route emergency-console -n user1-er-demo --template='{{ .spec.host }}')\n\n"
   ```
   ![](/images/erdemo_home.png)
- More information about the *Emergency Response Console* is found in the [Getting Started Guide](/gettingstarted.md).

## 4.2. **Disaster Simulator**
 - Navigate to the URL from the following command:

   ```
   echo -en "\nhttp://$(oc get route disaster-simulator -n user1-er-demo --template='{{.spec.host}}')\n\n"
   ```
   ![](/images/disaster_simulator.png)
 - More information about the *Disaster Simulator* is found in the [Getting Started Guide](/gettingstarted.md).

## 4.3. **Grafana Dashboards**
 - Navigate to the URL from the following command:

   ```
      echo -en "\nhttps://$(oc get route grafana-route -n user1-er-metrics --template='{{ .spec.host }}')\n\n"
   ```
   ![](/images/grafana_home.png)

 - Once Emergency Response *incidents* are created, you will see corresponding metrics:
   ![](/images/grafana_kpis.png)
- More information about the out-of-the-box Dashboards in Grafana for the Emergency Response application found in the [Getting Started Guide](/gettingstarted.md).


Further details regarding how to run the ER-Demo can be found in the [Getting Started Guide](/gettingstarted.md).



# 5. Appendix

## 5.1. OCP4 from RHPDS

### 5.1.1. Overview

The _Red Hat Product Demo System_ (RHPDS) provides a wide variety of cloud-based labs and demos showcasing Red Hat software.
One of the offerings from RHPDS is a cloud-based OCP 4 environment that meets all of the minimum requirements to support an installation of the Emergency Response application.  The default shutdown and lifetime durations of this environment are as follows:

* **Runtime:**  10 hours;
You have the ability to extend the runtime (one time only) and re-start if it was shutdown.

* **Lifetime:** 2 days;
You have the ability to extend the lifetime three times for 3 days each.
Subsequently, the max lifetime could be up to 11 days.

To utilize RHPDS, you will need the following:

1. [OPENTLC credentials](https://account.opentlc.com/account/).  OPENTLC credentials are available only to Red Hat associates and Red Hat partners.
2. SFDC Opportunity, Campaign ID or Partner Registration


### 5.1.2. Risks and Challenges

* **Provisioning failure rates from RHPDS can be high**.  Each provisioning attempt for an OCP 4 cluster consists of hundreds of steps, many of which rely on third-party services. You may need to attempt numerous times over the course of days.
  
* **Shutdown and Restarts**   Your OpenShift environment will shut down at known periods (typically 10 hours) and will be deleted after a certain duration (typically 2 days).  The real problems typically occur with a restart of the cluster.  ie:  AWS EBS is known to go stale from time to time with the restart of a cluster.  
  
    <span style="color:red">WARNING: ALWAYS SMOKE TEST YOUR ER-DEMO ENVIRONMENT AFTER A CLUSTER RESTART !!! </span>.


### 5.1.3. Order OCP4


1.  In a web browser, navigate to the *Red Hat Product Demo System* at:
    [Red Hat Product Demo System](https://rhpds.redhat.com/catalog/explorer).

2.  Authenticate using your *OPENTLC* credentials, for example:
    `johndoe-redhat.com`.

3.  Navigate to the following catalog: `Services → Service Catalogs → Multi-Product Demos.

4.  Select the following catalog item: `OCP Cluster for ER-Demo (self-install)`.
   
    ![Catalog_Item](/images/rhpds_catalog_item.png)

5.  Click `Order` on the next page.

6.  In the subsequent order form, add in details similar to the following:
    ![Order form](/images/rhpds_order_form.png)   

7.  At the bottom of the same page, click `Submit`.

### 5.1.4. Confirmation Emails

The provisioning of your OpenShift environment from RHPDS typically takes about 1 hour.

Upon ordering the lab environment, you will receive the following various confirmation emails:

1.  **Your RHPDS service provision request has started:**  This email should arrive within minutes of having ordered the environment.
2.  **Your RHPDS service provision request has updated:**  You will receive one or more of these emails indicating that the OCP 4 provisioning process continues to proceed.
3.  **Your RHPDS service provision request has completed:**
    
    1.  Read through this email in its entirety and save !
    2.  This email includes details regarding its deletion date.
    3.  This email also includes URLs to the OpenShift Web Console as well as the OpenShift Master API.
    4.  Also included is the userId and password of the OpenShift cluster-admin user.
        
        ![rhpds_completed_email](/images/rhpds_completed_email.png)
    


### 5.1.5. Access
1. Upon reading through the completion email, you should authenticate into the OpenShift environment as a cluster admin.  You will execute a command similar to the following:
   ```
   oc login https://api.cluster-242b.242b.example.opentlc.com:6443 -u <cluster-admin user> -p '<cluster-admin passwd>'
   ```
2. Validate the existance of all _master_ and _worker_ nodes:
   ```
   oc adm top nodes

   NAME                                              CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
   ip-10-0-138-177.ap-southeast-1.compute.internal   456m         13%    2632Mi          17%       
   ip-10-0-141-34.ap-southeast-1.compute.internal    332m         2%     3784Mi          5%        
   ip-10-0-145-167.ap-southeast-1.compute.internal   598m         17%    2847Mi          18%       
   ip-10-0-150-7.ap-southeast-1.compute.internal     341m         2%     3445Mi          5%        
   ip-10-0-167-23.ap-southeast-1.compute.internal    173m         1%     2699Mi          4%        
   ip-10-0-173-229.ap-southeast-1.compute.internal   567m         16%    3588Mi          23%
   ```

3. Verify login access using non cluster-admin user(s):
   
   1. Your OpenShift environment from RHPDS is pre-configured with 200 non cluster-admin users.  Details of these users is as follows:

       * **userId**:  user[1-200] ;   (ie:   user1, user2, user3, etc)
       * **passwd**:  see [this doc](https://docs.google.com/document/d/1s4FKXzXHLJ8Z7-ClUSGSvUbQ1Xc7AOWAwIiLb8Uffvc/edit) for details.

   2. Using the credentials of one of these users, verify you can authenticate into OpenShift:
       ```
       oc login -u user1 -p <password>

       ...

       Login successful.
       You don't have any projects. You can try to create a new project, by running
       oc new-project <projectname>

       ```
   3. Re-authenticate back as cluster-admin:
      ```
      oc login -u <cluster admin user> -p <cluster admin passwd>
      ```
   4. View the various users that have authenticated into OpenShift:
      ```
      oc get identity
      ```
Now that your OpenShift 4 environment has been provisioned from RHPDS, please return to the section above entitled: [Installation Procedure](#3-installation-procedure).


## 5.2. Advanced Multi-User Installation

The ER-Demo ansible provisioning allows for more than one ER-Demo installations per OpenShift cluster.
This becomes useful, for example, when using the ER-Demo as the basis of customer and/or partner workshops where each student would be assigned their own demo environment.

Its often the case that a fresh OpenShift cluster can accommodate more than one ER-Demo environment. 
Mileage will vary depending on the amount of available hardware resources.
For example, the _OCP4 for ER-Demo_ cluster available through RHPDS can typically run about 5 concurrent ER-Demo environments.

The ansible provisioning segregates ER-Demo environments on the same OpenShift cluster by OpenShift users.
This can be done by applying the _project_admin_ environment variable to each ansible command as follows:

1. Set an environment variable that reflects the userId of your non cluster-admin user.  ie:
   ```
   OCP_USERNAME=user2
   ```
   
2. From the _install/ansible_ directory, kick-off the Emergency Response app provisioning:
   ```
   ansible-playbook -i inventories/inventory playbooks/install.yml \
                    -e project_admin=$OCP_USERNAME
   ```

3. To uninstall:
   ```
    $ ansible-playbook playbooks/install.yml \
                   -e ACTION=uninstall \
                   -e project_admin=$OCP_USERNAME \
                   -e uninstall_cluster_resources=true \
                   -e uninstall_delete_project=true
   ```
