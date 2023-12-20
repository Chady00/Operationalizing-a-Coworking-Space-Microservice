## **Deployment Documentation Summary: Building and Deploying a Flask Application with Kubernetes, CodeBuild, and PostgreSQL**
In this deployment process, we leverage a robust set of tools and technologies to streamline the development, build, and deployment of a Flask application along with a PostgreSQL database within a Kubernetes cluster.
**Tools and Technologies:**
1.  **Flask Application:** A lightweight and versatile web framework for Python.
2.  **Docker:** Containerization tool used to package the Flask application, ensuring consistency across different environments.
3.  **GitHub:** Version control platform to host and manage the application's source code.
4.  **AWS CodeBuild:** Continuous integration service automating the build process by fetching the application code from GitHub, building a Docker image, and pushing it to Amazon Elastic Container Registry (ECR).
5.  **Amazon ECR:** Container registry for storing and managing Docker images securely.
6.  **Kubernetes:** Container orchestration platform facilitating the deployment, scaling, and management of containerized applications.
7.  **kubectl:** Command-line tool for interacting with Kubernetes clusters.
8.  **Helm:** Kubernetes package manager for simplifying and automating the deployment of applications.
9.  **CloudWatch:** Setting up cloudwatch daemon and container insights to monitor the flask app logs and ensure correctness.


## **Project Development Steps**

## **1\. Database Setup and Connection**

### **Port Forwarding (Option 1)**

*   Use **kubectl port-forward** to forward PostgreSQL service port (5432) to local machine.
*   Connect to the database using **psql** with forwarded port.

### **Connecting Via a Pod (Option 2)**

*   Use **kubectl exec** to enter a pod with access to PostgreSQL service.
*   Connect to the database using **psql** with service name and port.

### **Seed Database**

*   Run SQL seed files located in the **db/** directory to create tables and populate them with data.

## **2\. Local Testing of Analytics Application**

*   Navigate to the **analytics/** directory.
*   Install dependencies using **pip install -r requirements.txt**.
*   Run the application locally:
*   Test the application using curl commands.

## **3\. Docker Image Creation and Local Deployment**

*   Build the Docker image locally:

## **4\. AWS CodeBuild Pipeline**

*   Configure CodeBuild to pull the image from the GitHub repository, build it, and push it to the existing ECR repository.

## **5\. Kubernetes Deployment**

*   Deploy the application using the **kubernetes-flaskapp-api.yml** file, **db-configmap.yml** file, and **db-secret.yml** file.
*   Set **DB\_HOST** to the service name of the Kubernetes PostgreSQL pod.

## **6\. CloudWatch Agent Setup**

*   Set up CloudWatch agent for monitoring.
*   Use the provided script with appropriate substitutions for **ClusterName**, **RegionName**, **FluentBitHttpPort**, etc.

## **Important Notes**

*   Ensure PostgreSQL password is encoded in Base64 for use in Kubernetes secrets.
*   Update placeholders such as **\<SERVICE\_NAME>**, **\<POD\_NAME>**, **\<FILE\_NAME.sql>**, **\<ENV\_VARS>**, and **\<BASE\_URL>** with actual values.
*   Adjust network settings and configurations based on the development environment.
*   Verify application functionality at each step before proceeding to the next one.
*   Ensure that dependencies are installed, and required seed files are available.
*   Pay attention to security considerations, such as password handling and network configurations.

bashCopy code

`docker build -t myimage . docker run -e DB_USERNAME=postgres -e DB_PASSWORD=rJ6lPsG58r --network=host myimage`

bashCopy code

`DB_USERNAME=postgres DB_PASSWORD=rJ6lPsG58r python analytics/app.py`

bashCopy code

`kubectl port-forward --namespace default svc/my-pos-postgresql 5432:5432 & PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < db/2_seed_users.sql`

bashCopy code

`kubectl exec -it my-pos-postgresql-0 -- bash PGPASSWORD="$POSTGRES_PASSWORD" psql postgres://postgres@my-pos-postgresql:5432/postgres`

bashCopy code

`kubectl port-forward --namespace default svc/my-pos-postgresql 5432:5432 & PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < db/1_create_tables.sql`



**Deployment Process Overview:**
**Local Testing:** Developers locally test the Flask application with a PostgreSQL database to ensure its functionality using curl commands.

**Docker Image Creation:** A Docker image is created locally using the Docker build command, ensuring the application's dependencies are encapsulated.

**AWS CodeBuild Pipeline:** CodeBuild is configured to automatically trigger when changes are pushed to the GitHub repository. It fetches the code, builds the Docker image, and pushes it to the ECR repository.

**Kubernetes Deployment Files:** Deployment files, including kubernetes-flaskapp-api.yml, db-configmap.yml, and db-secret.yml, are used to deploy the Flask application and PostgreSQL database to the Kubernetes cluster.

**Configuration Management:** The database configuration, including host and credentials, is managed through ConfigMaps and Secrets in Kubernetes, promoting secure and scalable configuration management.

**Release Process:** To release new builds, developers update the code in the GitHub repository. CodeBuild automates the build process, ensuring that the latest Docker image is pushed to ECR. Kubernetes then deploys the updated application using the specified configuration files.

**Continuous Monitoring:** AWS CloudWatch logs provide insight into application behavior, aiding in monitoring and troubleshooting.
This deployment approach enhances collaboration, ensures consistency in different environments, and facilitates a seamless release process. Developers can focus on writing code, knowing that the CI/CD pipeline automates the build, test, and deployment stages, resulting in a reliable and scalable application in a Kubernetes environment.

**Resources:**
**Reasonable Memory and CPU allocation in the Kubernetes deployment configuration :** 2 vCPUs, 2.0 GiB of memory with 5 Gibps of bandwidth.

 **A reasonable AWS instance would be:** t4g.small ARM AMI which is enough to complete the entire workflow.
 
**To reduce costs**: autoscaling could be applied, as well as using efficient instance types.
