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
draft: true
---

- toc
    - architecture overview
    - Data flow when executing the workflow

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
    - **FlyteAdmin**
- **Data Plane**: Dispatch workflow into Kubernetes pods for execution. Handle reconcile
and keep status updated to the control plane.
    - **FlytePropeller**: Kubernetes controller handling task reconciliation and invoking
    the proper FlytePlugin for task execution.
    - **FlytePlugin**: Create pod or call the proper Kuberanetes operator for task
    execution


## Workflow Execution

In short, the process of executing the workflow will be:
1. User submit a workflow through FlyteKit, FlyteConsole, or FlyteCTL.
2. FlyteAdmin validate the input and compile the workflow, then forward the workflow to
   FlytePropeller
3. FlytePropeller choose a proper FlytePlugin to execute the workflow, then keep tracking
   and reconciling to ensure the workflow finish successfully

Below we will dive in a bit more of what's happening when executing a workflow

![register-execute-workflow](img/register-execute-workflow.png "Register and Execute Workflow")
