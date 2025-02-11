This Python code snippet shows use of a managed identity, together with the creation of an Azure Machine Learning pipeline job. Additionally, it shows use of a Spark component and an Azure Machine Learning Managed (Automatic) Synapse compute:

```python
from azure.ai.ml import MLClient, dsl, spark, Input, Output
from azure.identity import DefaultAzureCredential
from azure.ai.ml.entities import ManagedIdentityConfiguration
from azure.ai.ml.constants import InputOutputModes

subscription_id = "<SUBSCRIPTION_ID>"
resource_group = "<RESOURCE_GROUP>"
workspace_name = "<AML_WORKSPACE_NAME>"
ml_client = MLClient(
    DefaultAzureCredential(), subscription_id, resource_group, workspace_name
)

spark_component = spark(
    name="Spark Component",
    inputs={
        "titanic_data": Input(type="uri_file", mode="direct"),
    },
    outputs={
        "wrangled_data": Output(type="uri_folder", mode="direct"),
    },
    # The source folder of the component
    code="./src",
    entry={"file": "titanic.py"},
    driver_cores=1,
    driver_memory="2g",
    executor_cores=2,
    executor_memory="2g",
    executor_instances=2,
    args="--titanic_data ${{inputs.titanic_data}} --wrangled_data ${{outputs.wrangled_data}}",
)


@dsl.pipeline(
    description="Sample Pipeline with Spark component",
)
def spark_pipeline(spark_input_data):
    spark_step = spark_component(titanic_data=spark_input_data)
    spark_step.inputs.titanic_data.mode = InputOutputModes.DIRECT
    spark_step.outputs.wrangled_data = Output(
        type="uri_folder",
        path="azureml://datastores/workspaceblobstore/paths/data/wrangled/",
    )
    spark_step.outputs.wrangled_data.mode = InputOutputModes.DIRECT
    spark_step.identity = ManagedIdentityConfiguration()
    spark_step.resources = {
        "instance_type": "Standard_E8S_V3",
        "runtime_version": "3.2.0",
    }

pipeline = spark_pipeline(
    spark_input_data=Input(
        type="uri_file",
        path="azureml://datastores/workspaceblobstore/paths/data/titanic.csv",
    )
)

pipeline_job = ml_client.jobs.create_or_update(
    pipeline,
    experiment_name="Titanic-Spark-Pipeline-SDK",
)

# Wait until the job completes
ml_client.jobs.stream(pipeline_job.name)
```

> [!NOTE]
> To use an attached Synapse Spark pool, define the `compute` parameter in the `azure.ai.ml.spark` function, instead of `resources` parameter. For example, in the code sample shown above, define `spark_step.compute = "<ATTACHED_SPARK_POOL_NAME>"` instead of defining `spark_step.resources`.

# [Studio UI](#tab/ui)
This functionality isn't available in the Studio UI. The Studio UI doesn't support this feature.


## Next steps
- [Code samples for Spark jobs using Azure Machine Learning CLI](https://github.com/Azure/azureml-examples/tree/main/cli/jobs/spark)
- [Code samples for Spark jobs using Azure Machine Learning Python SDK](https://github.com/Azure/azureml-examples/tree/main/sdk/python/jobs/spark)
