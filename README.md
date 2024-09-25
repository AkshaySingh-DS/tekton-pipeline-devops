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

Useful Tekton commands:-

- tkn pipelinerun ls
- tkn pipelinerun logs --last
- tkn pipelinerun logs pipelinerun_name
- tkn tasks ls