To create the Helm charts for deploying the application (frontend, backend, MongoDB) and applying them to your Azure Kubernetes Service (AKS) cluster, you need to follow these steps:

### Steps to Create and Apply Helm Charts

1. **Initialize Helm Charts for Each Service**

Helm charts need to be created for each service (frontend, backend, MongoDB) using Helm's chart creation command:

```bash
helm create frontend
helm create backend
helm create mongodb
```

Each of these commands will create a directory structure for the Helm chart with the default files such as `Chart.yaml`, `values.yaml`, and templates for Kubernetes resources (deployment, service, etc.).

2. **Customize Helm Chart Values**

Next, you will need to modify the `values.yaml` and Kubernetes template files for each service to suit your deployment.

---

### MongoDB Helm Chart

#### `values.yaml` for MongoDB

This `values.yaml` file will define the MongoDB configurations. Replace sensitive values with references to secrets.

```yaml
replicaCount: 1

image:
  repository: mongo
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 27017

mongodb:
  username: username
  password: password
  database: mydatabase

persistence:
  enabled: true
  storageClass: default
  accessModes:
    - ReadWriteOnce
  size: 1Gi

resources: {}
```

#### MongoDB Deployment Template (`templates/deployment.yaml`)

Replace hard-coded values for username and password with references to Kubernetes secrets.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mongodb.fullname" . }}
  labels:
    {{- include "mongodb.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "mongodb.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "mongodb.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: mongodb
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: mongodb-username
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: mongodb-password
          ports:
            - containerPort: {{ .Values.service.port }}
          volumeMounts:
            - name: mongodb-storage
              mountPath: /data/db
      volumes:
        - name: mongodb-storage
          persistentVolumeClaim:
            claimName: {{ include "mongodb.fullname" . }}-pvc
```

#### Secret for MongoDB Credentials

Create a secret for MongoDB credentials using base64 encoded values.

```bash
kubectl create secret generic mongodb-secret \
  --from-literal=mongodb-username=$(echo -n 'username' | base64) \
  --from-literal=mongodb-password=$(echo -n 'password' | base64)
```

#### MongoDB Persistent Volume Claim (`templates/pvc.yaml`)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ include "mongodb.fullname" . }}-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: {{ .Values.persistence.size }}
```

---

### Frontend Helm Chart

#### `values.yaml` for Frontend

```yaml
replicaCount: 2

image:
  repository: origenai/cloud-engineer-test-sample-app-frontend
  tag: 1.0.0
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 3000

resources: {}
```

#### Frontend Deployment Template (`templates/deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "frontend.fullname" . }}
  labels:
    {{- include "frontend.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "frontend.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "frontend.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: frontend
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.service.port }}
```

---

### Backend Helm Chart

#### `values.yaml` for Backend

```yaml
replicaCount: 2

image:
  repository: origenai/cloud-engineer-test-sample-app-backend
  tag: 1.0.0
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 3001

resources: {}
```

#### Backend Deployment Template (`templates/deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "backend.fullname" . }}
  labels:
    {{- include "backend.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "backend.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "backend.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: backend
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.service.port }}
```

---

### Deploy the Helm Charts

1. **Install the MongoDB Helm Chart**:

```bash
helm install mongodb ./mongodb --namespace your-namespace
```

2. **Install the Backend Helm Chart**:

```bash
helm install backend ./backend --namespace your-namespace
```

3. **Install the Frontend Helm Chart**:

```bash
helm install frontend ./frontend --namespace your-namespace
```

---

### Accessing the Application

Since there is no domain, use a `NodePort` service to expose the application components and access them via the external node IP. You can get the external IP of the nodes using:

```bash
kubectl get nodes -o wide
```

Then, use the IP and assigned node port for each service (MongoDB, Backend, Frontend) to access the services.

---

### Conclusion

This README file provides detailed instructions on how to create and configure Helm charts for MongoDB, Backend, and Frontend services, along with steps to deploy them to AKS using Helm. 
The configurations include best practices such as the use of Kubernetes secrets for sensitive data, and persistent volume claims for storage.
