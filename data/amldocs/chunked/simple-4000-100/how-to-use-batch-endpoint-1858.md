
# [Python](#tab/python)

```python
job = ml_client.batch_endpoints.invoke(
    endpoint_name=endpoint_name,
    inputs=Input(path="https://pipelinedata.blob.core.windows.net/sampledata/mnist", type=AssetTypes.URI_FOLDER)
)
```

# [Studio](#tab/azure-studio)

1. Navigate to the __Endpoints__ tab on the side menu.
1. Select the tab __Batch endpoints__.
1. Select the batch endpoint you just created.
1. Select __Create job__.

    :::image type="content" source="./media/how-to-use-batch-endpoints-studio/create-batch-job.png" alt-text="Screenshot of the create job option to start batch scoring.":::

1. On __Deployment__, select the deployment you want to execute.

    :::image type="content" source="./media/how-to-use-batch-endpoints-studio/job-setting-batch-scoring.png" alt-text="Screenshot of using the deployment to submit a batch job.":::

1. Select __Next__.
1. On __Select data source__, select the data input you want to use. For this example, select __Datastore__ and in the section __Path__ enter the full URL `https://pipelinedata.blob.core.windows.net/sampledata/mnist`. Notice that this only works because the given path has public access enabled. In general, you'll need to register the data source as a __Datastore__. See [Accessing data from batch endpoints jobs](how-to-access-data-batch-endpoints-jobs.md) for details.

    :::image type="content" source="./media/how-to-use-batch-endpoints-studio/select-datastore-job.png" alt-text="Screenshot of selecting datastore as an input option.":::

1. Start the job.


### Configure job's inputs

Batch endpoints support reading files or folders that are located in different locations. To learn more about how the supported types and how to specify them read [Accessing data from batch endpoints jobs](how-to-access-data-batch-endpoints-jobs.md).

> [!TIP]
> Local data folders/files can be used when executing batch endpoints from the Azure ML CLI or Azure ML SDK for Python. However, that operation will result in the local data to be uploaded to the default Azure Machine Learning Data Store of the workspace you are working on.

> [!IMPORTANT]
> __Deprecation notice__: Datasets of type `FileDataset` (V1) are deprecated and will be retired in the future. Existing batch endpoints relying on this functionality will continue to work but batch endpoints created with GA CLIv2 (2.4.0 and newer) or GA REST API (2022-05-01 and newer) will not support V1 dataset.

### Configure the output location

The batch scoring results are by default stored in the workspace's default blob store within a folder named by job name (a system-generated GUID). You can configure where to store the scoring outputs when you invoke the batch endpoint.

# [Azure CLI](#tab/azure-cli)
    
Use `output-path` to configure any folder in an Azure Machine Learning registered datastore. The syntax for the `--output-path` is the same as `--input` when you're specifying a folder, that is, `azureml://datastores/<datastore-name>/paths/<path-on-datastore>/`. Use `--set output_file_name=<your-file-name>` to configure a new output file name.

```azurecli
az --version

set -e

# <set_variables>
export ENDPOINT_NAME="<YOUR_ENDPOINT_NAME>"
export DEPLOYMENT_NAME="<YOUR_DEPLOYMENT_NAME>"
# </set_variables>

export ENDPOINT_NAME=endpt-`echo $RANDOM`
export DEPLOYMENT_NAME="mnist-torch-dpl"

echo "Creating compute"
# <create_compute>
az ml compute create -n batch-cluster --type amlcompute --min-instances 0 --max-instances 5
# </create_compute>

echo "Creating batch endpoint $ENDPOINT_NAME"
# <create_batch_endpoint>
az ml batch-endpoint create --name $ENDPOINT_NAME
# </create_batch_endpoint>

echo "Creating batch deployment nonmlflowdp for endpoint $ENDPOINT_NAME"
# <create_batch_deployment_set_default>
az ml batch-deployment create --file endpoints/batch/mnist-torch-deployment.yml --endpoint-name $ENDPOINT_NAME --set-default
# </create_batch_deployment_set_default>

echo "Showing details of the batch endpoint"
# <check_batch_endpooint_detail>
az ml batch-endpoint show --name $ENDPOINT_NAME
# </check_batch_endpooint_detail>

echo "Showing details of the batch deployment"
# <check_batch_deployment_detail>
az ml batch-deployment show --name $DEPLOYMENT_NAME --endpoint-name $ENDPOINT_NAME
# </check_batch_deployment_detail>

echo "Invoking batch endpoint with public URI (MNIST)"
# <start_batch_scoring_job>
JOB_NAME=$(az ml batch-endpoint invoke --name $ENDPOINT_NAME --input https://pipelinedata.blob.core.windows.net/sampledata/mnist --input-type uri_folder --query name -o tsv)
# </start_batch_scoring_job>

echo "Showing job detail"
# <show_job_in_studio>
az ml job show -n $JOB_NAME --web
# </show_job_in_studio>

echo "Stream job logs to console"
# <stream_job_logs_to_console>
az ml job stream -n $JOB_NAME
# </stream_job_logs_to_console>

# <check_job_status>
STATUS=$(az ml job show -n $JOB_NAME --query status -o tsv)
echo $STATUS
if [[ $STATUS == "Completed" ]]
then
  echo "Job completed"
elif [[ $STATUS ==  "Failed" ]]
then
  echo "Job failed"
  exit 1
else 
  echo "Job status not failed or completed"
  exit 2
fi
# </check_job_status>

echo "Invoke batch endpoint with specific output file name"
# <start_batch_scoring_job_configure_output_settings>
export OUTPUT_FILE_NAME=predictions_`echo $RANDOM`.csv
JOB_NAME=$(az ml batch-endpoint invoke --name $ENDPOINT_NAME --input https://pipelinedata.blob.core.windows.net/sampledata/mnist --input-type uri_folder --output-path azureml://datastores/workspaceblobstore/paths/$ENDPOINT_NAME --set output_file_name=$OUTPUT_FILE_NAME --query name -o tsv)
# </start_batch_scoring_job_configure_output_settings>

echo "Invoke batch endpoint with specific overwrites"
# <start_batch_scoring_job_overwrite>
export OUTPUT_FILE_NAME=predictions_`echo $RANDOM`.csv
JOB_NAME=$(az ml batch-endpoint invoke --name $ENDPOINT_NAME --input https://pipelinedata.blob.core.windows.net/sampledata/mnist --input-type uri_folder --mini-batch-size 20 --instance-count 5 --query name -o tsv)
# </start_batch_scoring_job_overwrite>

echo "Stream job detail"
# <stream_job_logs_to_console>
az ml job stream -n $JOB_NAME
# </stream_job_logs_to_console>

# <check_job_status>
STATUS=$(az ml job show -n $JOB_NAME --query status -o tsv)
echo $STATUS
if [[ $STATUS == "Completed" ]]
then
  echo "Job completed"
elif [[ $STATUS ==  "Failed" ]]
then
  echo "Job failed"
  exit 1
else 
  echo "Job status not failed or completed"
  exit 2
fi
# </check_job_status>

echo "List all jobs under the batch deployment"
# <list_all_jobs>
az ml batch-deployment list-jobs --name $DEPLOYMENT_NAME --endpoint-name $ENDPOINT_NAME --query [].name
# </list_all_jobs>

echo "Create a new batch deployment (mnist-keras-dpl), not setting it as default this time"
# <create_new_deployment_not_default>
az ml batch-deployment create --file endpoints/batch/mnist-keras-deployment.yml --endpoint-name $ENDPOINT_NAME
# </create_new_deployment_not_default>

echo "Invoke batch endpoint with public data"
# <test_new_deployment>
DEPLOYMENT_NAME="mnist-keras-dpl"
JOB_NAME=$(az ml batch-endpoint invoke --name $ENDPOINT_NAME --deployment-name $DEPLOYMENT_NAME --input https://pipelinedata.blob.core.windows.net/sampledata/mnist --input-type uri_folder --query name -o tsv)
# </test_new_deployment>

echo "Show job detail"
# <show_job_in_studio>
az ml job show -n $JOB_NAME --web
# </show_job_in_studio>

echo "Stream job logs to console"
# <stream_job_logs_to_console>
az ml job stream -n $JOB_NAME
# </stream_job_logs_to_console>

# <check_job_status>
STATUS=$(az ml job show -n $JOB_NAME --query status -o tsv)
echo $STATUS
if [[ $STATUS == "Completed" ]]
then
  echo "Job completed"
elif [[ $STATUS ==  "Failed" ]]
then
  echo "Job failed"
  exit 1
else 
  echo "Job status not failed or completed"
  exit 2
fi
# </check_job_status>

echo "Update the batch deployment as default for the endpoint"
# <update_default_deployment>
az ml batch-endpoint update --name $ENDPOINT_NAME --set defaults.deployment_name=$DEPLOYMENT_NAME
# </update_default_deployment>

echo "Verify default deployment. In this example, it should be mlflowdp."
# <verify_default_deployment>
az ml batch-endpoint show --name $ENDPOINT_NAME --query "{Name:name, Defaults:defaults}"
# </verify_default_deployment>

echo "Invoke batch endpoint with the new default deployment with public URI"
# <test_new_default_deployment>
JOB_NAME=$(az ml batch-endpoint invoke --name $ENDPOINT_NAME --input https://pipelinedata.blob.core.windows.net/sampledata/mnist --input-type uri_folder --query name -o tsv)
# </test_new_default_deployment>

echo "Stream job logs to console"
# <stream_job_logs_to_console>
az ml job stream -n $JOB_NAME
# </stream_job_logs_to_console>

# <check_job_status>
STATUS=$(az ml job show -n $JOB_NAME --query status -o tsv)
echo $STATUS
if [[ $STATUS == "Completed" ]]
then
  echo "Job completed"
elif [[ $STATUS ==  "Failed" ]]
then
  echo "Job failed"
  exit 1
else 
  echo "Job status not failed or completed"
  exit 2
fi
# </check_job_status>

echo "Get Scoring URI"
# <get_scoring_uri>
SCORING_URI=$(az ml batch-endpoint show --name $ENDPOINT_NAME --query scoring_uri -o tsv)
# </get_scoring_uri>

echo "Get Token"
# <get_token>
AUTH_TOKEN=$(az account get-access-token --resource https://ml.azure.com --query accessToken -o tsv)
# </get_token>

echo "Invoke batch endpoint with REST API call"
# <start_batch_scoring_job_rest>
RESPONSE=$(curl --location --request POST "$SCORING_URI" \
--header "Authorization: Bearer $AUTH_TOKEN" \
--header "Content-Type: application/json" \
--data-raw "{
  \"properties\": {
    \"dataset\": {
      \"dataInputType\": \"DataUrl\",
      \"Path\": \"https://pipelinedata.blob.core.windows.net/sampledata/mnist\"
    }
  }
}")
# </start_batch_scoring_job_rest>

# <check_job_status_rest>
# define how to wait  
wait_for_completion () {
    operation_id=$1
    access_token=$2
    status="unknown"

    while [[ $status != "Completed" && $status != "Succeeded" && $status != "Failed" && $status != "Canceled" ]]
    do
        echo "Getting operation status from: $operation_id"
        operation_result=$(curl --location --request GET $operation_id --header "Authorization: Bearer $access_token")
        # TODO error handling here
        status=$(echo $operation_result | jq -r '.status')
        if [[ -z $status || $status == "null" ]]
        then
            status=$(echo $operation_result | jq -r '.properties.status')
        fi

        # Fail early if job submission failed and there is nothing to poll on
        if [[ -z $status || $status == "null" ]]
        then
            echo "No status found on operation, setting to failed."
            status="Failed"
        fi

        echo "Current operation status: $status"
        sleep 10
    done

    if [[ $status == "Failed" ]]
    then
        error=$(echo $operation_result | jq -r '.error')
        echo "Error: $error"
    fi
}

# get job from invoke response and wait for completion
JOB_ID=$(echo $RESPONSE | jq -r '.id')
JOB_ID_SUFFIX=$(echo ${JOB_ID##/*/})
wait_for_completion $SCORING_URI/$JOB_ID_SUFFIX $AUTH_TOKEN
# </check_job_status_rest>

# <delete_deployment>
az ml batch-deployment delete --name nonmlflowdp --endpoint-name $ENDPOINT_NAME --yes
# </delete_deployment>

# <delete_endpoint>
az ml batch-endpoint delete --name $ENDPOINT_NAME --yes
# </delete_endpoint>

```
