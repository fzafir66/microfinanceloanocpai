Will edit Read Me Session Accordingly

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
