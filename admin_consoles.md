- [1. Admin Consoles Overview](#1-admin-consoles-overview)
- [2. SSO Admin Console](#2-sso-admin-console)
- [3. Emergency Response Admin Console](#3-emergency-response-admin-console)
  - [3.1. Priority Zone](#31-priority-zone)
    - [3.1.1. Purpose](#311-purpose)
    - [3.1.2. Procedure](#312-procedure)


# 1. Admin Consoles Overview

# 2. SSO Admin Console

1. Point a browser tab to the output of the following command:
   ```
   echo -en "\n\nhttps://$(oc get route sso --template='{{ .spec.host }}' -n user-sso)/auth/admin/master/console/#/realms/master\n\n"
   ```
   ![SSO Admin Login](images/sso_admin_login.png)



2.  Authenticate in the _master_ realm of SSO using a userId of _admin_ and a password determined via the following:
    ```
    oc project user-sso

    pod_id=$(oc get pod -o json | jq '.items[]|select(.metadata.labels.deploymentConfig=="sso").metadata.name' | sed 's/"//g')

    oc rsh $pod_id env | grep SSO_ADMIN_PASSWORD

    ```

3.  Switch to your desired SSO realm by hovering over the realm drop-down in the top-left of the page:
   
    ![Realm selection](images/sso_select_realm.png)

# 3. Emergency Response Admin Console

## 3.1. Priority Zone

### 3.1.1. Purpose

### 3.1.2. Procedure

![Create Priority Zone](images/create_priority_zone.png)
