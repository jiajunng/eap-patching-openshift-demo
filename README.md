EAP S2I patching demo on OpenShift
============================================================
NOTE: This is forked from [luck3y/eap-patch-using-secrets-cli](https://github.com/luck3y/eap-patch-using-secrets-cli) with modification to demo applying patch to EAP on OpenShift without downtime.

## Background
This example illustrates a general method to apply a one-off patch to EAP running on OpenShift with zero downtime. For more complicated patching, the source image should be built with patches already applied during the container build process.

## Requirements

- Running OpenShift cluster (Tested on v4.2)
- EAP running on OpenShift (Tested on v7.1.5)
- EAP patch file (Can be downloaded on [Red Hat Product Download](https://access.redhat.com/downloads/)), this example uses jbeap-16108.zip which is in the repository

## Basic instructions

- Create a project in OpenShift:

  ```$ oc new-project eap-patching-demo ```

- Create a secret for the patch that will be applied:

  ```$ oc create secret generic jbeap-16108 --from-file=jbeap-16108.zip=jbeap-16108.zip```

- Install the template (EAP v7.1.5):
    ``` 
    $ oc -n openshift replace --force -f eap71-basic-s2i-patching.json

- Create the application:
     ```
     $ oc new-app --template=eap71-basic-s2i-patching \
       -p SOURCE_REPOSITORY_URL="https://github.com/jiajunng/eap-patch-using-secrets-cli.git" \
       -p SOURCE_REPOSITORY_REF="master" \
       -p CONTEXT_DIR="helloworld" \
       -p APPLICATION_NAME="eap-patching-demo" 

- Change the update strategy from Recreate to Rolling in the deploymentconfig of EAP: 
    ```
    $ oc patch dc eap-patching-demo -p '{"spec":{"strategy":{"type":"Rolling"}}}'
    
- Mount the secrets into the buildconfig of the EAP:
    ```
    $ oc patch bc/eap-patching-demo -p "$(<patching.json)"

- Trigger the new build:
    ```
    $ oc start-build eap-patching-demo 
  
- When the pod is created, a message similar to the following should be logged during boot: 
    ``` 
    03:19:08,195 INFO  [org.jboss.as.patching] (MSC service thread 1-2) WFLYPAT0050: JBoss EAP cumulative patch ID is: jboss-eap-7.1.5.CP, one-off patches include: eap-715-jbeap-16108
