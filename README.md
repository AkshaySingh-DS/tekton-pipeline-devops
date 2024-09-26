# Intro to CI/CD Practice Code

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Python 3.9](https://img.shields.io/badge/Python-3.9-green.svg)](https://shields.io/)

This repository contains the practice code for the labs in **IBM-CD0215EN-SkillsNetwork Introduction to CI/CD**

## Contents

- Lab 1: [Build an empty Pipeline](labs/01_base_pipeline/README.md)
- Lab 2: [Adding GitHub Triggers](labs/02_add_git_trigger/README.md)
- Lab 3: [Use Tekton CD Catalog](labs/03_use_tekton_catalog/README.md)
- Lab 4: [Integrate Unit Test Automation](labs/04_unit_test_automation/README.md)
- Lab 5: [Building an Image](labs/05_build_an_image/README.md)
- Lab 6: [Deploy to Kubernetes](labs/06_deploy_to_kubernetes/README.md)

## Instructor

John Rofrano, Senior Technical Staff Member, DevOps Champion, @ IBM Research

## <h3 align="center"> © IBM Corporation 2022. All rights reserved. <h3/>


=================================== Tekton Concepts =======================================

FOR LAB02:-

Step 1: Create an EventListener

The first thing you need is an event listener that is listening for incoming events from GitHub.
You will update the eventlistener.yaml file to define an EventListener named cd-listener that references a TriggerBinding named cd-binding and a TriggerTemplate named cd-template

Step 2: Create a TriggerBinding

The next thing you need is a way to bind the incoming data from the event to pass on to the pipeline. To accomplish this, you use a TriggerBinding.

Update the triggerbinding.yaml file to create a TriggerBinding named cd-binding that takes the body.repository.url and body.ref and binds them to the parameters repository and branch, respectively.

Step 3: Create a TriggerTemplate

The TriggerTemplate takes the parameters passed in from the TriggerBinding and creates a PipelineRun to start the pipeline.

Update the triggertemplate.yaml file to create a TriggerTemplate named cd-template that defines the parameters required, and create a PipelineRun that will run the cd-pipeline you created in the previous lab.


Step 4: Start a Pipeline Run

Now it is time to call the event listener and start a PipelineRun. You can do this locally using the curl command to test that it works.

you need to run the kubectl port-forward command to forward the port for the event listener so that you can call it on localhost.
Use the kubectl port-forward command to forward port 8090 to 8080.

kubectl port-forward service/el-cd-listener  8090:8080

Now you are ready to trigger the event listener by posting to the endpoint that it is listening on.

curl -X POST http://localhost:8090 \
  -H 'Content-Type: application/json' \
  -d '{"ref":"main","repository":{"url":"https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode"}}'


FOR LAB03:-

Step 1: Add the git-clone Task

You start by finding a task to replace the checkout task you initially created. While it was OK as a learning exercise, it needs a lot more capabilities to be more robust, and it makes sense to use the community-supplied task instead.

(Optional) You can browse the Tekton Hub, find the git-clone command, copy the URL to the yaml file, and use kubectl to apply it manually. But it is much easier to use the Tekton CLI once you have found the task that you want.

Use this command to install the git-clone task from Tekton Hub:

```tkn hub install task git-clone --version 0.8```

This installs the git-clone task into your cluster under your current active namespace.

Step 2: Create a Workspace

Viewing the git-clone task requirements, you see that while it supports many more parameters than your original checkout task, it only requires two things:

The URL of a Git repo to clone, provided with the url param
A workspace called output
You start by creating a PersistentVolumeClaim (PVC) to use as the workspace:

A workspace is a disk volume that can be shared across tasks. The way to bind to volumes in Kubernetes is with a PersistentVolumeClaim.

Since creating PVCs is beyond the scope of this lab, you have been provided with the following pvc.yaml file with these contents:

You can now reference this persistent volume by its name pipelinerun-pvc when creating workspaces for your Tekton tasks.

Step 3: Add a Workspace to the Pipeline

In this step, you will add a workspace to the pipeline using the persistent volume claim you just created. To do this, you will edit the pipeline.yaml file and add a workspaces: definition as the first line under the spec: but before the params: and call it pipeline-workspace. Then you will add the workspace to the pipeline clone task and change the task to reference git-clone instead of your checkout task.

Step 4: Run the Pipeline

You can now use the Tekton CLI (tkn) to create a PipelineRun to run the pipeline.

Use the following command to run the pipeline, passing in the URL of the repository, the branch to clone, the workspace name, and the persistent volume claim name.

```
tkn pipeline start cd-pipeline \
    -p repo-url="https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git" \
    -p branch="main" \
    -w name=pipeline-workspace,claimName=pipelinerun-pvc \
    --showlog
```


Output of suceessful run:-

```
theia@theiaopenshift-singhakshaye:/home/project$ tkn tasks ls
NAME        DESCRIPTION              AGE
checkout                             21 hours ago
echo                                 21 hours ago
git-clone   These Tasks are Git...   32 minutes ago

theia@theiaopenshift-singhakshaye:/home/project$ kubectl get pvc
NAME              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS             AGE
pipelinerun-pvc   Bound    pvc-1eb4119a-aa71-403e-aab7-e07c0eb7016e   1Gi        RWO            skills-network-learner   32m


tekton_catalog$ tkn pipelinerun ls
NAME                    STARTED          DURATION   STATUS
cd-pipeline-run-cj6pw   13 minutes ago   32s        Succeeded
cd-pipeline-run-fnqcq   12 hours ago     29s        Succeeded
cd-pipeline-run-dczc2   12 hours ago     42s        Succeeded
cd-pipeline-run-lvtn6   19 hours ago     0s         Failed(ParameterMissing)
cd-pipeline-run-j9dxk   19 hours ago     0s         Failed(ParameterMissing)
```

Summary of Lab03:-

In this lab, you learned how to use the git-clone task from the Tekton catalog. You learned how to install the task locally using the Tekton CLI and how to modify your pipeline to reference the task and configure its parameters. You also learned how to start a pipeline with the Tekton CLI pipeline start command and monitor its output using --showlog


Lab04:-
Learning Objectives - Integrating Unit Test Automation

After completing this lab, you will be able to:

- Use the Tekton CD catalog to install the flake8 task
- Describe the parameters required to use the flake8 task
- Use the flake8 task in a Tekton pipeline to lint your code
- Create a test task from scratch and use it in your pipeline

Step 0: Check for cleanup

Please check as part of Step 0 for the new cleanup task which has been added to tasks.yaml file.

When a task that causes a compilation of the Python code, it leaves behind .pyc files that are owned by the specific user. For consecutive pipeline runs, the git-clone task tries to empty the directory but needs privileges to remove these files and this cleanup task takes care of that.

The init task is added pipeline.yaml file which runs everytime before the clone task.

Check the tasks.yaml file which has the new cleanup task updated.

Step 1: Add the flake8 Task

Your pipeline has a placeholder for a lint step that uses the echo task. Now it is time to replace it with a real linter.

You are going to use flake8 to lint your code. Luckily, Tekton Hub has a flake8 task that you can install and use:

Use the following Tekton CLI command to install the flake8 task into your namespace.

```tkn hub install task flake8```

Step 2: Modify the Pipeline to Use flake8

Now you will modify the pipeline.yaml file to use the new flake8 task.

In reading the documentation for the flake8 task, you notice that it requires a workspace named source. Add the workspace to the lint task after the name:, but before the taskRef:.

Step 3: Modify the Parameters for flake8

Now that you have added the workspace and changed the task reference to flake8, you need to modify the pipeline.yaml file to change the parameters to what flake8 is expecting.

In reading the documentation for the flake8 task, you see that it accepts an optional image parameter that allows you to specify your own container image. Since you are developing in a Python 3.9-slim container, you want to use python:3.9-slim as the image.

The flake8 task also allows you to specify arguments to pass to flake8 using the args parameter. These arguments are specified as a list of strings where each string is a parameter passed to flake8. For example, the arguments --count --statistics would be specified as: ["--count", "--statistics"].

Edit the pipeline.yaml file:

Step 5: Create a Test Task

Your pipeline also has a placeholder for a tests task that uses the echo task. Now you will replace it with real unit tests. In this step, you will replace the echo task with a call to a unit test framework called nosetests.

There are no tasks in the Tekton Hub for nosetests, so you will write your own.

Update the tasks.yaml file adding a new task called nose that uses the shared workspace for the pipeline and runs nosetests in a python:3.9-slim

Step 6: Modify the Pipeline to Use nose

The final step is to use the new nose task in your existing pipeline in place of the echo task placeholder.

Edit the pipeline.yaml file.

 Open pipeline.yaml in IDE

Add the workspace to the tests task after the name but before the taskRef:, change the taskRef to reference your new nose task, and change the message parameter to pass in your new args parameter.

Step 7: Run the Pipeline Again

Now that you have your tests task complete, run the pipeline again using the Tekton CLI to see your new test tasks run:

```
tkn pipeline start cd-pipeline \
    -p repo-url="https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git" \
    -p branch="main" \
    -w name=pipeline-workspace,claimName=pipelinerun-pvc \
    --showlog
```



```
NAME                    STARTED          DURATION   STATUS
cd-pipeline-run-sdcqc   3 minutes ago    1m27s      Succeeded
cd-pipeline-run-grgw7   20 minutes ago   0s         Failed(CouldntGetTask)


theia@theiaopenshift-singhakshaye:/home/project$ tkn tasks ls
NAME        DESCRIPTION              AGE
checkout                             21 hours ago
cleanup     This task will clea...   26 minutes ago
echo                                 21 hours ago
flake8      This task will run ...   41 minutes ago
git-clone   These Tasks are Git...   1 hour ago
nose                                 7 minutes ago
```

Lab05:-
Welcome to the hands-on lab for Building an Image. You are now at the build step, which is the next to last step in your CD pipeline. Before you can deploy your application, you need to build a Docker image and push it to an image registry. Luckily, there is a ClusterTask from the Tekton catalog available on your cluster that can do that – the buildah ClusterTask.

Learning Objectives

After completing this lab, you will be able to:

- Determine which ClusterTasks are available on your cluster
- Describe the parameters required to use the buildah ClusterTask
- Use the buildah ClusterTask in a Tekton pipeline to build an image and push it to an image registry

Step 1: Check for ClusterTasks

Your pipeline currently has a placeholder for a build step that uses the echo task. Now it is time to replace it with a real image builder.

You search Tekton Hub for the word "build" and you see there is a task called buildah that will build images so you decide to use the buildah task in your pipeline to build your code.

Instead of installing it yourself, you first check the ClusterTasks in your cluster to see if it already exists. Luckily, the OpenShift environment you are using already has buildah installed as a ClusterTask. A ClusterTask is installed cluster-wide by an administrator and anyone can use it in their pipelines without having to install it themselves.

Check that the buildah task is installed as a ClusterTask using the Tekton CLI.


```
tkn clustertask ls
NAME                       DESCRIPTION              AGE
buildah                    Buildah task builds...   32 weeks ago
```

Step 2: Add a Workspace to the Pipeline Task

Now you will update the pipeline.yaml file to use the new buildah task.

Open pipeline.yaml in the editor. To open the editor, click the button below.
In reading the documentation for the buildah task, you will notice that it requires a workspace named source.

Step 3: Reference the buildah Task

Now, you need to reference the new buildah task that you want to use. In the previous steps, you simply changed the name of the reference to the task. But since the buildah task is a ClusterTask, you need to add the statement kind: ClusterTask under the name so that Tekton knows to look for a ClusterTask and not a regular Task.

Your Task

Change the taskRef from echo to reference the buildah task and add a line below it with kind: ClusterTask to indicate that this is a ClusterTask.

Step 4: Update the Task Parameters

The documentation for the buildah task details several parameters but only one of them is required. You need to use the IMAGE parameter to hold the name of the image you want to build.

Since you might want to reuse this pipeline to build different images, you will make it a variable parameter that can be passed in when the pipeline runs. To do this, you need to change it here and add a parameter to the Pipeline itself.

Step 6: Apply Changes and Run the Pipeline

Start the Pipeline

When you start the pipeline, you need to pass in the build-image parameter, which is the name of the image to build.

This will be different for every learner that uses this lab. Here is the format:

image-registry.openshift-image-registry.svc:5000/$SN_ICR_NAMESPACE/tekton-lab:latest

Notice the variable $SN_ICR_NAMESPACE in the image name. This is automatically set to point to your container namespace.

Now, start the pipeline to see your new build task run. Use the Tekton CLI pipeline start command to run the pipeline, passing in the parameters repo-url, branch, and build-image using the -p option. Specify the workspace pipeline-workspace and volume claim pipelinerun-pvc using the -w option:

```
tkn pipeline start cd-pipeline \
    -p repo-url="https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git" \
    -p branch=main \
    -p build-image=image-registry.openshift-image-registry.svc:5000/$SN_ICR_NAMESPACE/tekton-lab:latest \
    -w name=pipeline-workspace,claimName=pipelinerun-pvc \
    --showlog
```


```
an_image$ tkn pipelinerun ls
NAME                    STARTED          DURATION   STATUS
cd-pipeline-run-wz5n6   13 minutes ago   3m4s       Succeeded
cd-pipeline-run-sdcqc   1 hour ago       1m27s      Succeeded
```

FOR LAB06:-

Step 1: Check for the openshift-client ClusterTask

Your pipeline currently has a placeholder for a deploy step that uses the echo task. Now it is time to replace it with a real deployment.

Knowing that you want to deploy to OpenShift, you search Tekton Hub for “openshift” and you see there is a task called openshift-client that will execute OpenShift commands on your cluster. You decide to use the openshift-client task in your pipeline to deploy your image.

Instead of installing it yourself, you first check the ClusterTasks in your cluster to see if it already exists. Luckily, the OpenShift environment you are using already has openshift-client installed as a ClusterTask. A ClusterTask is installed cluster-wide by an administrator and anyone can use it in their pipelines without having to install it themselves.

Check that the openshift-client task is installed as a ClusterTask using the Tekton CLI.

```tkn clustertask ls```


Step 2: Reference the openshift-client task

First you need to update the pipeline.yaml file to use the new openshift-client task.

Open pipeline.yaml in the editor and scroll down to the deploy pipeline task. To open the editor, click the button below.

 Open pipeline.yaml in IDE

You must now reference the new openshift-client ClusterTask that you want to use in the deploy pipeline task.

In the previous steps, you simply changed the name of the reference to the task, but since the openshift-client task is installed as a ClusterTask, you need to add the statement kind: ClusterTask under the name so that Tekton knows to look for a ClusterTask and not a regular Task.

Step 3: Update the Task Parameters

The documentation for the openshift-client task details that there is a parameter named SCRIPTthat you can use to run oc commands. Any command you can use with kubectl can also be used with oc. This is what you will use to deploy your image.

```oc create deployment {name} --image={image-name}```

Step 4: Update the Pipeline Parameters

Now that you are passing in the app-name parameter to the deploy task, you need to go back to the top of the pipeline.yaml file and add the parameter there so that it can be passed into the pipeline when it is run.


Step 6: Apply Changes and Run the Pipeline

```
kubectl apply -f pipeline.yaml
```

```
tkn pipeline start cd-pipeline \
    -p repo-url="https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git" \
    -p branch=main \
    -p app-name=hitcounter \
    -p build-image=image-registry.openshift-image-registry.svc:5000/$SN_ICR_NAMESPACE/tekton-lab:latest \
    -w name=pipeline-workspace,claimName=pipelinerun-pvc \
    --showlog
```

Step 7: Check the Deployment

```kubectl get all -l app=hitcounter```


Useful Tekton commands:-

- tkn pipelinerun ls
- tkn pipelinerun logs --last
- tkn pipelinerun logs pipelinerun_name
- tkn tasks ls
