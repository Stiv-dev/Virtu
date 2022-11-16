# Part 4 - Container Orchestration with Kubernetes

## Prerequisites

You will need to create a kubernetes cluster with 3 nodes. 1 master node and 2 worker nodes.

### kubectl

You will need to use `kubectl`. Verify that you have the `kubectl` utility installed locally by running the following command:

```bash
kubectl version --short
```

This should print a response with a `Client Version` if it's successful.

#### Verify Cluster and Connection

Once `kubectl` is configured to communicate with your EKS cluster, run the following to validate that it's working:

```bash
kubectl get nodes
```

This should return information regarding the nodes that were created in your EKS clusteer. make sure nodes are ready

### Deployment

In this step, you will deploy the Docker containers for the frontend web application and backend API applications in their respective pods.

The environment variables, as well as AWS CLI configured locally are required while instantiating containers from the Dockerhub images.

1. **ConfigMap:** Create `env-configmap.yaml`, and save all your configuration values (non-confidential environments variables) in that file.

2. **Secret:** Do not store the PostgreSQL username and passwords in the `env-configmap.yaml` file. Instead, create `env-secret.yaml` file to store the confidential values, such as login credentials. Unlike the AWS credentials, these values do not need to be Base64 encoded.

3. **Secret:** Create _aws-secret.yaml_ file to store your AWS login credentials. Replace `___INSERT_AWS_CREDENTIALS_FILE__BASE64____` with the Base64 encoded credentials (not the regular username/password).
   - Mac/Linux users: If you've configured your AWS CLI locally, check the contents of `~/.aws/credentials` file using `cat ~/.aws/credentials` . It will display the `aws_access_key_id` and `aws_secret_access_key` for your AWS profile(s). Now, you need to select the applicable pair of `aws_access_key` from the output of the `cat` command above and convert that string into `base64` . You use commands, such as:

```bash
# Use a combination of head/tail command to identify lines you want to convert to base64
# You just need two correct lines: a right pair of aws_access_key_id and aws_secret_access_key
cat ~/.aws/credentials | tail -n 5 | head -n 2
# Convert
cat ~/.aws/credentials | tail -n 5 | head -n 2 | base64
```

     * **Windows users:** Copy a pair of *aws_access_key* from the AWS credential file and paste it into the encoding field of this third-party website: https://www.base64encode.org/ (or any other). Encode and copy/paste the result back into the *aws-secret.yaml*  file.

4. **volumes** mount volumes for your aws credentials to ~/.aws folder inside the container
   <br data-md>

5. **Deployment configuration:** Create _deployment.yaml_ file individually for each service. While defining container specs, make sure to specify the same images you've pushed to the Dockerhub earlier. Ultimately, the frontend web application and backend API applications should run in their respective pods.

6. **Service configuration: **Similarly, create the _service.yaml_ file thereby defining the right services/ports mapping.

Once, all deployment and service files are ready, you can use commands like:

```bash
# Apply env variables and secrets
kubectl apply -f aws-secret.yaml
kubectl apply -f env-secret.yaml
kubectl apply -f env-configmap.yaml
# Deployments - Double check the Dockerhub image name and version in the deployment files
kubectl apply -f backend-feed-deployment.yaml
# Do the same for other three deployment files
# Service
kubectl apply -f backend-feed-service.yaml
# Do the same for other three service files
```

Make sure to check the image names in the deployment files above.

## Connecting k8s services to access the application

If the deployment is successful, and services are created, there are two options to access the application:

1. If you deployed the services as CLUSTERIP, then you will have to [forward a local port to a port on the "frontend" Pod](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/#forward-a-local-port-to-a-port-on-the-pod). In this case, you don't need to change the URL variable locally.

2. If you exposed the "frontend" deployment using a Load Balancer's External IP, then you'll have to update the URL environment variable locally, and re-deploy the images with updated env variables.

Below, we have explained method #2, as mentioned above.

### Expose External IP

Use this link to <a href="https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/" target="_blank">expose an External IP</a> address to access your application in the EKS Cluster.

```bash
# Check the deployment names and their pod status
kubectl get deployments
# Check created servcices for all your deployments
kubectl get services
```

### Update the environment variables

Once you have the External IP of your front end and reverseproxy deployment, Change the API endpoints in the following places locally:

- _udagram-deployment/env-configmap.yaml_ file - Replace http://localhost:8100 string with the Cluster IP of the _frontend_.

- _udagram-frontend/src/environments/environment.ts_ file - Replace 'http://localhost:8080/api/v0' string with either the Cluster IP of the _reverseproxy_ deployment.

- _udagram-frontend/src/environments/environment.prod.ts_ - Replace 'http://localhost:8080/api/v0' string.

Then, push your changes to the Github repo.

```bash
kubectl apply -f env-configmap.yaml
# Rolling update "frontend" containers of "frontend" deployment, updating the image
kubectl set image deployment frontend frontend=[your_dockerhub_username]/udagram-frontend:v3
# Do the same for other three deployments
```

Check your deployed application at the http://localhost:8100 .

## Troubleshoot

1. Use this command to see the STATUS of your pods:

```bash
kubectl get pods
kubectl describe pod <pod-id>
# An example:
# kubectl logs backend-user-5667798847-knvqz
# Error from server (BadRequest): container "backend-user" in pod "backend-user-5667798847-knvqz" is waiting to start: trying and failing to pull image
```

In case of `ImagePullBackOff` or `ErrImagePull` or `CrashLoopBackOff`, review your deployment.yaml file(s) if they have the right image path.

2. Look at what's there inside the running container. [Open a Shell to a running container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/) as:

```bash
kubectl get pods
# Assuming "backend-feed-68d5c9fdd6-dkg8c" is a pod
kubectl exec --stdin --tty backend-feed-68d5c9fdd6-dkg8c -- /bin/bash
# See what values are set for environment variables in the container
printenv | grep POST
# Or, you can try "curl <cluster-IP-of-backend>:8080/api/v0/feed " to check if services are running.
# This is helpful to see is backend is working by opening a bash into the frontend container
```

3. When you are sure that all pods are running successfully, then use developer tools in the browser to see the precise reason for the error.

- If your frontend is loading properly, and showing _Error: Uncaught (in promise): HttpErrorResponse: {"headers":{"normalizedNames":{},"lazyUpdate":null,"headers":{}},"status":0,"statusText":"Unknown Error"...._, it is possibly because the _udagram-frontend/src/environments/environment.ts_ file has incorrectly defined the ‘apiHost’ to whom forward the requests.
- If your frontend is **not** not loading, and showing _Error: Uncaught (in promise): HttpErrorResponse: {"headers":{"normalizedNames":{},"lazyUpdate":null,"headers":{}},"status":0,"statusText":"Unknown Error", ...._ , it is possibly because URL variable is not set correctly.
- In the case of _Failed to load resource: net::ERR_CONNECTION_REFUSED_ error as well, it is possibly because the URL variable is not set correctly.

## Screenshots

So that I can verify that your project is deployed, please include the screenshots of the following commands with your completed project.

```bash
# Kubernetes deployments
kubectl get deploy
# Kubernetes pods are deployed properly
kubectl get pods
# Kubernetes services are set up properly
kubectl describe services
```
