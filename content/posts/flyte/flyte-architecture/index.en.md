---
title: "Flyte Architecture: How workflow runs"
summary: "Components in Flyte"
description: ""
date: 2025-08-23T20:09:42+08:00
authors: ["naryyeh"]
slug: "flyte-architecture"
tags: ["Data Engineer", "ML Platform"]
series: ["How to Flyte"]
# categories: ["Introduction"]
series_order: 2
cascade:
  showSummary: true
  hideFeatureImage: false
draft: false
---

This article will focus on introducing core components in Flyte, their job, and how they
interact with each other to form this awesome platform.

## Overview

This part is the simplified version of Flyte architecture. For full details, please refer
to Flyte official document: https://www.union.ai/docs/v1/flyte/architecture/component-architecture/

![flyte-architecture-overview-simple](img/flyte-architecture-overview-simple.png "Simple Flyte
Architecture Overview")

Flyte can be separated into 3 logical planes, namely user, control, and data plane. Below
are the brief description of components within each plane.

- **User Plane**: Interaction interface between platform and user
    - **Flytekit**: Python SDK
    - **FLyteConsole**: UI interface
    - **FlyteCTL**: CLI tool
- **Control Plane**: Transport the request from user to data plane. Record the workflow
status and handle scheduling.
    - **FlyteAdmin**: Validate and compile the workflow to executable for FlytePropeller
    to execute it
- **Data Plane**: Dispatch workflow into Kubernetes pods for execution. Handle reconcile
and keep status updated to the control plane.
    - **FlytePropeller**: Kubernetes controller handling task reconciliation and invoking
    the proper FlytePlugin for task execution.
    - **FlytePlugin**: Create pod or call the proper Kuberanetes operator for task
    execution


In short, the process of executing the workflow will be:
1. User submit a workflow through FlyteKit, FlyteConsole, or FlyteCTL.
2. FlyteAdmin validate the input and compile the workflow, then forward the workflow to
   FlytePropeller
3. FlytePropeller choose a proper FlytePlugin to execute the workflow, then keep tracking
   and reconciling to ensure the workflow finish successfully


## Workflow Execution

Below we will dive in a bit more of what each component do when executing a workflow


{{< alert "circle-info">}}
Flyte workflow have 3 main components: launch plane, workflow, and task.
- **launch plan**: Templates to define input for the workflow
- **workflow**: Grouping tasks to form a whole pipeline
- **task**: A step or a unit of computation (e.g. function that do data transform, model
training, etc.)

These are the brief description for those components, and we will have more articles
depicting more details on them and their variations.
{{< /alert >}}

![register-execute-workflow](img/register-execute-workflow.png "Register and Execute Workflow")

1. Client (User Plane) send request `getLaunchPlan` to get the launch plan from FlyteAdmin
    - If user does not set the launch plan explicitly, the default lauch plan will be
    created, which is as same name as the workflow
2. `FlyteAdmin` return the requested launch plan
3. Client validate if all inputs are provided based on the launch plan returned by
   `FlyteAdmin`
4. Workflow execution request is sent to `FlyteAdmin` from the client
5. `FlyteAdmin` validates the input and compile workflow and tasks
6. Upload the compiled task and workflow to Flyte metadata storage. If the workflow is
   compiled before, fetch from the metadata storage.
7. Translate compiled workflow to an executable format with inputs. The executable format
   here is the `flyteworkflow` CR(custom resource)
    - CR is the custom resources that can be
run on k8s with customized behaviour.
8. `FlytePropeller` get the `flyteworkflow` CR and try to execute it by invoking `FlytePlugin`,
   and keep tracking to ensure the workflow success
9. During the workflow execution, `FlytePropeller` will keep `FlyteAdmin` updated about the
   status for workflows

This is the brief steps heppens when executing a workflow, and how each component working
together.


## Summary

Executing workflow in Flyte requires the coordination between client, `FlyteAdmin`, and
`FlytePropeller`. To summarize:

- Client (`FlyteKit`, `FlyteConsole`, `FlyteCLI`): SDK/Tools for user manipulate Flyte
- `FlyteAdmin`: Entry point for workflow execution.
- `FlytePropeller`: Handle actual workflow execution
