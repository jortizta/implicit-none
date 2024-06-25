---
title: "Pipeline Compilation & Submission"
date: 2024-00-00
author: Joshua Thompson
---

# Pipeline Compilation & Submission
Following the previous article that demonstrated how to build a custom pipeline, we will now show how to compile and submit this pipeline for execution.

## Table of Contents
- [Compiling a Pipeline](#compiling-a-pipeline)
- [Submitting a Pipeline](#submitting-a-pipeline)


## Compiling a Pipeline
```
compiler.Compiler().compile(my_pipeline, package_path='pipeline.yaml')
```

Before we can submit a pipeline to run on a cloud service, we need to generate a cloud agnostic representation of our pipeline.
To do this, we compile our pipeline definition into a protocol buffer (protobuf) message. Protobuf messages are serialized representations of arbitrary data structures and are widely used in industry for developing programs that communicate with each other over a network or for  storing data.
Using the kfp.compiler class, we can create a pipeline’s protobuf message in a single line. The above command generates a kfp PipelineSpec protobuf message and saves it to a local YAML file. 

## Submitting a Pipeline
With our compiled pipeline, we can now submit it to run.

```
my_run = PipelineJob(
display_name=pipeilne_name,
template_path='pipeline.yaml',
job_id=job_id,
parameter_values=pipeline_inputs,
enable_caching=enable_caching,
project=project_id,
location=region,
)
my_run.submit(service_account=service_account)
```

On Google Cloud Platform, this is done using the `google-cloud-aiplatform` api. Specifically, we will create a PipelineJob object similar to the one above. The `pipeline_yaml_path` will point to our local pipeline yaml we just created. Note, that gcp will further compile this PipelineSpec into a final pipeline definition before running.

Notably, we could also run this pipeline locally without compiling by using a kfp’s local and DockerRunner classes. This can be useful for rapid local development, but currently only has limited functionality (does not support caching, retries, Conditions, ParallelFor, or ExitHandlers).
