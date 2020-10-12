## FORKED FROM IBM Cloud Garage Tekton Pipelines for IBM Cloud Secret Manager task.
This repo is forked from the IBM cloud Garage repo for adding the IBM Cloud Secret Manager task as part of the pipeline.

2-ibm-secret-manager.yaml:

This tekton pipeline task is integrated with the pipeline to pull the secrets from the IBM Cloud Secret Manager Service and make it available as part of the application secrets in the Openshift Cluster environment.

As part of the task, using the ibmcloud apikey available as part of the environment, the task generates the IBM Cloud Bearer Token that is essential to access the Secret Manager.  Apart from the IBM TOKENS, it also needs the 
    1. "Name" for the secret (that to be created in the Openshift environment for the application to use)
    2. "Base URL" to access the IBM Secret Manager Service
    3. "Secret Type"
    4. "Secret ID"
    
Using the available parameter, the Secret manager provides API Access to retrieve the Secret Data stored in the Service.
Note: The current task uses static data ("Name", "Base URL", "Secret Type", "Secret ID") that has to be parameterised during the Integration.

Below content provides the details on the IBM Cloud Garage Pipeline

## IBM Cloud Garage Tekton Pipelines

This repository provides Tekton pipelines and tasks for all nodejs and java application Code Pattern templates.

### Install the tasks and pipelines

The best way to install the tasks and template pipelines is through the versioned releases. The following
steps will get the tasks installed in your cluster. **Note:** These instructions assume you have already
logged into the cluster.

1. Look through the releases and select the one that should be installed - https://github.com/IBM/ibm-garage-tekton-tasks/releases
2. From the command-line, run the following (substituting the `RELEASE` and `NAMESPACE` values as appropriate):
    ```shell script
    RELEASE=$(curl -s https://api.github.com/repos/IBM/ibm-garage-tekton-tasks/releases/latest | jq -r '.tag_name')
    export NAMESPACE="tools"
    kubectl apply -n ${NAMESPACE} -f https://github.com/IBM/ibm-garage-tekton-tasks/releases/download/${RELEASE}/release.yaml
    ```

### Get the code

Clone this repo.

```bash
git clone git@github.com:IBM/ibm-garage-tekton-tasks.git
cd ibm-garage-tekton-tasks
```

## Service account to run Pipeline

If you install Tekton using the OpenShift Pipeline Operator on OCP4, a service account `pipeline` is already created and you can skip the following commands.

Create a service account like `pipeline`
```
oc create serviceaccount pipeline
oc adm policy add-scc-to-user privileged -z pipeline
oc adm policy add-role-to-user edit -z pipeline
```

### Create Pipeline Tasks

IMPORTANT: If Tekton version is lower than `0.7.0` then use the tasks from the `pre-0.7.0` directory.

- Create pipelines tasks for each environment for example the `dev` namespace:

    ```bash
    kubectl create -f ibm-garage-tekton-tasks/pre-0.7.0/tasks/ -n dev
    ```

- If using Tekton version `0.7.0` or greater use this command instead:

    ```bash
    kubectl create -f ibm-garage-tekton-tasks/tasks/ -n dev
    ```

This step will create the following tasks:
- ibm-nodejs-tests
- ibm-java-gradle-tests
- ibm-build-push.yaml
- ibm-build-tag-push.yaml
- ibm-build-tag-push-ibm.yaml
- ibm-deploy
- ibm-health-check
- ibm-helm-package
- ibm-gitops

### Create Pipelines

- Create pipelines for each environment for example the `dev` namespace.

    ```bash
    kubectl create -f ibm-garage-tekton-tasks/pipelines/ -n dev
    ```

This step will create following Pipelines:

- ibm-java-gradle
- ibm-nodejs

### Manually run a Pipeline

- The internal container registry service hostname is different for ocp3 and ocp4 
  - ocp3: `docker-registry.default.svc:5000`
  - ocp4: `image-registry.openshift-image-registry.svc:5000`

- To install the input pipeline resources for the respective application template run the following commands:
    
    For ocp4:
    ```bash
    OCP=ocp4 kubectl apply -f test/resources/$OCP/
    ```

    For ocp3:
    ```bash
    OCP=ocp3 kubectl apply -f test/resources/$OCP/
    ```

- Run a pipeline for one of the application templates using the Tekton CLI `tkn` and the helper script
    
    ```bash
    Usage: ./test/scripts/run.sh [nodejs-typescript | nodejs-react | nodejs-angular | java-spring]
    ```
    For example to run the pipeline for the application template `nodejs-typescript`
    ```bash
    ./test/scripts/run.sh nodejs-typescript
    ```
    The script will output the name of the pipelinerun, and a command to follow the logs
    ```
    Pipelinerun started: ibm-nodejs-run-fqgr7

    In order to track the pipelinerun progress run:
    tkn pipelinerun logs ibm-nodejs-run-fqgr7 -f -n dev
    ```

### Create Git Webhook

- Create a Git Webhook on the `dev` namespace using the tekton dashboard.

Now, your pipeline runs whenever the changes are pushed to the repository.
