# Here's a step-by-step guide for setting up and demonstrating GWI with GKE:

Setting up and demonstrating Google Workload Identity (GWI) for use with Google Kubernetes Engine (GKE) involves several steps. GWI allows you to securely authenticate and authorize workloads running on GKE clusters to access Google Cloud Platform (GCP) services without needing to manage service account keys. 

**Prerequisites**:
1. A Google Cloud Platform (GCP) account with billing enabled.
2. `gcloud` command-line tool installed and authenticated with your GCP account.
3. A GKE cluster created and configured.

**Step 1: Enable Workload Identity on Your GKE Cluster**
   
   Run the following command to enable GWI for your GKE cluster:
   ```shell
   gcloud container clusters update CLUSTER_NAME --workload-pool=PROJECT_ID.svc.id.goog --zone=ZONE
   ```
   Replace `CLUSTER_NAME`, `PROJECT_ID`, and `ZONE` with your specific values.

**Step 2: Create a Kubernetes Service Account and Associate It with a Google Cloud IAM Service Account**
   
   Create a Kubernetes service account and associate it with a Google Cloud IAM service account. This is where GWI comes into play, as it maps your Kubernetes service account to a Google Cloud IAM service account for secure access.
   ```shell
   kubectl create sa YOUR_K8S_SA_NAME
   gcloud iam service-accounts add-iam-policy-binding YOUR_GCLOUD_SA_NAME@PROJECT_ID.iam.gserviceaccount.com \
     --member="serviceAccount:PROJECT_ID.svc.id.goog[NAMESPACE/YOUR_K8S_SA_NAME]" \
     --role="roles/iam.workloadIdentityUser"
   ```

**Step 3: Annotate Your Kubernetes Pods**

   You need to annotate your Kubernetes pods to specify which service account to use. Add the following annotation to your pod YAML:

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: YOUR_POD_NAME
     annotations:
       iam.gke.io/gcp-service-account: YOUR_GCLOUD_SA_NAME@PROJECT_ID.iam.gserviceaccount.com
   spec:
     serviceAccountName: YOUR_K8S_SA_NAME
     # Your pod configuration...
   ```

   Replace `YOUR_POD_NAME`, `YOUR_GCLOUD_SA_NAME`, `PROJECT_ID`, `YOUR_K8S_SA_NAME`, and pod configuration as needed.

**Step 4: Deploy and Test Your Application**

   Deploy your Kubernetes application with the annotated pod. This will use the mapped Google Cloud IAM service account for authorization.

   ```shell
   kubectl apply -f YOUR_POD_CONFIG.yaml
   ```

**Step 5: Verify Access to GCP Services**

   To verify that your application can access GCP services using the mapped service account, you can run a simple test. For example, if your application needs to access Google Cloud Storage, you can create a bucket and list its contents to ensure it's working correctly:

   ```shell
   gsutil mb -p PROJECT_ID -c STANDARD -l REGION gs://YOUR_BUCKET_NAME
   gsutil ls gs://YOUR_BUCKET_NAME
   ```

   If your application can list the contents without any authentication issues, it means GWI is successfully configured.

**Step 6: Cleanup**

   After you've verified that everything works as expected, don't forget to delete any resources you no longer need to avoid unnecessary charges.

This guide should help you set up and demonstrate Google Workload Identity for use with GKE. Remember to replace placeholders with your actual values. Additionally, it's crucial to understand the permissions and roles associated with your Google Cloud IAM service account to ensure secure access to GCP services.
