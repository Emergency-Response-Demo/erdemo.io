---
layout: page
title: CI/CD
subtitle: DevSecOps
---

# 1. Introduction
The ER-Demo allows for provisioning from either source via CI/CD pipelines or from tagged binary images hosted in the Quay container image repository.

This document is a deep-dive into the former approach:  Provisioning from source using CI/CD pipelines.

# 2.  Jenkins Scripted vs Declarative Pipelines

Jenkins pipelines can be written in either a [Scripted or Declarative](https://opensource.com/article/18/8/devops-jenkins-2) approach.

The pipelines in the ER-Demo make use of the _Declarative_ approach.

# 3.  Jenkins Master node

# 4.  Jenkins Worker Nodes

Reference
    - https://docs.openshift.com/container-platform/4.5/openshift_images/using_images/images-other-jenkins.html#images-other-jenkins-config-kubernetes_images-other-jenkins



Auto-Discovery

The Jenkins image also provides auto-discovery and auto-configuration of slave images for the Kubernetes plug-in. 
With the OpenShift Sync plug-in, the Jenkins image on Jenkins start-up searches within the project that it is running, or the projects specifically listed in the plug-in’s configuration for the following:

    Image streams that have the label role set to jenkins-slave.

    Image stream tags that have the annotation role set to jenkins-slave.

    ConfigMaps that have the label role set to jenkins-slave.

When it finds an image stream with the appropriate label, or image stream tag with the appropriate annotation, it generates the corresponding Kubernetes plug-in configuration so you can assign your Jenkins jobs to run in a pod running the container image provided by the image stream.



```
$ oc get cm jenkins-maven-slave -o json | jq .metadata.labels.role

"jenkins-slave"
```

```
oc get cm jenkins-maven-slave -o yaml | more

```

```
oc get pvc jenkins-maven-slave-repository
```

Integration with Nexus 
````
        <envVars>
            <org.csanchez.jenkins.plugins.kubernetes.model.KeyValueEnvVar>
              <key>MAVEN_MIRROR_URL</key>
              <value>http://nexus.user1-er-tools.svc:8081/content/groups/public</value>
            </org.csanchez.jenkins.plugins.kubernetes.model.KeyValueEnvVar>
        </envVars>
````

```
maven-with-nexus
```

#jenkins_maven_slave_configmap_image: image-registry.openshift-image-registry.svc:5000/openshift/jenkins-agent-maven:latest
jenkins_maven_slave_configmap_image: quay.io/emergencyresponsedemo/jenkins_slave_upgraded_mvn:1.0
