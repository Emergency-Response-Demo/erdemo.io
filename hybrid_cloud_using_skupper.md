

cloud
`````
$ oc project user2-er-demo
$ skupper status
skupper not enabled for user2-er-demo

$ skupper init

$ skupper connection-token /tmp/public-cloud-er-demo-secret.yml

$ oc get route | grep skupper
skupper-controller           skupper-controller-user2-er-demo.apps.cluster-0ef9.0ef9.example.opentlc.com            skupper-controller           metrics        edge/Redirect      None
skupper-edge                 skupper-edge-user2-er-demo.apps.cluster-0ef9.0ef9.example.opentlc.com                  skupper-internal             edge           passthrough/None   None
skupper-inter-router         skupper-inter-router-user2-er-demo.apps.cluster-0ef9.0ef9.example.opentlc.com          skupper-internal             inter-router   passthrough/None   None

$ oc annotate service user2-disaster-simulator skupper.io/proxy=http
$ oc annotate service user2-emergency-console skupper.io/proxy=http

$ oc project user-sso
$ skupper init
$ skupper connection-token /tmp/public-cloud-user-sso-secret.yml
$ oc annotate service sso skupper.io/proxy=https
`````

local
`````
$ oc adm new-project developer-er-demo --admin=developer
$ oc project developer-er-demo

$ skupper status
skupper not enabled for developer-er-demo

$ skupper init

$ skupper connect /tmp/public-cloud-er-demo-secret.yml 
Skupper is now configured to connect to skupper-inter-router-user2-er-demo.apps.cluster-0ef9.0ef9.example.opentlc.com:443 (name=conn1)

$ oc expose service user2-disaster-simulator
$ oc expose service user1-emergency-console

$ oc adm new-project user-sso --admin=developer
$ oc project user-sso
$ skupper init
$ skupper connect /tmp/public-cloud-user-sso-secret.yml
`````
