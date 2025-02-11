
## YAML: AutoML text classification multilabel pipeline job

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/pipelineJob.schema.json
type: pipeline

description: Pipeline using AutoML Text Classification Multilabel task

display_name: pipeline-with-text-classification-multilabel
experiment_name: pipeline-with-automl

settings:
  default_compute: azureml:gpu-cluster

inputs:
  text_classification_multilabel_training_data:
    type: mltable
    path: ./training-mltable-folder
  text_classification_multilabel_validation_data:
    type: mltable
    path: ./validation-mltable-folder

jobs:
  preprocessing_node:
    type: command
    component: file:./components/component_preprocessing.yaml
    inputs:
      train_data: ${{parent.inputs.text_classification_multilabel_training_data}}
      validation_data: ${{parent.inputs.text_classification_multilabel_validation_data}}
    outputs:
      preprocessed_train_data:
        type: mltable
      preprocessed_validation_data:
        type: mltable
  text_classification_multilabel_node:
    type: automl
    task: text_classification_multilabel
    log_verbosity: info
    primary_metric: accuracy
    limits:
      max_trials: 1
      timeout_minutes: 60
    target_column_name: terms
    training_data: ${{parent.jobs.preprocessing_node.outputs.preprocessed_train_data}}
    validation_data: ${{parent.jobs.preprocessing_node.outputs.preprocessed_validation_data}}
    # currently need to specify outputs "mlflow_model" explicitly to reference it in following nodes
    outputs:
      best_model:
        type: mlflow_model
  register_model_node:
    type: command
    component: file:./components/component_register_model.yaml
    inputs:
      model_input_path: ${{parent.jobs.text_classification_multilabel_node.outputs.best_model}}
      model_base_name: paper_categorization_model
      
```

## Next steps

- [Install and use the CLI (v2)](how-to-configure-cli.md)
