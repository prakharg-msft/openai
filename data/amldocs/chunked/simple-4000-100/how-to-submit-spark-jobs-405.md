    1. Select an attached Synapse Spark pool from the **Select Azure ML attached compute** menu.
1. Select **Next**.
1. On the **Environment** screen:
    1. Select one of the available environments from the list. Environment selection is optional.
    1. Select **Next**.
1. On **Job settings** screen:
    1. Provide a job **Name**. You can use the job **Name**, which is generated by default.
    1. Select **Experiment name** from the dropdown menu.
    1. Under **Add tags**, provide **Name** and **Value**, then select **Add**. Adding tags is optional.
    1. Under the **Code** section:
        1. Select an option from **Choose code location** dropdown. Choose **Upload local file** or **Azure Machine Learning workspace default blob storage**.
        1. If you selected **Choose code location**:
            - Select **Browse**, and navigate to the location containing the code file(s) on your local machine.
        1. If you selected **Azure Machine Learning workspace default blob storage**:
            1. Under **Path to code file to upload**, select **Browse**.
            1. In the pop-up screen titled **Path selection**, select the path of code files on the workspace default blob storage.
            1. Select **Save**.
        1. Input the name of **Entry file** for the standalone job. This file should contain the Python code that takes arguments.
        1. To add any another Python file(s) required by the standalone job at runtime, select **+ Add file** under **Py files** and input the name of the `.zip`, `.egg`, or `.py` file to be placed in the `PYTHONPATH` for successful execution of the job. Multiple files can be added.
        1. To add any Jar file(s) required by the standalone job at runtime, select **+ Add file** under **Jars** and input the name of the `.jar` file to be included in the Spark driver and the executor `CLASSPATH` for successful execution of the job. Multiple files can be added.
        1. To add archive(s) that should be extracted into the working directory of each executor for successful execution of the job, select **+ Add file** under **Archives** and input the name of the archive. Multiple archives can be added.
        1. Adding **Py files**, **Jars**, and **Archives** is optional.
        1. To add an input, select **+ Add input** under **Inputs** and
            1. Enter an **Input name**. The input should refer to this name later in the **Arguments**.
            1. Select an **Input type**.
            1. For type **Data**:
                1. Select **Data type** as **File** or **Folder**.
                1. Select **Data source** as **Upload from local**, **URI**, or **Datastore**.
                    - For **Upload from local**, select **Browse** under **Path to upload**, to choose the input file or folder.
                    - For **URI**, enter a storage data URI (for example, `abfss://` or `wasbs://` URI), or enter a data asset `azureml://`.
                    - For **Datastore**:
                        1. **Select a datastore** from the dropdown menu.
                        1. Under **Path to data**, select **Browse**.
                        1. In the pop-up screen titled **Path selection**, select the path of the code files on the workspace default blob storage.
                        1. Select **Save**.
            1. For type **Integer**, enter an integer value as **Input value**.
            1. For type **Number**, enter a numeric value as **Input value**.
            1. For type **Boolean**, select **True** or **False** as **Input value**.
            1. For type **String**, enter a string as **Input value**.
        1. To add an input, select **+ Add output** under **Outputs** and
            1. Enter an **Output name**. The output should refer to this name later to in the **Arguments**.
            1. Select **Output type** as **File** or **Folder**.
            1. For **Output URI destination**, enter a storage data URI (for example, `abfss://` or `wasbs://` URI) or enter a data asset `azureml://`.
