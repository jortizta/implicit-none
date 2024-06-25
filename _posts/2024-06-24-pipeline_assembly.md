---
title: "Pipeline Assembly"
date: 2024-00-00
author: Joshua Thompson
---

# Pipeline Assembly
In this article, we will explain how to effectively orchestrate kubeflow pipelines. 

## Table of Contents
- [Defining a Pipeline](#defining-a-pipeline)
	* [Pipeline Decorators](#pipeline-decorators)
    * [Pipeline Signatures](#pipeline-signatures)
    * [Data & Task Dependencies](#data-and-task-dependencies)
    * [Control Flow](#control-flow)
    * [Task Configurations](#task-configurations)


## Defining a Pipeline
For workflows with custom components, this orchestration is best managed with a pipeline.py file. Here, we can import all of the required components and describe the execution workflow. 
To create our pipeline, we first need to load all of its dependent components. As shown in the previous article, these components can be defined inline or with a separate file. 
To load a component stored in a local file, we’ll use the `load_component_from_file()` function. However, components can be loaded from remote urls using the `load_component_from_url()` method as well.
With the components loaded, we can now define the pipeline itself. 
All kubeflow pipeline definitions have 5 parts:
* Pipeline decorator
* Pipeline signature
* Data/task dependencies
* Control flow
* Task configurations

### Pipeline Decorators
A pipeline decorator is our first requirement. By prefacing any function with the `@dsl.pipeline decorator`, kubeflow will recognize this function as a pipeline definition that uses the KFP domain-specific language (DSL).
Notably, the decorator has 4 optional arguments:
name: This is the name of your pipeline. By default, it is set to a modified version of the function name.
description: This is an arbitrary, user-defined description of the pipeline.
pipeline_root: This sets the remote storage location where pipeline generated outputs will be stored at runtime.
display_name: This is an arbitrary human-readable name for your pipeline.

### Pipeline Signatures
With the decorator defined, we can now specify the pipeline’s signature. Here we can mark any parameters or input artifacts that the pipeline consumes as well as any output artifacts that are produced. However, pipeline outputs are specified as the function’s returnables and not as input parameters. Note that artifacts are also annotated with their corresponding kfp.dsl data type.

### Data and Task Dependencies
With the signature defined, we can now orchestrate the pipeline itself. This is done by defining inter-task dependencies which are created by passing the outputs from one component to another. By calling a given component function, it will return a PipelineTask object with the fields to access these outputs (`output` and `outputs` respectively if one or more values are returned). By feeding these outputs directly into another component function call, we define a task dependency between the components and implicitly modify the execution order for the pipeline. We can also explicitly define the execution order by calling .after(EX_PIPELINE_TASK) on a particular component and any PipelineTasks that do not share any dependencies will just run in parallel. 

#### Special input types
You can also define what metadata is fed directly into components at this stage. This is done by passing special attributes of the kfp.dsl class:

```
dsl.PIPELINE_JOB_NAME_PLACEHOLDER
dsl.PIPELINE_JOB_RESOURCE_NAME_PLACEHOLDER
dsl.PIPELINE_JOB_ID_PLACEHOLDER
dsl.PIPELINE_TASK_NAME_PLACEHOLDER
dsl.PIPELINE_TASK_ID_PLACEHOLDER
dsl.PIPELINE_JOB_CREATE_TIME_UTC_PLACEHOLDER
dsl.PIPELINE_JOB_SCHEDULE_TIME_UTC_PLACEHOLDER
dsl.PIPELINE_ROOT_PLACEHOLDER
```
These values should be consumed by the components as strings and are replaced at runtime with the job-specific values.

### Control Flow
When workflows require more complex execution orders than standard task dependencies allow, we can leverage KFP Conditions, Loops, and Exit Handling.
Conditions: dsl.If, dsl.Elif, and dsl.Else enable the conditional execution of tasks based on conditions within a given scope.
Loops: dsl.ParallelFor allows for parallel executions of the same task over a given set of data.
Exit Handling: The dsl.ExitHandler functions similar to a `try` and `finally` block. KFP allows pipelines to define an exit task that runs after everything a context manager’s scope has completed (successfully or not).
For more details on Control Flow, see your dedicated article on Control Flow.

### Task Configurations
Before compiling this pipeline, we also have the ability to specify PipelineTask specific options through clean function calls. These methods allow users to configure environment variables, machine types, caching options and more through single-line function calls on the PipelineTasks themselves.
```
.add_accelerator_type()
.set_accelerator_limit()
.set_cpu_limit()
.set_memory_limit()
.set_env_variable()
.set_caching_options()
.set_display_name()
.set_retry()
.ignore_upstream_failure()
```


