Note: Work in Progress

# Pre-requisites
View the requirements in [docs.redhat.com](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/2.18/html/installing_and_uninstalling_openshift_ai_self-managed/installing-and-deploying-openshift-ai_install) to install Red Hat OpenShift AI Self-Managed.

The required operators for this demo include:
* Node Feature Discovery Operator
* NVIDIA GPU Operator
* OpenShift Serverless Operator
* OpenShift Service Mesh 2 Operator
* OpenShift AI Operator
* Grafana Operator

# Create a New Project
1. Create a Data Science project in RHOAI, e.g. `demo`.

# Deploy MinIO to store raw data and model files
1. In OCP console, navigate to the `demo` project. Create a S3-compatible storage server, MinIO, using the `/minio/minio.yaml` manifest file.
2. In OCP console, navigate to Networking > Routes. Login to MinIO console and create a bucket, e.g. `test`.
3. In RHOAI console, under the `demo` project created earlier, and add a new data connection, e.g. `test-dc`.


# Store the files in MinIO
In the created bucket called ‘demo’ for example, store the files in the following path:
* dataset
  * Store raw data from [kaggle](https://www.kaggle.com/datasets/youngdaniel/loan-dataset)
* models
  * xgboost
    * 1
      * <trained_xgboost.onnx>
  * t5-small
    * config.pbtxt
    * 1
      * <model_files>
* flanT5-fine-tuned
  * <tokenizer_files>
* training_columns.pkl
* best_model.joblib
* X_train.csv (need to run ‘1. TrainTest.ipynb’ to generate this file)

# Create a New Workbench
1. In RHOAI console, navigate to the `demo` project.
2. 

# Import Jupyter Notebooks into the Workbench
1. 

# Create a Custom Serving Runtime
1. In RHOAI console, got to Settings > Serving Runtimes. Create a custome serving runtime, using the `/triton/servingruntime.yaml` manifest file.

# Deploy and Serve the models
1.



# Edit the model endpoints in app.py
1. 

llm_infer_url = "https://t5-torchscript-micro-finance-demo.apps.example.com/v2/models/t5-torchscript/infer"
infer_url = "https://xgboost-micro-finance-demo.apps.example.com/v2/models/xgboost/infer"


# Monitoring the Model on Grafana


# Monitor NVIDIA GPU Metrics
Refer to the [NVIDIA DCGM Exporter Dashboard](https://docs.nvidia.com/datacenter/cloud-native/openshift/latest/enable-gpu-monitoring-dashboard.html#viewing-gpu-metrics) documentation for the steps to configure.

# References
OpenShift AI Documentation:
[https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/2.18/html/installing_and_uninstalling_openshift_ai_self-managed/installing-and-deploying-openshift-ai_install#requirements-for-openshift-ai-self-managed_install](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/2.18/html/installing_and_uninstalling_openshift_ai_self-managed/installing-and-deploying-openshift-ai_install#requirements-for-openshift-ai-self-managed_install)

Kaggle Dataset:
[https://www.kaggle.com/datasets/youngdaniel/loan-dataset](https://www.kaggle.com/datasets/youngdaniel/loan-dataset)

Configuring the NVIDIA DCGM Exporter Dashboard:
[https://docs.nvidia.com/datacenter/cloud-native/openshift/latest/enable-gpu-monitoring-dashboard.html#viewing-gpu-metrics](https://docs.nvidia.com/datacenter/cloud-native/openshift/latest/enable-gpu-monitoring-dashboard.html#viewing-gpu-metrics)
