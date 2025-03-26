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

# Monitoring the Model on Grafana
Login to the cluster via OpenShift CLI (oc command).
Enable monitoring for user defined projects as listed here or run the command:
oc -n openshift-monitoring patch configmap cluster-monitoring-config -p '{"data":{"config.yaml":"enableUserWorkload: true"}}'

Validate that the prometheus and thanos-ruler pods were created in the openshift-user-workload-monitoring project:


oc get pods -n openshift-user-workload-monitoring
NAME                                   READY   STATUS    RESTARTS   AGE
prometheus-operator-675f9d4b96-f9zxd   2/2     Running   0          8d
prometheus-user-workload-0             6/6     Running   0          8d
prometheus-user-workload-1             6/6     Running   0          8d
thanos-ruler-user-workload-0           4/4     Running   0          8d
thanos-ruler-user-workload-1           4/4     Running   0          8d


Run the command oc new-project grafana, oc create sa grafana-serviceaccount
Navigate to Openshift-console --> Projects --> Grafana
Click the + button on the top right to add the following:

apiVersion: v1
kind: Secret
metadata:
  name: grafana-k8s-proxy
type: Opaque
stringData:
  username: admin
  password: secret
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: grafana-proxy
rules:
  - apiGroups:
      - authentication.k8s.io
    resources:
      - tokenreviews
    verbs:
      - create
  - apiGroups:
      - authorization.k8s.io
    resources:
      - subjectaccessreviews
    verbs:
      - create
---
apiVersion: authorization.openshift.io/v1
kind: ClusterRoleBinding
metadata:
  name: grafana-proxy
roleRef:
  name: grafana-proxy
subjects:
  - kind: ServiceAccount
    name: grafana-serviceaccount
    namespace: grafana
userNames:
  - system:serviceaccount:grafana:grafana-serviceaccount
---
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    config.openshift.io/inject-trusted-cabundle: "true"
  name: ocp-injected-certs


Navigate to OperatorHub --> Install Grafana Operator
Make sure to change the project context to openshift-user-workload-monitoring at the top
Create the grafana instance under "Installed Operators --> Grafana Operator --> Grafana --> Create Grafana --> YAML View" and paste the following

apiVersion: grafana.integreatly.org/v1beta1
kind: Grafana
metadata:
  name: grafana-oauth
  namespace: grafana
spec:
  config:  
    auth:
      disable_login_form: 'false'
      disable_signout_menu: 'true'
    auth.anonymous:
      enabled: 'false'
    auth.basic:
      enabled: 'true'
    log:
      level: warn
      mode: console
    security:  
      admin_password: secret
      admin_user: root
  secrets:
    - grafana-k8s-tls
    - grafana-k8s-proxy
  client:
    preferService: true
  containers:  
    - args:
        - '-provider=openshift'
        - '-pass-basic-auth=false'
        - '-https-address=:9091'
        - '-http-address='
        - '-email-domain=*'
        - '-upstream=http://localhost:3000'
        - '-tls-cert=/etc/tls/private/tls.crt'
        - '-tls-key=/etc/tls/private/tls.key'
        - >-
          -client-secret-file=/var/run/secrets/kubernetes.io/serviceaccount/token
        - '-cookie-secret-file=/etc/proxy/secrets/session_secret'
        - '-openshift-service-account=grafana-serviceaccount'
        - '-openshift-ca=/etc/pki/tls/cert.pem'
        - '-openshift-ca=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt'
        - '-openshift-ca=/etc/grafana-configmaps/ocp-injected-certs/ca-bundle.crt'
        - '-skip-auth-regex=^/metrics'
        - >-
          -openshift-sar={"namespace": "grafana", "resource": "services",
          "verb": "get"}
      image: 'quay.io/openshift/origin-oauth-proxy:4.8'
      name: grafana-proxy
      ports:
        - containerPort: 9091
          name: grafana-proxy
      resources: {}
      volumeMounts:
        - mountPath: /etc/tls/private
          name: secret-grafana-k8s-tls
          readOnly: false
        - mountPath: /etc/proxy/secrets
          name: secret-grafana-k8s-proxy
          readOnly: false
  ingress:
    enabled: true
    targetPort: grafana-proxy
    termination: reencrypt
  service:
    annotations:
      service.alpha.openshift.io/serving-cert-secret-name: grafana-k8s-tls
    ports:
      - name: grafana-proxy
        port: 9091
        protocol: TCP
        targetPort: grafana-proxy
  serviceAccount:
    annotations:
      serviceaccounts.openshift.io/oauth-redirectreference.primary: >-
        {"kind":"OAuthRedirectReference","apiVersion":"v1","reference":{"kind":"Route","name":"grafana-route"}}
  configMaps:
    - ocp-injected-certs
  dashboardLabelSelector:
    - matchExpressions:
        - key: app
          operator: In
          values:
            - grafana


Run the following command
oc adm policy add-cluster-role-to-user cluster-monitoring-view -z grafana-serviceaccount
Run the following command and save/paste the token for future use. 
oc create token grafana-serviceaccount -n grafana
Create route

oc project grafana
oc get svc
oc create route edge grafana --service=grafana-oauth-service --insecure-policy=Redirect
oc get route

Create grafana datasource under: "Installed Operators --> Grafana Operator --> Grafana --> Create Grafana DataSource --> YAML View" and paste this

apiVersion: grafana.integreatly.org/v1beta1
kind: GrafanaDatasource
metadata:
  name: prometheus-grafanadatasource
  namespace: grafana
spec:
  datasource:
    access: proxy
    editable: true
    isDefault: true
    jsonData:
      httpHeaderName1: Authorization
      timeInterval: 5s
      tlsSkipVerify: true
    name: Prometheus
    secureJsonData:
      httpHeaderValue1: >-
        Bearer <TOKEN>  
    type: prometheus
    url: 'https://thanos-querier.openshift-monitoring.svc.cluster.local:9091'  
  instanceSelector:
    matchLabels:
      app: grafana

Edit the yaml on line 18 to paste the <token> from previous step
Run the following command
oc port-forward svc/grafana-oauth-service 3000:3000
In the browser --> Navigate to grafana route localhost:3000 --> Login with root/secret 
Add Data Source:
Prometheus
Name: Prometheus
Connection: https://<prometheus-k8s-service>:9091
Authentication: Forward OAuth Identity
Skip TLS certificate validation
HTTP headers
Header: Authorization
Value: Bearer <prometheus-k8s-token>
On the Grafana Dashboard > Connections > Add Data Source:
Prometheus
Name: Thanos
Connection: https://<thanos-querier-openshift-monitoring-service>:9091
Authentication: Forward OAuth Identity
Skip TLS certificate validation
HTTP headers
Header: Authorization
Value: Bearer <grafana-serviceaccount-token>
In the left hand panel, navigate to Dashboards --> Manage --> Import and paste the following json
Open the dashboard in the grafana UI and enter values for the following parameters on top of the dashboard
Namespace: <demo or whatever you named the project>
Service Name: modelmesh-serving
Container: mm
Runtime : triton
