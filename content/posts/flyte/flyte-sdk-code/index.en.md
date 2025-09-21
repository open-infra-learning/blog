---
title: "Flyte v2 SDK Code Deepdive"
summary: "Dive into some components in flyte v2 and the structure of flyte sdk"
description: ""
date: 2025-09-21T12:46:25+08:00
authors: ["naryyeh", "jiaweijiang"]
slug: "flyte-sdk-code"
tags: ["Data Engineer", "ML Platform"]
# series: ["Documentation"]
# categories: ["Introduction"]
# series_order: 1
cascade:
  showSummary: true
  hideFeatureImage: false
draft: false
---

Focus on execution remote / locally
- local / remote execution
- image caching
- task caching (remote) / local is implementing


Main folder component in the code

Code path for local

- TOC
    - main component
    - local
        - Submit task
        - execute task
    - remote
        - Submit task
        - execute task

This is the learning note of developing flyte v2 sdk. The back-end is not released yet, so
we will use a more general way to describe the component that the sdk connects to in this
article.


## Terms

Let's start with define some terms in flyte v2.

- Task: Individual unit of computation we defined in script. This is the functions that are decorated with `@task`.
- Action: The execution of a task is an action. The difference is task is the static definition, and action is
  the things that's being executed.
- Run: As we can define nested tasks in flyte (task 0 calls task1), we regard the top action (the one without
parent) as run.


## Main Components

Main code lie under `flyte-sdk/src/flyte/` directory. The important files that we will
cover today are:

- `_task.py`: Define the `TaskTemplate`. The class contains all things needed for defining
  a task
- `bin/_runtime.py`: Entrypoint when running remotely. The function `main` in this file is decorated with the
  click options (command `a0 --inputs ... --outputs-path ...`). When configuring the flyte task pod when
running in the k8s, this command will be put intot the command field in the container inside the pod. This
command will then be executed when the container starts. 
- `_internal/runtime/entrypoints.py`: Defines the entry functions for executing task locally or remotely.
- `_run.py`: The runner for running the task. Runner is responsible for submitting the run.


We will focus on 3 main part in flyte-sdk in this article:
- execution
- task caching
- image caching

Execution is the biggest and the major part we will be talking.


## Execution

### Remote Execution


When we decorate a function with `@task`, a `AsyncFunctionTaskTemplate` is created with the function we
defined recorded in it.


In remote execution, there's three objects that are involved: TaskTemplate, Controller, Informer. Same as in
flyte v1, a task runs inside a pod.

At first, sdk will download and unpack the code. The action and run name will be passed as the args, so that
during the unpacking process, code will know which function we are unpacking. The `_download_and_load_task()`
function will unpack the `TaskTemplate` out. Then we call the `convert_and_run()`, which leads to
`task.execute()`, and we goes into `AsyncFunctionTaskTemplate.execute()`.

In the `AsyncFunctionTaskTemplate.execute()`, it will execute the original function with `self.func()`. During
thie funciton execution, if it encouter the task function, it will trigger `TaskTemplate.__call__()` for that
function and continue on submitting the task to the queue service. For example, during the execution of
`main()` function, if it tries to execute `await say_hello(...)`, as `say_hello` here is the task template,
this will trigger `TaskTempalte.__call__()`.


```python
@env.task
async def say_hello(data: str, lt: List[int]) -> str:
    print(f"Hello, world! - {flyte.ctx().action}")
    return f"Hello {data} {lt}"

@env.task
async def main(data: str = "default string", n: int = 3) -> str:
    print(f"Hello, nested! - {flyte.ctx().action}")

    return await say_hello(data=data, lt=vals)
```

