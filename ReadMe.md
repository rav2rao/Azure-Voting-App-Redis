## Task Summary:- 

This task is specifically about creating an Azure DevOps pipeline for a multi-container application which is
deployed into Azure Kubernetes Service (AKS).
   
This task uses a publicly available basic voting app consisting of a front-end web component and a backend
Redis instance. It does not require any programming effort. All source code for the service being built
can be found here: https://github.com/Azure-Samples/azure-voting-app-redis.

For components that are referenced in the pipeline (e.g. ACR, AKS), if you will not be demoing your
submission on the Azure platform, it is acceptable to assume that these components already exist. Please
use names that follow Azure naming standards.

## Task Deliverables: 
    4 files that make up 1 Azure DevOps Build pipeline which will deploy the voting app into an AKS
        cluster:
    -> File 1: Main pipeline
         Pipeline should have sensible split of stages that demonstrate the CI and CD activities
         Each stage may have 1 or more Jobs and Steps.
         Please use variables opposed to hard coded values, wherever suitable
    -> File 2: A CI Pipeline Stage
        This stage of the pipeline should be stored in a separate file and referenced within
        the main pipeline (i.e. file 1). To illustrate a DRY technique.
    -> File 3: A CD Pipeline Stage
         This stage of the pipeline should be stored in a separate file and referenced within
        the main pipeline (i.e. file 1)
    -> File 4: Variables
        Pass in variables to the main pipeline (i.e. file 1)

### Prerequisites

    -> Sign up an Azure account.
    -> Azure container Registry setup ACR -  so that we can push our images.
    -> Ensure you got an AKS cluster to deploy the manifest.
    -> Docker -  so that we can check the images are created and builty locally.

## Solution:

The multi-container application chosen is a simple voting app written in Python 3, with a Redis back-end to store voting data. 
    The following steps guide you through the process of deploying the web application using Azure DevOps.

## Azure Pipelines Breakdown:

The Azure DevOps pipeline is designed to automate the process of building, testing, publish and deploying the containers in AKS. 
The following files are created as part of task deliverables:

1. **azure-vote/azure-pipelines.yml**: The main pipeline file that orchestrates the entire CI/CD process.
2. **stages/ci.yaml**: Defines the CI stage that builds and pushes the Docker images to the Azure Container Registry (ACR).
3. **stages/cd.yaml**: Defines the CD stage that deploys the application into an Azure Kubernetes Service (AKS) cluster.
4. **variables/vars.yaml**: Contains variables used throughout the pipeline to parameterize resource names and avoid hard-coding values.

### Breif explaination of Key Steps in Azure Pipelines: 

1. **CI Pipeline stage (`ci.yaml`)**: In this created the stage into two one is Build and Push, Second is manifest generation.
   
   a. stage (build and push):- In this stage, created build job with multilpe tasks explained below

   - **login to ACR**: First task - is connecting using Azure Service Connection to the ACR, so that we can push the image into created ACR ""

   - **Image Build & Push**: Second task - Built and pushed the voting-app-front using Azure DevOps Docker task into ACR to use at later stages.
                             Third task - Built and pushed the voting-app-back (redis) using Azure DevOps Docker task into ACR to use at later stages.

   - **Image Versioning**: The image version is tagged using `$(Build.BuildId)` to ensure each build is unique.

   - **string parameterize**:Generated a string parameter "manifestName" in as suggested format using bash script.

   b. stage (Generate manifest job):- 

   - **Artifact Publishing**: A Kubernetes manifest file is generated and published as a build artifact for later deployment and assigned 
        it to a string variable as suggested and renamed the file with the new manifestName.

2. **CD Pipeline Stage (`cd.yaml`)**: Deploying the renamed manifest into AKS.

   - **Download manifest**:- In this step, Downloaded the generated manifestName from build artifact in the CI stage to deploy into AKS.

   - **login to Azure**: Task@AzureClI - In this task logged into Azure, to set up the AKS credentials.

   - **Image Pull Secrets**: Task@KubernetesManifest@0 generated `imagePullSecret` to allow the AKS cluster to access images from the private ACR.

   - **Deploy to AKS**: Task@KubernetesManifest@0- Deploys the both the containers voting-app-front & voting-app-back into AKS using the Kubernetes manifestName.
   
### Validation:- Validated the code by deplying localling: 

 ### Running Locally

To run the application locally using Docker Compose:

1. Ensure Docker is installed and running.
2. Run the following command:
   ```sh
   docker-compose up -d
   ```
3. Access the voting app in your browser at `http://localhost:8000`.

   #### Here are the screen shots of the verification and valition of the build images, running container and accessing the app

   -> showing the images are created and running
   ![image](https://github.com/user-attachments/assets/83e0127d-a8ab-4f05-ab3b-8cf5e1246095)

   -> Showing the containers are running in docker desktop
   ![image](https://github.com/user-attachments/assets/d3d2ecb1-3abe-4943-9d98-1eb40dd071d8)

   -> Redis is up and running
   ![image](https://github.com/user-attachments/assets/4591814a-b3f5-40c8-93ed-038f38b69698)

   -> voting-app-front is listening at local:8000 port

   ![image](https://github.com/user-attachments/assets/17131d1c-3931-4cfa-bf10-1215c0fa413f)

   -> Access the service at local:8000 and accessed
   ![image](https://github.com/user-attachments/assets/589cd44a-8949-4910-8392-870996d4c01e)






## Suggested Improvements:- 

- **Git Branch stratagies** -  Instead of deploying main, will suggest to deploy from feature branch to check any issues.
    - Implementing PR approval process.

- **Scan & Test** - Will add implement additional stages like - Secret scan, test cases, Code coverage, image scan using relevant tools.

- **Security Enhancements**: Implement security best practices, such as using Azure Key Vault to manage secrets.

    - Container Scanning using Trivy for finding vulnerabilities in the Docker Comtainer
    - ⁠SAST using Sonar Qube
    - ⁠SCA using OWASP Dependency Checker

- **Monitoring and Logging**: Integrate Azure Monitor and Azure Log Analytics to track application performance and errors.

- **Scalability**: Add horizontal pod autoscaling to the Kubernetes manifest to ensure the app scales based on demand.

Note: Followed Azure DevOps Docs as Reference links to create the Main, CI, CD and Variables pipelines for Azure DevOps Build pipeline.

### Template types & usage
https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops

### Build an image
https://docs.microsoft.com/en-us/azure/devops/pipelines/ecosystems/containers/build-image?view=azure-devops

### Define variables
https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch

### Kubernetes manifest task
https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/deploy/kubernetes-manifest?view=azure-devops

https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/kubernetes-manifest-v0?view=azure-pipelines&viewFallbackFrom=azure-devops

### Publish and download pipeline Artifacts 
https://docs.microsoft.com/en-us/azure/devops/pipelines/artifacts/pipeline-artifacts?view=azure-devops&tabs=yaml.

### License

This project is licensed under the terms specified in the LICENSE file.
