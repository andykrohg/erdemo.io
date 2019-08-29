# docs
    - https://docs.google.com/presentation/d/12pSURFjOI0FCIoQ4F0aI4S3cmLmzKfNU9US5vKCOzEo/edit#slide=id.g5fa6e356b7_0_354   :   Chuck preso; 19 August 2019
    - https://bluejeans.com/playback/s/L3NXTdoLZ5JizUlmA53GFCtNn9kCV1We9PvEj4pZ0OM45FcIkhCpFiD3busPDgeC                     :   Shaf recording
    - https://docs.google.com/presentation/d/1kRaYUJjU1nR-7fCkTW-EVSu2Z2r7oRK49GMJSm3BzX8/edit#slide=id.g5927df6944_0_89    :   intial Google slide deck
    - https://docs.google.com/document/d/17W3D3Fd2XiCJYDuwHhkCDWEExD5NZPHRv-eJamLmjW0/edit#                                 :   ER System Demo
    - http://www.erdemo.io/                                                                                                 :   erdemo homepage
    - https://github.com/Emergency-Response-Demo                                                                            :   github
    - https://docs.google.com/document/d/145jBCx3wDjB23CzBNU-N2rYM2WBbIW3ZrSZ5VwXDrNA/edit?ts=5d272b32                      :   content planning
    - https://gist.github.com/aidenkeating/f942b1a836b296791471e9cf731a6947 

# Installation

Integreatly installer branch to install a minimum Integreatly stack (basically an instance of RH-SSO and the monitoring stack):
    - https://github.com/btison/integr8ly-installation/tree/emergency-response-demo .

    - The playbook can be run from your local workstation (no need to run it from bastion).
    - You need to be logged in as admin.
    - You need to provide an inventory file that looks like (adapt urls as needed):
[local:vars]
ansible_connection=local

run_master_tasks=false
eval_app_host=apps.5eb0.openshift.opentlc.com
openshift_master_url=https://master.5eb0.openshift.opentlc.com

[local]
127.0.0.1

[OSEv3:children]
master

[OSEv3:vars]
ansible_user=ec2-user

[master]
master.5eb0.openshift.opentlc.com

[master:vars]
run_master_tasks=false

## Procedure
So starting from scratch:
# dnf install python3-jsonpointer
$ git clone https://github.com/btison/integr8ly-installation.git
$ git checkout emergency-response-demo
$ cp inventories/hosts.template inventories/hosts  # modify inventories/hosts as detailed above
$ ansible-playbook -i inventories/hosts playbooks/install.yml

## Installs the following namespaces

        1)  console-config
        2)  managed-service-broker
        3)  middleware-monitoring
        4)  sso
        5)  user-sso
        6)  webapp


## SSO

### Default login info:
      - integr8ly-installation/roles/rhsso/defaults/main.yml

        - rhsso_cluster_admin_username  : admin
        - rhsso_cluster_admin_password  : Password1
        - rhsso_realm                   : openshift

### sso
- `echo -en "\n\nhttps://`oc get route sso -n sso --template "{{.spec.host}}"`/auth/admin\n"`

- `oc project sso`
- Obtain admin userId / password
+
`oc rsh `oc get pod | grep "^sso-1" | awk '{print $1}'` env | grep SSO_ADMIN`

#### openshift realm

. Users:  
.. customer-admin           (rhsso_evals_admin_username / rhsso_evals_admin_password)
.. evals01

#### emergency-realm
. Users:
.. jbride

### user-sso

Appears to only include the master realm upon provisioning of ER Demo

- `echo -en "\n\nhttps://`oc get route sso -n user-sso --template "{{.spec.host}}"`/auth/admin/\n"`

- `oc project user-sso`
- Obtain admin userId / password
+
`oc rsh `oc get pod | grep "^sso-1" | awk '{print $1}'` env | grep SSO_ADMIN`


# Install

Once Integreatly is installed, you can run the Emergency-Response-Demo installation playbook.

Account / Boat details
    - ie; Phone Number, Boat Capacity, Has Medical
    - Without this info, the boat is not included in the selection process of missions.
    - this is done when registering as a new user in SSO. Requires the custom login screen
    - The custom login SSO screen is part of a RHSSO custom theme added to the SSO pod with oc rsync.
    - This is far from ideal as you lose the theme when the SSO pod is restarted.
    - There is an open issue for this (https://github.com/Emergency-Response-Demo/install/issues/28), but no fix yet.

The grafana dashboard Chuck was showing is appplication-specific, separate from th OpenShift infra metrics
That's something we're looking into. We want to be able to do post-event data-analysis by replaying the events into something like Spark.




# Kafka

- `oc describe Kafka kafka-cluster`
- `oc get KafkaTopics` 
