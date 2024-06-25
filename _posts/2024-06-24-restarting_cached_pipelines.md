---
title: "Caching & Restarting Pipelines"
date: 2024-00-00
author: Joshua Thompson
---

# Caching & Restarting Pipelines
After reviewing pipeline submissions, we can understand how to restart halted pipelines and leverage their cached results.

## Table of Contents
- [How to Cache Pipelines](#how-to-cache-pipelines)
- [Restarting with Vertex AI’s GUI](#restarting-with-vertex-ai’s-gui)
- [Restarting with a Compiled Pipeline File](#restarting-with-a-compiled-pipeline-file)
- [Unexpected Caching Behaviors](#unexpected-caching-behaviors)

## How to Cache Pipelines
The ability to restart and cache pipelines is useful in a variety of scenarios:
* You may need to evaluate component changes within a pipeline but not wish to recompute prior components.
* Someone may need to check the quality of a pipeline’s intermediate results before processing continues.
* A modeler may want to modify hyperparameters for training without rerunning some expensive preprocessing.\

In order to leverage components’ cached results, there are several rules to keep in mind:
* The component inputs need to be the same.
    - “Component Inputs” refers to any input parameter values or input artifact ids.
* The component output definition has to be the same.
    - “Component output definition” refers to any output parameter (output_1: str) or artifact definition (output_2: Artifact)
* The component’s specification has to be the same.
    - “Component specification” refers to the image, commands, arguments, and environment variables specified for a component’s execution. These parameters are normally set in a component’s and pipeline’s configuration yaml files (see ___ article for more detail).
* The pipelines have to have the same name.
    - “Name” refers to the `google.cloud.aiplatform.PipelineJob.display_name` that is consistent between pipelines of the same type. Notably, the display_name is distinct from a pipeline’s “Run name” or “job_id”.
    - display_name: A display_name is the user-defined string that identifies pipelines of a particular category. Pipeline runs of the same category can share the same display name to better organize and filter runs for future reference. `display_name` is specified at pipeline submission.
    - Run name & job_id: Run name & job_id both refer to the user-defined string that uniquely identifies a pipeline run within a region. This identifier is useful for filtering PipelineJob objects. `job_id` is specified at pipeline submission.

With these conditions in mind, we can now restart a given pipeline. There are 2 methods for doing so:
Restarting through the Vertex AI interface
Re-submitting a pipeline using the original compiled pipeline file

## Restarting with Vertex AI’s GUI
You can restart a pipeline run through Vertex AI’s user-interface by “cloning” a pipeline. Cloning gives you the option to modify any of your input parameters via the UI prior to submitting the original pipeline. However, this approach can prove difficult when one needs to modify a large number of parameters or nested dictionaries. Additionally, cloning does not provide you the ability to modify what code is running (i.e. the component images). For these reasons, we predominantly rely on method 2 instead.

## Restarting with a Compiled Pipeline File
If you have the original compiled pipeline yaml file, then you can directly pass that to the submission function call to resubmit a pipeline. However, when this is not the case, you can also recreate the file using the `aiplatform` API.
`aiplatform.PipelineJob.list(filter=in_context("projects/PROJECT_NUMBER/locations/LOCATION/PipelineJobs/PIPELINE_JOB_ID")`
Using the above `aiplatform.PipelineJob.list()` method,  we can retrieve the target PipelineJob object. Notably, in this example, the pipeline’s job_id is used with an in_context filter to collect the PipelineJob.
With the PipelineJob object, we can reconstruct a local copy of the configuration file that was used to submit the original pipeline. With this file, we can also more easily modify the original pipeline hyperparameters in the same fashion as the original submission.
Once we are finished updating the configuration, we can resubmit the pipeline as we would any other. Simply point to the local pipeline.json and config.yaml and set the `enable_caching` parameter to true. Notably, the original pipeline needs to have this parameter specified as well. In this way, any components that satisfy the four criteria above will not be recomputed. Their previous executions and results will be pulled directly into the current pipeline.
`python run_pipeline.py -p PATH_TO_PIPELINE -c PIPELINE_JOB_ID -b NAME_OF_COMPONENT_1 NAME_OF_COMPONENT_2`
Alternatively, if you need to test a local change in some remote PipelineJob, you can build your local files as a new component container. Then, within the downloaded pipeline json file, you can insert that new container path as the one your component should use. This enables faster development when local testing is not an option. We recommend automating this logic as an optional build flag similar to the command above.

With these options, Data Scientists and Machine Learning Engineers have plenty of flexibility to pause and restart pipelines, test development changes, and experiment with hyperparameters without incurring needless cloud cost.

## Unexpected Caching Behaviors
When restarting a pipeline, it is important to consider how caching requirements will apply to all of your components. Without proper precaution, it is possible to unintentionally cache components that were supposed to be recomputed. 
This can occur in the following scenarios:
Collecting multiple outputs from a ParallelFor loop
Importing an artifact containing different content, but the same uri (replacing a file)
References:
https://cloud.google.com/vertex-ai/docs/pipelines/configure-caching




