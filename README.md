Will edit Read Me Session Accordingly

# Pre-requisites
View the requirements in [docs.redhat.com](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/2.18/html/installing_and_uninstalling_openshift_ai_self-managed/installing-and-deploying-openshift-ai_install) to install Red Hat OpenShift AI Self-Managed.

The required operators for this demo include:
* Node Feature Discovery Operator
* NVIDIA GPU Operator
* OpenShift Serverless Operator
* OpenShift Service Mesh 2 Operator
* OpenShift AI Operator
* Grafana Operator

# Deploy MinIO to store raw data and model files
1. Create a Data Science project in RHOAI, e.g. `demo`.
2. In OCP console, navigate to the `demo` project. Create a S3-compatible storage server, MinIO, using the `/minio/minio.yaml` manifest file.
3. In OCP console, navigate to Networking > Routes. Login to MinIO console and create a bucket, e.g. `test`.
4. In RHOAI console, under the `demo` project created earlier, and add a new data connection, e.g. `test-dc`.


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



# Define inference URL for the hosted LLM
llm_infer_url = "https://t5-torchscript-micro-finance-demo.apps.dell90b.tecaiocp.com/v2/models/t5-torchscript/infer"

# Load necessary files
training_columns = joblib.load("training_columns.pkl")
best_model = joblib.load("best_model.joblib")
preprocessor = best_model.named_steps['preprocessor']

# Load training data for LIME initialization
X_train = pd.read_csv("X_train.csv")
X_train_processed = preprocessor.transform(X_train)

#ENDPOINTS
# Hosted Model API Endpoint
infer_url = "https://xgboost-micro-finance-demo.apps.dell90b.tecaiocp.com/v2/models/xgboost/infer"
headers = {"Content-Type": "application/json"}

# Monitoring the Model on Grafana

