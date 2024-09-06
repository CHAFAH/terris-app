# terris-app

To deploy the services defined in your Docker Compose file on an AKS (Azure Kubernetes Service) cluster using Terraform and Helm, we can break down the task into several key steps:

### 1. **Understanding the Docker Compose Services**

Your `docker-compose.yaml` file defines three services:

- **web**: The frontend web application.
- **api**: The backend API service.
- **mongo**: The database service (MongoDB).

All three services are connected to the same network (`network-backend`), and the `mongo` service uses a volume (`mongodb_data`) for persistent storage.

### 2. **Infrastructure with Terraform (AKS Cluster)**

You’ll need to use Terraform to provision the necessary infrastructure on Azure, specifically the AKS cluster. Here’s a quick outline of the Terraform steps:

1. **Create the AKS Cluster** using Terraform. Example configuration for AKS cluster:
   
   ```hcl
   provider "azurerm" {
     features {}
   }

   resource "azurerm_resource_group" "rg" {
     name     = "aks-resource-group"
     location = "East US"
   }

   resource "azurerm_kubernetes_cluster" "aks" {
     name                = "aks-cluster"
     location            = azurerm_resource_group.rg.location
     resource_group_name = azurerm_resource_group.rg.name
     dns_prefix          = "aks-cluster"

     default_node_pool {
       name       = "default"
       node_count = 3
       vm_size    = "Standard_DS2_v2"
     }

     identity {
       type = "SystemAssigned"
     }
   }

   output "kube_config" {
     value = azurerm_kubernetes_cluster.aks.kube_config
   }
   ```

2. **Apply the Terraform Configuration**:
   ```bash
   terraform init
   terraform apply
   ```

### 3. **Convert Docker Compose to Kubernetes Resources**

Each Docker Compose service needs to be transformed into Kubernetes manifests.

#### **MongoDB Deployment and Service**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: "username"
        - name: MONGO_INITDB_ROOT_PASSWORD
          value: "password"
        volumeMounts:
        - name: mongodb-data
          mountPath: /data/db
      volumes:
      - name: mongodb-data
        persistentVolumeClaim:
          claimName: mongodb-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: mongo
spec:
  selector:
    app: mongo
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
```

#### **API Deployment and Service**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
      - name: api
        image: origenai/cloud-engineer-test-sample-app-backend:1.0.0
        ports:
        - containerPort: 3001
---
apiVersion: v1
kind: Service
metadata:
  name: api
spec:
  selector:
    app: api
  ports:
    - protocol: TCP
      port: 3001
      targetPort: 3001
```

#### **Web Deployment and Service**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: origenai/cloud-engineer-test-sample-app-frontend:1.0.0
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
```

### 4. **Helm Charts**

To make the deployment modular and reusable, you can create Helm charts for each service.

1. **Create Helm Charts**: 
   ```bash
   helm create web
   helm create api
   helm create mongo
   ```

2. **Deploy Helm Charts**:
   After creating the charts, you can deploy them like this:
   ```bash
   helm install web ./web
   helm install api ./api
   helm install mongo ./mongo
   ```

### 5. **Ingress for External Access**

If you want to expose the frontend (web) application externally, you can use an ingress resource:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
  - host: <your-domain>
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 3000
```

### 6. **Access Control and Security**

- Ensure that your Kubernetes cluster has RBAC (Role-Based Access Control) enabled.
- Use Kubernetes secrets for sensitive information (such as MongoDB credentials).
- Use network policies to restrict access between services.

### 7. **Document Your Steps in the README**

In your `README.md` file, include:
- A description of the application.
- Instructions to set up Terraform, Helm, and Kubernetes.
- Instructions for accessing the application.

Once this is done, package your project and submit it as per the test requirements. Let me know if you need any additional clarification!
