# Connecting a Cloud Run service to Cloud SQL for PostgreSQL via Private IP

A demo Cloud Run application in Java that connects to a Cloud SQL instance via Private IP. Based on the 
official GCP [sample](https://github.com/GoogleCloudPlatform/java-docs-samples/tree/main/cloud-sql/postgres/servlet).


## Before you begin

1. Enable the following APIs:

    - Compute Engine API
    - Cloud SQL Admin API
    - Cloud Run API
    - Container Registry API
    - Cloud Build API

2. Create a [Serverless VPC Connector](https://cloud.google.com/vpc/docs/configure-serverless-vpc-access#create-connector)
   for connect to the instance's Private IP.

3. Create a 2nd Gen Cloud SQL instance for PostgreSQL with Private IP by following these 
[instructions](https://cloud.google.com/sql/docs/postgres/quickstart-cloud-run#expandable-2).

4. Create a [database](https://cloud.google.com/sql/docs/postgres/quickstart-cloud-run#create-instance) and 
a [user](https://cloud.google.com/sql/docs/postgres/quickstart-cloud-run#create_a_user) for your application. 

5. Create a service account with the 'Cloud SQL Client' and 'Storage Admin' permissions. Download a JSON key to use to 
authenticate your connection. Alternatively, configure the service account used by Cloud Run.


## Running locally

To run this application locally, run the following command inside the project folder:

    mvn jetty:run

Navigate towards http://127.0.0.1:8080 to verify your application is running correctly.


## Deploy to Cloud Run

1. Build the container image using [Jib](https://cloud.google.com/java/getting-started/jib):

       mvn clean package -DskipTests

2. To publish the image to container registry run:

       mvn deploy -DskipTests

3. Deploy the app:
    ```sh
    gcloud run deploy run-sql --image gcr.io/YOUR_PROJECT_ID/run-sql \
      --add-cloudsql-instances INSTANCE-CONNECTION-NAME \
      --set-env-vars INSTANCE_CONNECTION_NAME="INSTANCE-CONNECTION-NAME" \
      --set-env-vars CLOUD_SQL_CONNECTION_NAME="INSTANCE-CONNECTION-NAME" \
      --set-env-vars DB_NAME="DB-NAME" \
      --set-env-vars DB_USER="DB-USER-NAME" \
      --set-env-vars DB_PASS="DB-USER-PASSWORD" \
      --set-env-vars DB_HOST="DB-PRIVATE-IP" \
      --set-env-vars DB_PORT="5432" \
      --vpc-connector="VPC-CONNECTOR-NAME"
    ```
    Replace environment variables with the correct values for your Cloud SQL
instance configuration.

Take note of the URL output at the end of the deployment process.

**Note:** Saving credentials in environment variables is convenient, but not secure - consider a more
secure solution such as [Cloud KMS](https://cloud.google.com/kms/) to help keep secrets safe.


---

  It is recommended to use the [Secret Manager integration](https://cloud.google.com/run/docs/configuring/secrets) 
  for Cloud Run instead of using environment variables for the SQL configuration. The service injects the SQL 
  credentials from Secret Manager at runtime via an environment variable.

  Create secrets via the command line:
  ```sh
  echo -n "my-awesome-project:us-central1:my-cloud-sql-instance" | \
      gcloud secrets versions add INSTANCE_CONNECTION_NAME_SECRET --data-file=-
  ```

  Deploy the service to Cloud Run specifying the env var name and secret name:
  ```sh
  gcloud beta run deploy SERVICE --image gcr.io/[YOUR_PROJECT_ID]/run-sql \
      --add-cloudsql-instances [INSTANCE_CONNECTION_NAME] \
      --update-secrets INSTANCE_CONNECTION_NAME=[INSTANCE_CONNECTION_NAME_SECRET]:latest,\
        DB_USER=[DB_USER_SECRET]:latest, \
        DB_PASS=[DB_PASS_SECRET]:latest, \
        DB_NAME=[DB_NAME_SECRET]:latest
  ```


## Relevant Documentation

- https://cloud.google.com/sql/docs/postgres/quickstart-cloud-run
- https://cloud.google.com/sql/docs/postgres/connect-run
- https://cloud.google.com/vpc/docs/configure-serverless-vpc-access
- https://cloud.google.com/sql/docs/postgres/private-ip