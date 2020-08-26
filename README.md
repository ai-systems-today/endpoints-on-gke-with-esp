# Endpoints on GKE with ESP

This tutorial is referenced from: https://cloud.google.com/endpoints/docs/openapi/get-started-kubernetes-engine on Cloud Endpoints with OpenAPI.

## Objectives

Prerequisite: Have a Google Cloud Platform (GCP) account.

1. Set up a Google Cloud project.
2. Create a container cluster on Google Kubernetes Engine (GKE).
3. Install required software.
4. Download the sample code.
5. Configure Cloud Endpoints.
6. Deploy the configuration to create an Endpoints service (API).
7. Deploy the API and ESP to the cluster.
8. Get the cluster&#39;s External IP address.
9. Send a request to the API by using an IP address.
10. Track API activity.
11. Configure a DNS record for the sample API.
12. Send a request to the API using the fully qualified domain name (FQDN).
13. Clean up to avoid incurring charges to your GCP account.

## Set up a Google Cloud project

1. In the Cloud Console, select or create a Cloud project.
2. Make sure that billing is enabled for your Google Cloud project.
3. Make a note of the Google Cloud project ID because it is needed later.

## Create a container cluster on GKE

1. In the Google Cloud Console, go to the GKE clusters page.
2. Click  **Create cluster**.
3. Accept the defaults and click  **Create**.

	This step can take a few minutes to complete.

	Make a note the cluster name and zone because they are needed when you authenticate [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview) to the container cluster.

## Install required software

If you already have the required software installed, continue with the next step.

1. Install curl
2. Install and initialize the Cloud SDK.

	Update the Cloud SDK and install the Endpoints components:

	>gcloud components update

	Make sure that the Cloud SDK (gcloud) is authorized to access your data and services on Google Cloud:

	>gcloud auth login

	In the new browser tab that opens, select an account. Set the default project to your project ID:

	>gcloud config set project _ **YOUR\_PROJECT\_ID** _

	Replace _ **YOUR\_PROJECT\_ID** _ with your project ID.

1. Install kubectl:

	>gcloud components install kubectl

	Acquire new user credentials to use for Application Default Credentials. The user credentials are needed to authorize kubectl.

	>gcloud auth application-default login

	In the new browser tab that opens, select an account.

## Download the sample code

Clone the sample app repository to your local machine:

	>git clone https://github.com/ai-systems-today/endpoints-on-gke-with-esp

## Configure Endpoints

In the sample code directory, open the openapi.yaml configuration file.

	>swagger: &quot;2.0&quot;
	>info:
	>  description: &quot;A simple Google Cloud Endpoints API example.&quot;
	>   title: &quot;Endpoints Example&quot;
	>   version: &quot;1.0.0&quot;
	>host: &quot;echo-api.endpoints.YOUR-PROJECT-ID.cloud.goog&quot;

	In the host field, replace _ **YOUR\_PROJECT\_ID** _ with your Google Cloud project ID, which should be in the following format:

	>host: &quot;echo-api.endpoints._ **YOUR\_PROJECT\_ID** _.cloud.goog&quot;

## Deploy the Endpoints configuration

1. Upload the configuration and create a managed service:

	>gcloud endpoints services deploy openapi.yaml

	When it finishes configuring the service, Service Management displays a message with the service configuration ID and the service name, similar to the following:

	Service Configuration [2017-02-13r0] uploaded for service[echo-api.endpoints.example-project-12345.cloud.goog]

## Check required services

At a minimum, Endpoints and ESP require the following Google services to be enabled:

| Name | Title |
| --- | --- |
| servicemanagement.googleapis.com | Service Management API |
| servicecontrol.googleapis.com | Service Control API |
| endpoints.googleapis.com | Google Cloud Endpoints |

1. Use the following command to confirm that the required services are enabled:

	>gcloud services list

2. If you do not see the required services listed, enable them:

	>gcloud services enable servicemanagement.googleapis.com

	>gcloud services enable servicecontrol.googleapis.com

	>gcloud services enable endpoints.googleapis.com

3. Also enable your Endpoints service:

	>gcloud services enable _ **ENDPOINTS\_SERVICE\_NAME**

	For OpenAPI, the _ **ENDPOINTS\_SERVICE\_NAME** is what you specified in the host field of your OpenAPI spec.

## Deploy the API backend

Deploy prebuilt containers for the sample API and ESP to the cluster.

### Check required permissions

**Note: ** This step is not needed if the GKE cluster is deployed with default service

In the Cloud Console, go to the Kubernetes clusters page.

1. Select your cluster from the list.
2. Click  **Permissions**  to see the service account associated with the cluster.
3. Grant required permissions to the service account:

	>gcloud projects add-iam-policy-binding _ **PROJECT\_NAME** _
	>--member &quot;serviceAccount:_ **SERVICE\_ACCOUNT** _&quot;
	>--role roles/servicemanagement.serviceController

### Deploy the containers to the cluster

1. Get cluster credentials and make them available to kubectl:

	>gcloud container clusters get-credentials _ **NAME** _ --zone _ **ZONE** _

	Replace _ **NAME** _ with the cluster name and _ **ZONE** _ with the cluster zone.

2. Open deployment.yaml and change:

	>name: esp
	>  image: gcr.io/endpoints-release/endpoints-runtime:1
	>  args: [
	>	--http\_port=8081&quot;,
	>	--backend=127.0.0.1:8080&quot;,
	>	--service=SERVICE\_NAME&quot;,
	>	--rollout\_strategy=managed&quot;,
	>   ]
	
	Replace _ **SERVICE\_NAME** in the ESP startup options with the name of your service.

3. Start the Kubernetes service using the:

	>kubectl apply -f deployment.yaml

## Get the cluster&#39;s external IP address

1. View the external IP address:

	>kubectl get service

2. Make a note of the value for EXTERNAL-IP. You use that IP address when you send a request to the sample API.

## Send a request by using an IP address

Create an API key and set an environment variable

1. In the same Google Cloud project that you used for your API, create an API key on the API credentials page.
2. Click  **Create credentials** , and then select  **API key**.
3. Copy the key to the clipboard.
4. Click  **Close**.
5. On your local computer, paste the API key to assign it to an environment variable:

	In Linux or macOS: 
	
	>export ENDPOINTS\_KEY=AIza...

	In Windows PowerShell: 
	
	>$Env:ENDPOINTS\_KEY=&quot;AIza...&quot;

## Send the request

Use curl to send an HTTP request by using the **ENDPOINTS\_KEY** environment variable you set previously.

1. Replace **IP\_ADDRESS** with the external IP address of your instance.

	>curl --request POST \
	>	--header &quot;content-type:application/json&quot; \
	>	--data &#39;{&quot;message&quot;:&quot;hello world&quot;}&#39; \
	>	&quot;http://_ **IP\_ADDRESS** _:80/echo?key=${ENDPOINTS\_KEY}&quot;

2. The API echoes back the message that you send, and responds with the following:

	>{
	>   &quot;message&quot;: &quot;hello world&quot;
	>}

	If you didn&#39;t get a successful response, see [Troubleshooting response errors](https://cloud.google.com/endpoints/docs/openapi/troubleshoot-response-errors).

## Track API activity

1. Look at the activity graphs for your API in the  **Endpoints**  \&gt;  **Services**  page.
2. Look at the request logs for your API in the Logs Viewer page.

## Configure DNS for Endpoints

1. Open your OpenAPI configuration file, openapi.yaml, and add the x-google-endpoints property at the top level of the file:

	>host: "echo-api.endpoints.**YOUR\_PROJECT\_ID**.cloud.goog"
	
	>x-google-endpoints:
	
	>  name: "echo-api.endpoints.**YOUR\_PROJECT\_ID**.cloud.goog"
	
	>  target:**IP\_ADDRESS**

	Replace **YOUR\_PROJECT\_ID** with your project ID.

	In the target property, replace **IP\_ADDRESS** with the IP address that you used when you sent a request to the sample API.

2. Deploy your updated OpenAPI configuration file to Service Management:

	>gcloud endpoints services deploy openapi.yaml

## Send a request by using FQDN

Now that you have the DNS record configured for the sample API, send a request to it by using the FQDN (replace _ **YOUR\_PROJECT\_ID** _ with your project ID) and the _ **ENDPOINTS\_KEY** _ environment variable set previously:

In Linux / mac OS:

	>curl --request POST \
	>	 --header &quot;content-type:application/json&quot; \
	>	 --data {'message':'hello world'} \
	>	 "http://echo-api.endpoints._ **YOUR\_PROJECT\_ID** .cloud.goog:80/echo?key=${ENDPOINTS\_KEY}"

## Clean up

	To avoid incurring charges to your Google Cloud Platform account for the resources used in this tutorial:

	See [Deleting an API and API instances](https://cloud.google.com/endpoints/docs/openapi/deleting-an-api-and-instances) for information on stopping the services used by this tutorial.