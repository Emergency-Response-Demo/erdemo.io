---
layout: page
title: Install
permalink: /install/
---

By this time you are excited and want to try out this demo. There are a few ways you can get hands on with this demo. 
Following is the logical breakdown for an installation. In many cases you might just need to install the demo only. However assuming you might just have an Openshift (OCP) Cluster or a Red Hat Managed Integration. We have put the instructions for the following scenario.
The installation can be broken into three steps:
- You have a running Openshift cluster
- You have RHMI or Integreatly installed on Openshift
- You are a Red Hat Partner or Employee and have access to Demo system

The last secton on this page is for the partner demo system, which Red Hat Consultants, Solution Architect and Partners use. This might not apply to everyone.

## I have my own Openshift Cluster
The demo is currently baselined to Red Hat Openshift 3.11, work is underway to port it to 4.2. 
The following instructions assume you have a running Openshift cluster i.e. a 3.11 cluster.
If you have a Red Hat Managed Integration(RHMI) or Integreatly cluster move to the next section.

```
The demo uses features like:
 Security e.g. SSO (Single Sign On), 
 Observability (Prometheus, Grafana), 
 logging and more from the Integreatly services. 
Its an important component and supporting infra-services for the demo. 
```
To install Integreatly, follow the installation instructions. [here]: https://github.com/integr8ly/installation


## I have a Red Hat Managed Integration(RHMI) or Integreatly cluster
If you do not have a running RHMI or Integreatly cluster please check the previous section on how you can install it. 

Assuming that you have a running Integreatly cluster. 
Login to the Openshift console as a cluster admin via command line

```
oc login master.example.openshiftworkshop.com -u <user> -p <password>
```

Make sure you can clone the following repo e.g. 
```
git clone https://github.com/Emergency-Response-Demo/install.git

```

This installer also uses Ansible. theres a comprehensive list or roles and playbooks that install the demo. 
Ensure that you are logged in as a Cluster admin.

Assuming you have cloned the git repo, from the command line change dir (cd) to install/ansible

The README file should list the following prerequisites:

* `oc` client on the PATH
* logged in into the cluster with a user with cluster admin rights
* A Registry Credentials Secret with access to `registry.redhat.io` named `imagestreamsecret` should exist in the `openshift` Namespace. See https://access.redhat.com/RegistryAuthentication for more details.
* A MapBox access token. See https://docs.mapbox.com/help/how-mapbox-works/access-tokens/ for more details.


### Installing

Create a copy of the inventory template file:
```
$ cp inventories/inventory.template inventories/inventory
```

Replace the sample MapBox access token in the inventory file (`map_token`) with a real MapBox token. Assuming you have a token for mapbox. If you do not have a token, 

To provision all the components:
```
$ ansible-playbook -i inventories/inventory playbooks/install.yml
```

This could take upto 30 minutes to complete.

By default, the `developer` user will be given project admin permissions for
the Emergency Response projects. To give these permissions to another user,
override the `cluster_provisioner` variable during provisioning.

Individual components can be installed by the executing the corresponding playbook. For example, to install only the Kafka cluster:
```
$ ansible-playbook playbooks/strimzi_operator.yml
$ ansible-playbook playbooks/kafka_cluster.yml
$ ansible-playbook playbooks/kafka_topics.yml
```

Apart from the `strimzi_operator` playbook, none of the playbooks require cluster admin rights.

Headover to the getting started section to understand more about the demo components 

### Uninstalling

To uninstall:
```
$ ansible-playbook playbooks/install.yml -e ACTION=uninstall
```


## I am a Red Hat partner and would like to access this demo via the partner demo system.
1.  The `ssh` utility installed on your laptop.
    
    > **Note**
    > 
    > If your network connection is intermittent, consider installing
    > the [mosh](https://mosh.org/) utility (`yum install mosh`) as an
    > alternative to the `ssh` utility.

2.  Web browser installed on your laptop.

3.  Broadband internet connectivity.

4.  [Red Hat GPTE *Opentlc*
    userId](https://account.opentlc.com/account/)

### Overview

The target audience for this guide are those Red Hat Solution
Architects, Consultants and Red Hat Partners that are interested in
using the *Emergency Response* application to:

1.  Demonstrate the Red Hat middleware product suite running on
    OpenShift.

2.  Provide training on the Red Hat middleware product suite running on
    OpenShift.

If you are interested in building this demo from its source code, please
refer to the [TODO]() document.

#### Lab Virtual Machine

Your lab environment is remote and consists of the following:

1.  **OpenShift Container Platform** (OCP)

2.  [Emergency Response
    Demo](https://github.com/Emergency-Response-Demo/)

This lab environment can be accessed via ssh as well as through your
local browser.

#### Order Virtual Machine

This section guides you through the procedure to order a virtual machine
(VM) for this course.

1.  In a web browser, navigate to the *Red Hat Partner Demo System* at:
    <https://rhpds.redhat.com/catalog/explorer>.

2.  Authenticate using your *OPENTLC* credentials, for example:
    `johndoe-redhat.com`.

3.  Navigate to the following catalog: `Services → Catalog → Catalog
    Items → Multi-product Demos`.

4.  Select the following catalog item: `RHT Emergency Response`.

5.  Click `Order` on the next page.

6.  In the subsequent order form, select the check box confirming you
    understand the runtime and expiration dates. :

7.  At the bottom of the same page, click `Submit`.

#### Confirmation Emails

Upon ordering the lab environment, you will receive the following two
emails:

1.  **Your lab environment is building**
    
    1.  Save this email.
    
    2.  This email Includes details of the three VMs that make up your
        lab application similar to the following:
        
        ![aio_first_email](/images/aio_first_email.png)
    
    3.  Make note of the 4 digit GUID (aka: REGION CODE)
        
          - Whenever you see "GUID" or "$GUID" in a command, make sure
            to replace it with your GUID.
    
    4.  Make note of the URL of the `workstation` VM.
        
        You will use this when ssh’ing to your application.
    
    5.  Make note of the URL of the `master` VM.
        
        You will use this when accessing the OCP Web Console.
        
          - The OpenShift master URL varies based on the region where
            you are located, and may vary from the example shown above.
        
          - For the duration of the course, you navigate to this
            OpenShift Container Platform master node.

2.  **VM ready for authentication**
    
    Once you receive this second email, wait about another 5 minutes and
    then you can then ssh into the `workstation` VM of your Ravello
    application.

#### SSH Access and `oc` utility

SSH access to the remote lab environment provides you with the OpenShift
`oc` utility.

1.  ssh access to your lab environment by specifying your *opentlc
    userId* and lab environment $GUID in the following command:
    
        $ ssh <opentlc-userId>@workstation-$GUID.rhpds.opentlc.com

2.  Authenticate into OpenShift as a non cluster admin user (user1)
    using the `oc` utility
    
        $ oc login https://master00.example.com -u user1 -p r3dh4t1!
    
    > **Note**
    > 
    > If you initially receive a HTTP 403 or *no such host* response,
    > give it a minute or so and try again.

3.  OCP cluster admin access
    
    OCP cluster admin access is provided by switching to the root
    operating system of your lab environment as follows.
    
        $ sudo -i
        
        # oc login -u system:admin      # NOTE: This command is typically not needed
                                        #       /root/.kube/config already contains the _system:admin_ user's token

4.  Provide cluster admin access to OpenShift *user1*:
    
        # oc adm policy add-cluster-role-to-user cluster-admin user1

#### Refresh Red Hat SSO state

Your Red Hat SSO needs to be refreshed with valid *redirect* and
*web\_origin* URLs to support your Emergency Response demo. For this
purpose, a script has been provided as follows:

1.  SSH into the *workstation* node of your demo environment as disussed
    in the previous section.

2.  As the root operating system user, execute the following exactly as
    listed:
    
        $ sudo -i
        
        mkdir -p $HOME/lab && \
               wget https://raw.githubusercontent.com/gpte-emergency-response-demo/misc/master/erdemo_state_refresh.sh -O $HOME/lab/erdemo_state_refresh.sh \
               && chmod 755 $HOME/lab/erdemo_state_refresh.sh \
               && $HOME/lab/erdemo_state_refresh.sh

3.  You should see a response similar to the following:
    
        will update the following stale guid in RHSSO from: 43b5 to 5dff
        
        UPDATE 3
        UPDATE 2
        
        ...
        
        deploymentconfig.apps.openshift.io/sso rolled out
        deploymentconfig.apps.openshift.io/emergency-console rolled out
    
    If you are curious as to what exactly is getting modified in the
    RH-SSO, you can review [the
    script](https://raw.githubusercontent.com/gpte-emergency-response-demo/misc/master/erdemo_state_refresh.sh).
    
    In particular, notice that the *redirect\_uris* and *web\_origins*
    are modified to reflect the actual URL of your Emergency Response
    lab environment.

4.  After a couple of minutes, expect your RH-SSO pod to have
    re-started:
    
        $ oc get pods -n sso
        
        keycloak-operator-d894597dc-pkfkc   1/1       Running   1          5h
        sso-3-4rg52                         1/1       Running   0          1m
        sso-postgresql-1-dn4fl              1/1       Running   1          5h

5.  Exit out of the root operating system user shell:
    
        # exit
    
    Make sure to exit out of the root shell after every use

#### OpenShift Container Platform

Your lab environment is built on Red Hat’s OpenShift Container Platform
(OCP).

Access to your OCP resources can be gained via both the `oc` CLI utility
and the OCP web console.

#### OCP Web Console

1.  Point your browser to the URL created by executing the following :
    
        $ echo -en "\nhttps://master00-$GUID.generic.opentlc.com\n\n"
    
    > **Note**
    > 
    > Substitute `GUID` in the above command with the GUID of your lab
    > environment.

2.  Authenticate using the following user credentials provided in the confirmation email.


## Installation Complete!
Assuming everything has turned up smoothly. If you now login to the Openshift console you should be able to see the project "emergency-response-demo" and all the services running inside it. To understand how to run this demo, please move to the Getting Started page.






