---
title: 'From Chaos to Scale-Orchestrating AI Pipelines with Flyte'
date: 2025-08-18T11:00:19+08:00
draft: false
authors: ["naryyeh", "jiaweijiang"]
series: ["Flyte"]
tags: ["Introduction", "Data Engineer", "ML Platform"]
---

Since the term *big data* first introduced in the early 1990s, the volume of global data has grown at an exponential pace. As machine learning continues to rise, companies are building countless AI pipelines to solve real-world business problems, from predicting ad clicks to personalizing recommendations.


{{< figure src="./img/data-growth.png" width="500" caption="Figure 1. The exponential growth of global data (Credit: [1])" class="center" >}} 

At its core, an AI pipeline can be thought of as:

{{< katex >}}
$$
\text{AI Pipeline = Code + Data [2]}
$$


But in practice, building and managing these pipelines at scale is far from simple. As
companies grow, two major problems emerge: deploying across multiple clusters becomes a
nightmare, and pipelines break constantly in production.

To address these pain points, Lyft created Flyte [3], an open-source orchestration
platform designed to support tens of thousands of AI pipelines at scale.

1. **Multi-cluster deployment** - Reliably ship code to production at scale, without endless manual fixes
    - 🤕 **Without Flyte**:
        - Users need to manually setup clusters for different domains (e.g. development and production)
        - Need to take care of moving workflow between domains and clusters
        - Manual intervention required when nodes fail
    - 😄 **With Flyte**:
        - Simply execute the workflow with `--domain` argument specifying "development" or "production"
        - Flyte automatically determines where to run the code for you
        - If any node failed, Flyte will automatically transfer the workflow to other available nodes
2. **Robust pipeline** - Build resilient workflows that handle failures gracefully and ensure data integrity
    - 🤕 **Without Flyte**: 
        - Runtime type errors crash production pipelines
        - Inconsistent environments cause "works on my machine" issues
        - Difficult to track which exact code version produced results
    - 😄 **With Flyte**: 
        - Strong typing catches errors before deployment
        - Containerized tasks ensure identical environments
        - Immutable workflow versions guarantee reproducibility

## Features

Flyte tackles five specific challenges that AI teams commonly face:

1. **Scalability** - Handle massive growth in requests and data with efficient concurrency
    - Built on Kubernetes, Flyte automatically scales workflows across clusters and
    handles node failures without manual intervention.
2. **Reusability** - Reuse components across pipelines and business units, avoiding “reinventing the wheel."
    - Centralized tasks and workflows components management across teams with cache
    support to save time and compute resources
3. **Reproducibility** - Ensure experiments can be replicated by mirroring dev environments in production
    - Versions all components with packages and resources set, which guarantees
    result consistency across development and production environments
4. **Maintainability** - Operate and update thousands of pipelines with ease
    - Simple SDK interface separates workflow management from execution, allowing users
    to define workflows once in a central location and run them across multiple clusters

![multi-region routing](./img/multi_region-routing.png "Figure 2. Flyte multi-region routing setup (Credit: [4])") 

5. **Extensibility** - Integrate smoothly with third-party tools and services
    - Connects with popular data platforms like Databricks, Snowflake, and AWS Batch while
    providing centralized workflow management, monitoring, and error handling

Today, Flyte is trusted by leading companies such as Tesla, Spotify, LinkedIn, and Toyota, and runs over 30 million tasks per day. For example, in a recent LinkedIn Engineering blog [4], the team unveiled their next-generation AI pipelines powered by Flyte, achieving 20–30× faster training and launch times, while enabling them to train models 200× larger than before.


## How to Run a Workflow in Flyte?

The Flyte workflow can be defined through simple Python syntax that is similar to other
workflow orchastration tools.

```python
import flytekit as fl


@fl.task
def say_hello(name: str) -> str:
    return f"Hello, {name}!"


@fl.workflow
def wf(name: str = "world") -> str:
    greeting = say_hello(name=name)
    return greeting
```


After defining the workflow, we can either executing it locally by `pyflyte run task.py
wf` for debugging locally or running on remote with `pyflyte run --remote task.py wf`.
When running on remote, Flyte automatically creates Kubernetes pods to execute tasks and
provides real-time status updates.

The command prints a URL to access Flyte Console, a web dashboard for monitoring workflow
progress, viewing logs, and managing operations like retries.

```sh
❯ pyflyte run --remote hello_world.py wf
Running Execution on Remote.

[✔] Go to http://localhost:30080/console/projects/flytesnacks/domains/development/executions/a465495htfhprw5fhp2w to see execution in the console.
```

![flyte-console](./img/flyte-console.png "Figure 3. Flyte console") 


## References
[1] [Amount of Data Created Daily (2025)](https://explodingtopics.com/blog/data-generated-per-day) <br>
[2] [A Chat with Andrew on MLOps: From Model-centric to Data-centric AI](https://www.youtube.com/watch?v=06-AZXmwHjo) <br>
[3] [Flyte OSS](https://www.union.ai/docs/v1/flyte/user-guide/) <br>
[4] [OpenConnect: LinkedIn’s next-generation AI pipeline ecosystem](https://www.linkedin.com/blog/engineering/infrastructure/openconnect-linkedins-next-generation-ai-pipeline-ecosystem) <br>
[5] [On Orchestrators: You Are All Right, But You Are All Wrong Too](https://dlthub.com/blog/on-orchestrators) <br>
