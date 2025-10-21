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
2. Create a workbench e.g. `test-wb` with the following:
* Notebook Image: CUDA
* Container Size: Medium
* Data Connection: `test-dc`

# Import Jupyter Notebooks into the Workbench
1. In RHOAI console, navigate to the `demo` project. Open the workbench `test-wb`.
2. Upload the files in `GenerativeModelWorkbench` and `PredictiveModelWorkbench`.

# Create a Custom Serving Runtime
1. In RHOAI console, go to Settings > Serving Runtimes. Create a custom serving runtime, using the `/triton/servingruntime.yaml` manifest file.

# Deploy and Serve the models
1. In RHOAI console, navigate to the `demo` project.
2. Go to the Model Serving tab and create a model server
3. Deploy both models where:
   * The file path for the predictive xgboost model should be `models/xgboost/1/`
   * the file path for the generative t5 model should be `models/t5-small/`


# Edit the Model Endpoints in app.py
1. In RHOAI, go to your project `demo `> workbench `test-wb` > open the workbench > navigate to app.py
2. Edit the model endpoints, for e.g.,

llm_infer_url = "https://t5-torchscript-micro-finance-demo.apps.example.com/v2/models/t5-torchscript/infer"
infer_url = "https://xgboost-micro-finance-demo.apps.example.com/v2/models/xgboost/infer"


# Deploy the Frontend
1. In the CLI, go to your project by running `oc project <project_name>`
2. Find the pod name of the pod containing the workbench
* `oc get po`
3. Create a service for the pod
* `oc expose pod <your-pod-name> --port=8080 --name=dev-frontend-service`
4. Create a route for the service
* `oc expose service dev-frontend-service`
5. Port-forward the pod running the workbench to port 8080
* `oc port-forward <your-pod-name> 8080:8080`
6. In RHOAI console, go to your project > workbench > open the workbench > navigate to DevFrontend.ipynb and run the last cell which should have the command
* `"!python flask_app/app.py"`
7. Get the route by running `oc get routes` and access it on the browser.

# Monitoring the Model on Grafana
1. Login to the cluster via OpenShift CLI (oc command).
2. Enable monitoring for user defined projects by running the command:
	* `oc -n openshift-monitoring patch configmap cluster-monitoring-config -p '{"data":{"config.yaml":"enableUserWorkload: true"}}'`
3. Validate that the prometheus and thanos-ruler pods were created in the openshift-user-workload-monitoring project:
	* `oc get pods -n openshift-user-workload-monitoring`
4. Run the command `oc new-project grafana`, `oc create sa grafana-serviceaccount`
5. Navigate to Openshift-console --> Projects --> Grafana
6. Click the + button on the top right to add the following: `/modelmesh_grafana/grafana-prep.yaml`
7. Navigate to OperatorHub --> Install Grafana Operator
8. Make sure to change the project context to openshift-user-workload-monitoring at the top
9. Create the grafana instance under "Installed Operators --> Grafana Operator --> Grafana --> Create Grafana --> YAML View" and paste the following: `/modelmesh_grafana/grafana-oauth.yaml`
10. Run the following command
	* `oc adm policy add-cluster-role-to-user cluster-monitoring-view -z grafana-serviceaccount`
11. Run the following command and save/paste the token for future use. 
	* `oc create token grafana-serviceaccount -n grafana`
12. Create route
	* `oc project grafana`
	* `oc get svc`
	* `oc create route edge grafana --service=grafana-oauth-service --insecure-policy=Redirect`
	* `oc get route`
13. In the browser --> Navigate to grafana route --> Login with root/secret
14. On the Grafana Dashboard > Connections > Add Data Source:
* Prometheus
* Name: Thanos
* Connection: https://\<thanos-querier-openshift-monitoring-service>:9091
* Authentication: Forward OAuth Identity
* Skip TLS certificate validation
* HTTP headers
  * Header: Authorization
  * Value: Bearer paste \<grafana-serviceaccount-token>>
15. In the left hand panel, navigate to Dashboards --> Manage --> Import and paste the following: `/modelmesh_grafana/grafana-dashboard.json`
16. Open the dashboard in the grafana UI and enter values for the following parameters on top of the dashboard
* Namespace: demo or whatever you named the project
* Service Name: modelmesh-serving
* Container: mm
* Runtime : triton


# Monitor NVIDIA GPU Metrics
Refer to the [NVIDIA DCGM Exporter Dashboard](https://docs.nvidia.com/datacenter/cloud-native/openshift/latest/enable-gpu-monitoring-dashboard.html#viewing-gpu-metrics) documentation for the steps to configure.

# References
OpenShift AI Documentation:
[https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/2.18/html/installing_and_uninstalling_openshift_ai_self-managed/installing-and-deploying-openshift-ai_install#requirements-for-openshift-ai-self-managed_install](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/2.18/html/installing_and_uninstalling_openshift_ai_self-managed/installing-and-deploying-openshift-ai_install#requirements-for-openshift-ai-self-managed_install)

Kaggle Dataset:
[https://www.kaggle.com/datasets/youngdaniel/loan-dataset](https://www.kaggle.com/datasets/youngdaniel/loan-dataset)

Configuring the NVIDIA DCGM Exporter Dashboard:
[https://docs.nvidia.com/datacenter/cloud-native/openshift/latest/enable-gpu-monitoring-dashboard.html#viewing-gpu-metrics](https://docs.nvidia.com/datacenter/cloud-native/openshift/latest/enable-gpu-monitoring-dashboard.html#viewing-gpu-metrics)
