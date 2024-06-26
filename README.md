# Deploying a Voting App with PostgreSQL and Redis using Terraform and Kubernetes
Introduction
In this blog post, I'll walk you through the process of deploying a multi-component application using Kubernetes and Terraform. The application consists of a PostgreSQL database, a Redis instance, a voting app, a results app, and a worker app. Each component is managed as a Kubernetes deployment and exposed via a Kubernetes service. By the end of this guide, you'll have a fully functioning application running on your Kubernetes cluster.

## Prerequisites
Before we get started, ensure you have the following installed:

* Terraform
* kubectl
* A Kubernetes cluster (local or cloud-based)
## Project Structure
Here’s an overview of what we’ll be deploying:

1. PostgreSQL Database
2. Redis Instance
3. Voting App
4. Results App
5. Worker App
Each component will be defined as a Kubernetes deployment and exposed via a Kubernetes service.

## Kubernetes Manifests
Below are the Kubernetes manifests for each component.

**PostgreSQL Deployment and Service**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      name: postgres-deploy
      app: postgres-demo-deploy
  template:
    metadata:
      labels:
        name: postgres-deploy
        app: postgres-demo-deploy
    spec:
      containers:
      - name: postgres
        image: postgres
        ports:
        - containerPort: 5432
        env:
          - name: POSTGRES_USER
            value: "postgres"
          - name: POSTGRES_PASSWORD
            value: "postgres"
          - name: POSTGRES_HOST_AUTH_METHOD
            value: trust

---
apiVersion: v1
kind: Service
metadata:
  name: db
  labels:
    name: postgres-service
    app: postgres-demo-service
spec:
  ports:
    - port: 5432
      targetPort: 5432
  selector:
    name: postgres-deploy
    app: postgres-demo-deploy
```

### Redis Deployment and Service
yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      name: redis-deploy
      app: redis-demo-deploy
  template:
    metadata:
      labels:
        name: redis-deploy
        app: redis-demo-deploy
    spec:
      containers:
      - name: redis
        image: redis
        ports:
        - containerPort: 6379

---
apiVersion: v1
kind: Service
metadata:
  name: redis
  labels:
    name: redis-service
    app: redis-demo-service
spec:
  ports:
    - port: 6379
      targetPort: 6379
  selector:
    name: redis-deploy
    app: redis-demo-deploy
Results App Deployment and Service
yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: results-app-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: results-app-demo
      name: results-app-prod
  template:
    metadata:
      labels:
        app: results-app-demo
        name: results-app-prod
    spec:
      containers:
      - name: results-app
        image: kodekloud/examplevotingapp_result:v1
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: results-service
  labels:
    name: results-service
    app: results-demo-service
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: results-app-demo
    name: results-app-prod
Voting App Deployment and Service
yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: voting-app-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: voting-app-demo
      name: voting-app-prod
  template:
    metadata:
      labels:
        app: voting-app-demo
        name: voting-app-prod

---
apiVersion: v1
kind: Service
metadata:
  name: voting-service
  labels:
    name: voting-service
    app: voting-demo-service
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: voting-app-demo
    name: voting-app-prod
Worker App Deployment
yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker-app-deployment
  labels:
    name: worker-app-deploy
    app: demo-voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: worker-app-demo
      name: worker-app-prod
  template:
    metadata:
      name: worker-app-demo
      labels:
        app: worker-app-demo
        name: worker-app-prod
    spec:
      containers:
      - name: worker-app
        image: kodekloud/examplevotingapp_worker:v1
Deploying with Terraform
To deploy these Kubernetes resources using Terraform, you need to create a Terraform configuration file that defines your Kubernetes provider and the resources. Below is an example Terraform configuration for deploying the PostgreSQL service.

Terraform Configuration
hcl
Copy code
provider "kubernetes" {
  config_path = "~/.kube/config"
}

resource "kubernetes_deployment" "postgres" {
  metadata {
    name = "postgres-deployment"
  }
  spec {
    replicas = 1
    selector {
      match_labels = {
        name = "postgres-deploy"
        app  = "postgres-demo-deploy"
      }
    }
    template {
      metadata {
        labels = {
          name = "postgres-deploy"
          app  = "postgres-demo-deploy"
        }
      }
      spec {
        container {
          name  = "postgres"
          image = "postgres"
          port {
            container_port = 5432
          }
          env {
            name  = "POSTGRES_USER"
            value = "postgres"
          }
          env {
            name  = "POSTGRES_PASSWORD"
            value = "postgres"
          }
          env {
            name  = "POSTGRES_HOST_AUTH_METHOD"
            value = "trust"
          }
        }
      }
    }
  }
}

resource "kubernetes_service" "postgres" {
  metadata {
    name = "db"
  }
  spec {
    selector = {
      name = "postgres-deploy"
      app  = "postgres-demo-deploy"
    }
    port {
      port        = 5432
      target_port = 5432
    }
  }
}
Applying the Configuration
To apply the Terraform configuration and deploy your resources, run the following commands:

sh
Copy code
terraform init
terraform apply
Conclusion
In this blog post, we've covered how to deploy a multi-component application using Kubernetes and Terraform. This approach simplifies the management of your infrastructure and ensures consistency across environments. Feel free to extend this example by adding more components or integrating it with other tools and services.

Happy coding!
