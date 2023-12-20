# **Deployment Documentation Summary: Building and Deploying a Flask Application with Kubernetes, CodeBuild, and PostgreSQL**

        Exclusively composed by Chady Achraf (Chady00), this README bears the sole authorship of Chady himself. Only Chady00 contributed to the project developement steps.


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

<table><tbody><tr><td><code>kubectl port-forward --namespace default svc/my-pos-postgresql 5432:5432 &amp;</code><br><code>PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 &lt; db/1_create_tables.sql</code></td></tr></tbody></table>

### **Connecting Via a Pod (Option 2)**

*   Use **kubectl exec** to enter a pod with access to PostgreSQL service.
*   Connect to the database using **psql** with service name and port.

<table><tbody><tr><td><code>kubectl exec -it my-pos-postgresql-0 -- bash</code><br><code>PGPASSWORD="$POSTGRES_PASSWORD" psql postgres://postgres@my-pos-postgresql:5432/postgres</code></td></tr></tbody></table>

### **Seed Database**

*   Run SQL seed files located in the **db/** directory to create tables and populate them with data.

<table><tbody><tr><td><code>kubectl port-forward --namespace default svc/my-pos-postgresql 5432:5432 &amp;</code><br><code>PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 &lt; db/<file_name.sql></code></td></tr></tbody></table>

*   Testing the validity of the populated data.
  
![8  Select Query on Table](https://github.com/Chady00/Operationalizing-a-Coworking-Space-Microservice/assets/84717550/c31bd593-b40d-4a38-b134-48c4442ef956)


## **2\. Local Testing of Analytics Application**

*   Navigate to the **analytics/** directory.
*   Install requirements for the analytics application.  
    

<table><tbody><tr><td><code>python -m pip install --upgrade pip</code><br><code>pip install -r analytics/requirements.txt</code></td></tr></tbody></table>

*   Run the analytics application locally:

<table><tbody><tr><td><code>DB_USERNAME=postgres DB_PASSWORD=rJ6lPsG58r python analytics/app.py</code></td></tr></tbody></table>


![2  running docker image locally](https://github.com/Chady00/Operationalizing-a-Coworking-Space-Microservice/assets/84717550/c1ab48bd-fd9e-4b3c-856a-cf248ad90b64)


*   Test the application using curl commands.

<table><tbody><tr><td><code>curl http://127.0.0.1:5153/api/reports/daily_usage</code><br><code>curl http://127.0.0.1:5153/api/reports/user_visits</code></td></tr></tbody></table>


![9-teting the database locally using curl](https://github.com/Chady00/Operationalizing-a-Coworking-Space-Microservice/assets/84717550/f8d954f8-b04d-4951-a8ee-a908ca25f636)



## **3\. Docker Image Creation and Local Deployment**

*   Build the Docker image locally:

<table><tbody><tr><td><code>docker build -t myimage .</code></td></tr></tbody></table>

*   Run the Docker container locally.  
    

<table><tbody><tr><td><code>docker run -e DB_USERNAME=postgres -e DB_PASSWORD=rJ6lPsG58r --network=host myimage</code></td></tr></tbody></table>


![2  running docker image locally](https://github.com/Chady00/Operationalizing-a-Coworking-Space-Microservice/assets/84717550/65f44ee5-a6cb-4efb-afc3-da14dc482719)


## **4\. AWS CodeBuild Pipeline**

*   Configure CodeBuild to pull the image from the GitHub repository, build it, and push it to the existing ECR repository. Make sure to use the buildspecs.yml file.
  
![3  Codebuild success_no_trigger](https://github.com/Chady00/Operationalizing-a-Coworking-Space-Microservice/assets/84717550/860897f6-7aa7-4cd4-b6e8-01ee2edf7067)


## **5\. Kubernetes Deployment**

*   Deploy the application using the **kubernetes-flaskapp-api.yml** file, **db-configmap.yml** file, and **db-secret.yml** file.
*   Set **DB\_HOST** to the service name of the Kubernetes PostgreSQL pod.

![13  kubectl describe deployment SERVICE_NAM](https://github.com/Chady00/Operationalizing-a-Coworking-Space-Microservice/assets/84717550/84ab4a93-0383-43e5-bd1b-731ea3ebc8b2)


## **6\. CloudWatch Agent Setup**

*   Set up CloudWatch agent for monitoring.
*   Use the provided script with appropriate substitutions for **ClusterName**, **RegionName**, **FluentBitHttpPort**, etc.

<table><tbody><tr><td><code>ClusterName=flask-cluster</code><br><code>RegionName=us-east-1</code><br><code>FluentBitHttpPort='2020'</code><br><code>FluentBitReadFromHead='Off'</code><br><code>[[ ${FluentBitReadFromHead} = 'On' ]] &amp;&amp; FluentBitReadFromTail='Off'|| FluentBitReadFromTail='On'</code><br><code>[[ -z ${FluentBitHttpPort} ]] &amp;&amp; FluentBitHttpServer='Off' || FluentBitHttpServer='On'</code><br><code>curl https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluent-bit-quickstart.yaml | sed 's/{{cluster_name}}/'${ClusterName}'/;s/{{region_name}}/'${RegionName}'/;s/{{http_server_toggle}}/"'${FluentBitHttpServer}'"/;s/{{http_server_port}}/"'${FluentBitHttpPort}'"/;s/{{read_from_head}}/"'${FluentBitReadFromHead}'"/;s/{{read_from_tail}}/"'${FluentBitReadFromTail}'"/' | kubectl apply -f -</code></td></tr></tbody></table>

![17  cloudwatch containerInsights](https://github.com/Chady00/Operationalizing-a-Coworking-Space-Microservice/assets/84717550/cd64a095-70f9-4233-bf1e-1db075d41d86)

![14  CloudWatch live tail for logs](https://github.com/Chady00/Operationalizing-a-Coworking-Space-Microservice/assets/84717550/52ada513-587a-40d5-87cb-21c666e8ceb9)


## **Important Notes**

*   Ensure PostgreSQL password is encoded in Base64 for use in Kubernetes secrets.
*   Update placeholders such as **\<SERVICE\_NAME>**, **\<POD\_NAME>**, **\<FILE\_NAME.sql>**, **\<ENV\_VARS>**, and **\<BASE\_URL>** with actual values.
*   Adjust network settings and configurations based on the development environment.
*   Verify application functionality at each step before proceeding to the next one.
*   Ensure that dependencies are installed, and required seed files are available.
*   Pay attention to security considerations, such as password handling and network configurations.

`docker build -t myimage . docker run -e DB_USERNAME=postgres -e DB_PASSWORD=rJ6lPsG58r --network=host myimage`

`DB_USERNAME=postgres DB_PASSWORD=rJ6lPsG58r python analytics/app.py`

`kubectl port-forward --namespace default svc/my-pos-postgresql 5432:5432 & PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < db/2_seed_users.sql`

`kubectl exec -it my-pos-postgresql-0 -- bash PGPASSWORD="$POSTGRES_PASSWORD" psql postgres://postgres@my-pos-postgresql:5432/postgres`

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
To avoid limited resources problems, it's mandatory to choose a reasonable AMI instance according to the project requirements.

![8 reason for my resources](https://github.com/Chady00/Operationalizing-a-Coworking-Space-Microservice/assets/84717550/c55db40a-e52c-4db6-a329-f42beafdd335)


**Reasonable Memory and CPU allocation in the Kubernetes deployment configuration :** 2 vCPUs, 2.0 GiB of memory with 5 Gibps of bandwidth.

**A reasonable AWS instance would be:** t4g.small ARM AMI which is enough to complete the entire workflow.

**To reduce costs**: autoscaling could be applied, as well as using efficient instance types.


![16  Cloudwatch Logs insights for cluster](https://github.com/Chady00/Operationalizing-a-Coworking-Space-Microservice/assets/84717550/494f76bb-3c9a-44ed-a4ed-cae73c7f8385)


