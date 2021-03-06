

- [1. Installation Overview](#1-installation-overview)
- [2. Pre-requisites](#2-pre-requisites)
  - [2.1. Local Tooling](#21-local-tooling)
  - [2.2. MapBox Access Token](#22-mapbox-access-token)
  - [2.3. OpenShift](#23-openshift)
    - [2.3.1. Minimum Requirements](#231-minimum-requirements)
- [3. Installation Procedure](#3-installation-procedure)
  - [3.1. Pre-built Images](#31-pre-built-images)
  - [3.2. CI/CD](#32-cicd)
  - [3.3. Uninstalling](#33-uninstalling)
- [4. Installation Complete!](#4-installation-complete)
- [5. Appendix](#5-appendix)
  - [5.1. OCP4 from RHPDS](#51-ocp4-from-rhpds)
    - [5.1.1. Order OCP4](#511-order-ocp4)
    - [5.1.2. Confirmation Emails](#512-confirmation-emails)
    - [5.1.3. Access](#513-access)


# 1. Installation Overview
By this time you are excited and want to try out this application.  To do so, you will need to install the application on an OpenShift Container Platform (OCP) 4.* environment.  The installation of the application on OpenShift is done using Ansible.

Using Ansible, there are two options for the installation of the Emergency Response app on an OCP 4 environment:

1. **Pre-built Linux container images** 
   
   This approach is best suited for those that want to utilize (ie: for a customer demo) the Emergency Response app.  This installation approach does not use Jenkins pipelines.  Instead, the OpenShift _deployments_ for each component of the Emergency Response application are started using pre-built Linux container images pulled from corresponding [public Quay image repositories](https://quay.io/organization/emergencyresponsedemo).  With this approach, the typical duration to build the Emergency Response app is about 20 minutes.  This is the default installation approach.

2. **CI/CD**
   
   This approach is best suited for code contributors to the Emergency Response app.  Individual Jenkins pipelines are provided for each component of the app.  Each pipeline builds from source, tests, creates a Linux container image and deploys that image to OpenShift.  The typical duration to build the Emergency Response application from source using these pipelines is about an hour.  This approach is also of value if the focus of a demo to a customer and/or partner is to introduce them to CI/CD best practices of a microservice architected application deployed to OpenShift.





# 2. Pre-requisites

## 2.1. Local Tooling

To install the Emergency Response application, you will need the following tools on your local machine:

1. **Unix flavor OS with BASH shell**:  ie; Fedora, RHEL, CentOS, Ubuntu, OSX
2. **git**
3. **[oc utility v4.2](https://mirror.openshift.com/pub/openshift-v4/clients/oc/4.2/)**
4. **[Ansible](https://www.redhat.com/en/technologies/management/ansible)**
   
   Installation of the Emergency Response application is tested using the _ansible-playbook_ utility from the _ansible_ package of Fedora 31.  Others in the community have also succeeded in installing the app using ansible on OSX.

## 2.2. MapBox Access Token

The Emergency Response application makes use of a third-party SaaS API called [MapBox](https://www.mapbox.com/).
MapBox APIs provide the Emergency Response application with an optimized route for a responder to travel given pick-up and drop-off locations.  To invoke its APIs, MapBox requires an [access token](https://docs.mapbox.com/help/how-mapbox-works/access-tokens).  For normal use of the Emergency Response application the _free-tier_ account provides ample rate-limits.

![MapBox token](/images/mapbox_token.png)

## 2.3. OpenShift

You can utilize your own vanilla OpenShift 4 environment so long as it meets the minimum requirements described below in this section.
The benefit of utilizing your own OpenShift environment is that you decide if/when to shut it down and the duration of its lifetime.  In addition, if there are any errors in the provisioning process of OpenShift, you will have some ability to troubleshoot the problem.

Otherwise, if you are a Red Hat associate or Red Hat partner, you can order an OpenShift 4 environment from Red Hat's _Partner Demo System_ (RHPDS).  Using RHPDS, the minimum requirements described below are met.  However, your OpenShift environment will shut down at known periods (typically 10 hours) and will be deleted after a certain duration (typically 2 days).  Also, **the failure rates of provisioning a base OpenShift from RHPDS are known to be high**.  (Each provisioning attempt consists of hundreds of steps, many of which rely on third-party services). You may need to attempt numerous times over the course of days.  Details pertaining to accessing an OpenShift 4 environment from RHPDS are found in [the Appendix](##51-ocp4-from-rhpds) of this document.

### 2.3.1. Minimum Requirements
To install the Emergency Response application, you will need an OpenShift Container Platform environment with the following minimum specs:

1. **OCP Version:**  4.2 (although any version of OpenShift in the 4.* family will most likely work)  
2. **Memory:**    24 GBi allocated to one or more _worker_ node(s)
3. **CPU:** 10 cores allocated to one or more _worker_ nodes
4. **Disk:** 50 GB of storage that supports [Read Write Once (RWO)](https://docs.openshift.com/container-platform/4.2/storage/understanding-persistent-storage.html#pv-access-modes_understanding-persistent-storage).
   
   NOTE:  The Emergency Response application currently does not require Read-Write-Many (RWX).

5. **Credentials:**  You will need _cluster-admin_ credentials to your OpenShift environment.
6. **CA signed certificate:** Optional
   
   Preferably, all public routes of your Emergency Response application utilize a SSL certificate signed by a legitimate certificate authority.  ie:  [LetsEncrypt](https://letsencrypt.org/)
7. **Non cluster-admin users**:
   The Emergency Response application will be owned by a non cluster-admin _project administrator_.  In your OpenShift environment, you will need one or more non cluster-admin users to serve this purpose.  In _Code Ready Containers_, the default non cluster-admin user is:  _developer_ .  In other OpenShift clusters, the convention tends to be:  user[1-200].
8. **Pull Secret**:
   Some Linux container images used in the Emergency Response application reside in the following secured image registry:  _registry.redhat.io_.
   Those images will need to be pulled to your OpenShift 4 environment.
   As part of the installation of OCP4, you should have already been prompted to provide your [pull secret](https://cloud.redhat.com/openshift/install/pull-secret) that enables access to various secured registries to include regisry.redhat.io.







# 3. Installation Procedure
Now that you have an OpenShift environment that meets the minimum requirements, you can now layer the Emergency Response application on that OpenShift.

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

4. Copy the _inventory.template_
   ```
   cp inventories/inventory.template inventories/inventory
   ```

5. Using your favorite text editor, open the copied file: _inventories/inventory_
6. Replace the sample MapBox access token (`map_token`) with a real MapBox token.
   ```
   # MapBox API token, see https://docs.mapbox.com/help/how-mapbox-works/access-tokens/
   map_token=pk.egfdgewrthfdiamJyaWRERKLJWRIONEWRwerqeGNjamxqYjA2323czdXBrcW5mbmg0amkifQ.iBEb0APX1Vmo-2934rj
   ```
7. Save the changes.
8. Set an environment variable that reflects the userId of your non cluster-admin user.  ie:
   ```
   OCP_USERNAME=user1
   ```

You will now execute the ansible to install the Emergency Response application.
**Select one of the following two approaches:  _Pre-built Images_ or _CI/CD_**

## 3.1. Pre-built Images
This approach is best suited for those that want to utilize (ie: for a customer demo) the Emergency Response app.  This installation approach does not use Jenkins pipelines.  Instead, the OpenShift _deployments_ for each component of the Emergency Response application are started using pre-built Linux container images pulled from corresponding [public Quay image repositories](https://quay.io/organization/emergencyresponsedemo).  With this approach, the typical duration to build the Emergency Response app is about 20 minutes.  This is the default installation approach.

1. From the _install/ansible_ directory, kick-off the Emergency Response app provisioning:
   ```
   ansible-playbook -i inventories/inventory playbooks/install.yml \
                    -e project_admin=$OCP_USERNAME
   ```
2. After about 20 minutes, you should see ansible log messages similar to the following:
   ```
   PLAY RECAP ********************************************************************************
   localhost : ok=432  changed=240  unreachable=0    failed=0    skipped=253  rescued=0    ignored=0 
   ```


## 3.2. CI/CD

This approach is best suited for code contributors to the Emergency Response app.  Individual Jenkins pipelines are provided for each component of the app.  Each pipeline builds from source, tests, creates a Linux container image and deploys that image to OpenShift.  The typical duration to build the Emergency Response application from source using these pipelines is about an hour.  This approach is also of value if the focus of a demo to a customer and/or partner is to introduce them to CI/CD best practices of a microservice architected application deployed to OpenShift.

1. From the _install/ansible_ directory, kick-off the Emergency Response app provisioning:
   ```
   ansible-playbook -i inventories/inventory playbooks/install.yml \
                    -e deploy_from=source \
                    -e project_admin=$OCP_USERNAME
   ```
   
2. After about an hour, the provisioning should be complete.

3. You can review any of the CI/CD pipelines that ran as part of this provisioning process:
   1. List all build pipelines:
       ```
       oc get build -n $OCP_USERNAME-tools-erd | grep pipeline

       ...

       user2-incident-service-pipeline-1         JenkinsPipeline            Complete   2 hours ago         
       user2-mission-service-pipeline-1          JenkinsPipeline            Complete   2 hours ago         
       user2-responder-service-pipeline-1        JenkinsPipeline            Complete   2 hours ago         
       user2-assignment-rules-model-pipeline-1   JenkinsPipeline            Complete   2 hours ago  
       ```
   2. From that list, pick any of the builds to get the URL to the corresponding Jenkins pipeline:
      ```
      oc logs user2-incident-service-pipeline-1 -n $OCP_USERNAME-tools-erd

      info: logs available at https://jenkins-user2-tools-erd.apps.cluster-denver-8ab6.denver-8ab6.example.opentlc.com/blue/organizations/jenkins/user2-tools-erd%2Fuser2-tools-erd-user2-mission-service-pipeline/detail/user2-tools-erd-user2-mission-service-pipeline/1/
      ```
   3. Open a browser tab and navigate to the provided URL:
      ![](/images/pipeline_example.png)



## 3.3. Uninstalling

To uninstall:
```
$ ansible-playbook playbooks/install.yml \
                   -e ACTION=uninstall \
                   -e project_admin=$OCP_USERNAME
```

# 4. Installation Complete!

Now that installation of the Emergency Response app is complete, you should be able to navigate your browser to the following URLs:

1. **Emergency Response Console**
   ```
   echo -en "\nhttps://$(oc get route $OCP_USERNAME-emergency-console -n $OCP_USERNAME-er-demo --template='{{ .spec.host }}')\n\n"
   ```
   ![](/images/erdemo_home.png)

2. **Disaster Simulator**
   ```
   echo -en "\nhttp://$(oc get route $OCP_USERNAME-disaster-simulator -n $OCP_USERNAME-er-demo --template='{{.spec.host}}')\n\n"
   ```
   ![](/images/disaster_simulator.png)


Further details regarding how to run this demo can be found in the _Getting Started_ page.


# 5. Appendix

## 5.1. OCP4 from RHPDS

The _Red Hat Partner Demo System_ (RHPDS) provides a wide variety of cloud-based labs and demos showcasing Red Hat software.
One of the offerings from RHPDS is a cloud-based OCP 4 environment that meets all of the minimum requirements to support an installation of the Emergency Response application.  The default shutdown and lifetime durations of this environment are as follows:

* **Runtime:**  10 hours;
  You have the ability to extend the runtime (one time only) and re-start if it was shutdown.

* **Lifetime:** 2 days;
  You have the ability 

To utilize RHPDS, you will need the following:

1. [OPENTLC credentials](https://account.opentlc.com/account/).  OPENTLC credentials are available only to Red Hat associates and Red Hat partners.
2. SFDC Opportunity, Campaign ID or Partner Registration


### 5.1.1. Order OCP4


1.  In a web browser, navigate to the *Red Hat Partner Demo System* at:
    [Red Hat Partner Demo System](https://rhpds.redhat.com/catalog/explorer).

2.  Authenticate using your *OPENTLC* credentials, for example:
    `johndoe-redhat.com`.

3.  Navigate to the following catalog: `Services → Service Catalogs → Workshops.

4.  Select the following catalog item: `OpenShift 4.2 Workshop`.

5.  Click `Order` on the next page.

6.  In the subsequent order form, add in details similar to the following:
    ![Order form](/images/rhpds_order_form.png)   

7.  At the bottom of the same page, click `Submit`.

### 5.1.2. Confirmation Emails

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
    


### 5.1.3. Access
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