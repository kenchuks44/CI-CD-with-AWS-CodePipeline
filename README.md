# CI-CD-with-AWS-CodePipeline

Creating this project to demonstrate how to setup a Continuous Integration and Continuous Deployment pipeline using AWS CodePipeline to automate the delivery process of a React Nodejs Application.

The steps to accomplish this will involve implementing a full code pipeline incorportating commit, build and deploy steps. Firstly, we will configure Security & Create a CodeCommit Repo. Next, we configure CodeBuild to clone the repo, create a container image and store on ECR. Thereafter, we configure a CodePipeline with commit and build steps to automate build on commit. Finally, we create an ECS Cluster, Target Group , Application Load Balancer and configure the CodePipeline for deployment to ECS Fargate.

Below is how the target architecture will look like

![Arch6](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/817c6f36-f1d0-4ce5-8c05-a9b9a8d1f968)

## AWS CodePipeline
AWS CodePipeline is a fully managed continuous delivery service that helps you automate your release pipeline. It allows users to build, test and deploy code into a test or production environment.

## AWS CodeCommit
AWS CodeCommit is a fully-managed source control service that hosts secure Git-based repositories. It makes it easy for teams to collaborate on code in a secure and highly scalable ecosystem.

## AWS CodeBuild
AWS CodeBuild is a fully managed build service that compiles source code, runs tests, and produces software packages that are ready to deploy. With CodeBuild, you do not need to worry about provisioning and managing your own build infrastructure.

## Elastic Container Registry
AWS Elastic Container Registry (ECR) is a fully managed Docker container registry service. It allows us to easily store, manage, and deploy Docker container images. ECR seamlessly integrates with other AWS services, such as Amazon ECS (Elastic Container Service), Amazon EKS (Elastic Kubernetes Service), and AWS Fargate, making it a powerful tool for container-based applications.

## Elastic Container Service
Amazon Elastic Container Service (ECS), also known as Amazon EC2 Container Service, is a managed service that allows users to run Docker-based applications packaged as containers across a cluster of EC2 instances.

## AWS Fargate
AWS Fargate is a compute engine for Amazon Elastic Container Service(ECS) that allows us to run containers without having to provision, configure & scale clusters of VMs that host container applications. AWS Fargate eliminates the need for users to manage the EC2 instances on their own.

## Step 1: Setup CodeCommit Repo as well as Authentication
Firstly, we generate an SSH key which will be used to authenticate with CodeCommit using the command below:
```
ssh-keygen -t rsa -b 4096 -C "<your-email.com>"
```
Then, we copy the public key using the command below:
```
cat ~/.ssh/id_rsa.pub
```

Next, we go to the `Security Credentials` tab of AWS account. Then under the AWS Codecommit section, we upload the SSH key and copy the generated SSH key ID.
Then, from the terminal, we run `vi ~/.ssh/config` and add the following to the config file:
```
Host git-codecommit.*.amazonaws.com
  User <SSH_KEY_ID>
  IdentityFile ~/.ssh/id_rsa
```
![Screenshot (766)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/b24cdcfe-f8fb-48b6-8da5-5a77043521b3)

![Screenshot (767)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/82e11ec7-2c2b-4a6b-85a9-f3aaaae18963)

![Screenshot (768)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/90aa7bab-ea26-4702-bb09-ca10a2815d2e)

![Screenshot (770)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/ed2085bd-c216-4736-ae40-34312f88ebb8)

Next, we create a CodeCommit repository and clone the repository. Thereafter, we move the application source code into the cloned repo directory and then push to CodeCommit.

![Screenshot (771)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/582e7ea4-19c8-4530-a59e-21c1862d88cf)

![Screenshot (772)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/4359422d-31ae-4d6b-89a5-4dd7bf27b4d9)

![Screenshot (773)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/50f9aba6-4fe8-456e-b251-f90b51dde2a6)

![Screenshot (774)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/15c77aa0-6ee1-411e-bf6a-baf3dc62c771)

![Screenshot (776)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/bf8c93bd-ae12-4801-bb4f-0e33cd8b63ba)

![Screenshot (777)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/1a50b1f8-9b20-41d4-bcce-c871abe201de)

![Screenshot (778)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/3fbe0319-922a-4bd9-bee8-77b07bb363cc)

## Step 2: Setting up Private Repository in ECR and a CodeBuild Project
In this step, we will configure the Elastic Container Registry and use the codebuild service to build a docker container which will be pushed into the registry. Firstly, we will create the private repo. This is the repository that codebuild will store the docker image in, created from the codecommit repo.

![Screenshot (779)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/0c108340-1972-44e6-a9dd-f0f27498f9a5)

![Screenshot (785)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/50e0756e-4ad5-4c72-99e7-62ba2d9318fd)

![Screenshot (786)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/a09f5c16-5d11-4dff-8b3d-0f7d4916b089)

Next, we will configure a codebuild project which we utilize the files in the codecommit repo, build a docker image & store it within ECR in the above repository.

![Screenshot (787)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/e502ade0-37a6-462e-82c2-6d270e2122f3)

For the CodeBuild configuration:
For Source Provider, we will select `AWS CodeCommit`. Then, for Repository, we choose the repo we created in CodeCommit. For the Branch, we will select the branch from the Branch dropdown.

For `Environment` image, we select `Managed Image`. Under Operating system, we will select Amazon Linux 2. Then, for `Runtime(s)`, we select `Standard` and under `Image` select `aws/codebuild/amazonlinux2-x86_64-standard:X.0` where X is the highest number. Under Image version, Always use the latest image for this runtime version. Next, for `Envrironment` type, select Linux. Check the Privileged box (because we're creating a docker image). For Service role, select New Service Role and leave the default suggested name.

![Screenshot (788)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/47aff2fa-071b-41ee-bb2e-f9782ef3e881)

![Screenshot (789)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/63489de1-a048-4c38-9bd1-71ed96b713f0)

![Screenshot (790)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/0b47db1c-a0e2-4b66-934e-1516b7a27b85)

![Screenshot (791)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/d09f6075-eb47-49eb-bcce-8a719215fe05)

![Screenshot (792)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/a0c0e354-1aac-4c6a-8982-0ac6793a47c3)

![Screenshot (793)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/bc4554b2-9e40-4011-b21b-0d7c2a4b93c6)

Next, we add environmental variables
```
AWS_DEFAULT_REGION with a value of us-east-1
AWS_ACCOUNT_ID with a value of your account ID
IMAGE_TAG with a value of latest
IMAGE_REPO_NAME with a value of the name of ECR private repository created
```
![Screenshot (794)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/1eb82ba1-bf2c-429b-b877-b395d4e8c494)

Next, we select the buildspec.yml file. The buildspec.yml file tells codebuild how to build your code. The steps involved, what the build needs, possible testing and what to do with the output (artifacts).

```
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...
      - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker image...
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
```

![Screenshot (795)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/dbe27871-aa1b-4f54-a367-768687c3241c)

Then, we create the build project.

![Screenshot (798)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/a207c80d-bbda-426b-baba-b1366b981332)

Our build project will be accessing ECR to store the docker image to be created, and we need to ensure it has the permissons to do that. The build process will use an IAM role created by CodeBuild, so we need to update the role permissions. To do this, we go to the IAM console, click on Roles. Then, we identify the `codebuild-reactapp-pipeline-build-service-role`, click on it. Then, we click the Permissions tab and add the inline policy below:
```
{
	"Statement": [
	  {
		"Action": [
		  "ecr:BatchCheckLayerAvailability",
		  "ecr:GetAuthorizationToken",
		  "ecr:PutImage",
		  "ecr:InitiateLayerUpload",
		  "ecr:UploadLayerPart",
		  "ecr:CompleteLayerUpload"
	    ],
		"Resource": "*",
		"Effect": "Allow"
	  }
	],
	"Version": "2012-10-17"
}
```

![Screenshot (799)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/42715496-c72a-4143-9404-5f659eb03088)

![Screenshot (800)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/c54799e4-658e-4a45-97b6-2091ddaef438)

![Screenshot (801)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/a9025a65-5c7d-4582-b947-ae8ed39b1059)

![Screenshot (802)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/225b540e-7bd5-47dd-b501-76e3a8c693c9)

![Screenshot (803)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/631c3517-052b-42fe-bad1-34ba8ff19911)

We then test the CodeBuild project by starting the build and monitoring the progress.

![Screenshot (804)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/24d4fbf8-205b-4cec-9955-a178176cc02e)

![Screenshot (805)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/6dc959c8-98a3-4e5e-a293-c4d6d74ce3e5)

![Screenshot (806)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/1fb0b4e8-d4f1-4497-81bf-c973dd067a4e)

![Screenshot (807)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/b1f9ff80-717d-4230-9a99-40db7a570562)

![Screenshot (808)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/0f9585a3-e044-48aa-b3ca-09a2768d915d)

We then check the private repository in ECR to see the docker image created and stored

![Screenshot (809)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/39896f35-0540-4f99-93c9-66206075082c)

## Step 3: Creating and Testing a CodePipeline
In this step, we will create a CodePipeline which will utilize CodeCommit and CodeBuild to create a continous build process. The aim is that every time a new commit is made to the CodeCommit repo, a new docker image is created and pushed to ECR. For now, we will skip the CodeDeploy to be implemented later. We create the pipeline, selecting CodeCommit as the source stage and Codebuild as the build stage and follow subsequent prompts accordingly. As already mentioned, we will skip the deploy stage for now and create the pipeline. The pipeline will go through an initial execution and then completes without any issues

![Screenshot (811)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/64d52dcb-b550-4404-8111-9547d5ba770f)

![Screenshot (812)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/de65fde7-6d47-49e8-b3f0-7fe277dc2286)

![Screenshot (813)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/86c0153f-aeb6-43e4-83c4-52b0502e7bdd)

![Screenshot (814)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/8aa1d5a2-f7bc-4f33-b19f-787c7bdd71c3)

![Screenshot (815)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/a0176775-aed7-4a3d-8e0c-a2907c46ae8c)

![Screenshot (816)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/4753d3e0-5d88-4908-89bf-cdeb81e7b72e)

![Screenshot (817)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/8ba61543-4d1b-44f3-b475-9ce805e0eab3)

![Screenshot (818)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/54e30a24-47af-4648-ae8b-dd54e24c792e)

![Screenshot (822)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/c188b01a-46b3-4c7d-a986-612bd81eaecb)

![Screenshot (823)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/da9854a4-d772-4ba8-a647-2fe2660440cf)

We then see the new image created and stored in ECR

![Screenshot (824)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/4c5d6ead-3ab2-45a1-9fd3-c333bc86dd3b)

We also see the artifacts generated and stored in S3

![Screenshot (826)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/a26547ab-d654-4c8a-aca6-2868254ca8e3)

To observe the pipeline triggered and generate a new version of the image in ECR automatically when a commit happens, we update the buildspec.yml file as below:
```
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
      - REPOSITORY_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - IMAGE_TAG=${COMMIT_HASH:=latest}
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...
      - docker build -t $REPOSITORY_URI:latest .
      - docker tag $REPOSITORY_URI:latest $REPOSITORY_URL:$IMAGE_TAG
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker image...
      - docker push $REPOSITORY_URI:latest
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - echo Writing image definitions file...
      - printf '[{"name":"%s","imageUri":"%s"}]' "$IMAGE_REPO_NAME" "$REPOSITORY_URI:$IMAGE_TAG" > imagedefinitions.json
artifacts:
  files: imagedefinitions.json
```

![Screenshot (831)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/c2b8d471-a07c-481b-a6a2-26286db0e132)

![Screenshot (832)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/1dfcb205-61b4-4c37-a7c7-c3bc400d0a15)

![Screenshot (833)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/c2ec25cc-d908-4ef6-ae15-f6154abedab0)

![Screenshot (834)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/d866584e-accf-4e9f-b0a2-67555f0699e5)

![Screenshot (835)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/c0c306c0-80b9-4b32-8a78-5237bd8c5725)

![Screenshot (836)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/d8cb43e1-cc12-4969-beae-7fd79adae920)

![Screenshot (837)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/de205bc1-e24a-4229-be7c-3f1b3dbaaa79)

## Step 4: Implementing the deploy stage
In this step, we will configure automated deployment of the reactapp application to ECS Fargate
Firstly, we will create a load balancer which will be the entry point for the containerised application (here an application load balancer)

![Screenshot (840)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/5373c57a-0c7d-47c3-b445-17eb0e6607c4)

![Screenshot (841)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/b09be222-12dc-4b88-b68d-caea7483462f)

![Screenshot (842)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/f15610bb-40af-4d3b-a65b-3aa28b44bffb)

With an Internet Facing IPv4 For network, we select the default VPC and select all subnets in the VPC

![Screenshot (843)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/433503a9-feb3-46fd-9662-f7eda31f5b65)

![Screenshot (844)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/b52f81ba-00d3-43dd-a2b1-f94008cd6b18)

![Screenshot (858)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/2d5f8152-d716-4e20-9dfd-5cc100aa0331)

Next, we create a security group

![Screenshot (845)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/eb5e0a8b-fc8e-4dd9-acea-f93ab3173f48)

![Screenshot (846)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/28aa0f59-38b5-4782-b22f-6b8ea3194614)

![Screenshot (846)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/9b9a7235-ba82-42d9-9a2e-d2974bb5955a)

Next, we create a target group. A target group is used to route requests to one or more registered targets, such as EC2 instances, IP addresses, or Lambda functions. Target groups are essential for managing and distributing incoming traffic efficiently to ensure that applications are scalable and highly available. This target group is going to be pointing to containers running in the ECS Fargate( hence, target type of IP selected). Note that for now, no target would be registered.

![Screenshot (849)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/324550d8-df41-4fa0-8628-9cd7ac947b37)

![Screenshot (850)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/231485ea-5d11-43e9-9db4-4ee4df07d74d)

![Screenshot (851)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/c7a01a5f-906c-456c-bd03-2d12062e1628)

![Screenshot (853)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/26ab7a08-2ebd-453d-a602-7855cdba8e22)

![Screenshot (852)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/5a03ef0e-4982-4418-8158-8304f02e9cd1)

Next, we setup a Fargate cluster

![Screenshot (854)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/cf7ec50d-240d-4651-a82a-a41d34b0d2a0)

![Screenshot (854)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/990edf59-f8a2-456d-a3d8-97c9ab87fcd6)

![Screenshot (856)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/61513ab2-78ab-459a-8fc3-3563364cbb03)

![Screenshot (857)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/6a85eb09-4b15-4404-a266-812264ff6661)

![Screenshot (859)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/11435f67-b64c-4e03-ae90-6af676f50e73)

Next, we create a task and container definition.

## Task definition
A task definition is a blueprint of our application. It is a text file, in JSON format, that describes one or more containers that form your application.

## Task
A task is the instantiation of a task definition within a cluster. We have the option to specify the number of tasks that will run on your cluster.

## Cluster
A cluster is basically the logical grouping of resources that our application needs. If we use Fargate launch type with tasks within clusters, then Amazon ECS manages our cluster resources. If we use EC2 launch type, then our clusters will be a group of Amazon EC2 container instances that we manage.

Before proceeding to create the task definition, we create an ECS Task Execution Role. ECS (Elastic Container Service) Task Execution Role grants the Amazon ECS container agent permissions to make API calls on our behalf. These permissions are necessary for the agent to pull container images, log data to Amazon CloudWatch, and manage other AWS services used by your ECS tasks.

![Screenshot (861)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/4fb597ba-387c-491d-99fc-80f56592e003)

![Screenshot (862)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/ad3a9f5d-3cd2-4c3e-b0aa-ef769d8f7696)

![Screenshot (863)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/b5b3872e-65d2-4387-b927-823358b8ed77)

![Screenshot (864)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/d6a8bfd5-82fd-4332-8f59-426165f5a9c6)

![Screenshot (865)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/4702d78a-4cfe-4b16-bb6f-f0d570bbb41d)

Now, we proceed to the task definition creation:

![Screenshot (865)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/9917597b-9c73-4d8f-b202-0665542150fb)

![Screenshot (866)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/d6bf83c7-e48a-4186-aa12-54d99b69582f)

![Screenshot (868)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/32906e57-c88e-477d-a3b1-aaee90000071)

![Screenshot (869)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/98ff2e9d-6e56-4dd8-a3ec-f9eb5171439c)

![Screenshot (915)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/56a8b8d6-418c-4cad-98e3-1da9cf593e15)

![Screenshot (916)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/8ce14df2-f264-40b5-be52-3be0f9492e43)

Note that we can use different task definition revisions to specify different containers in Amazon ECS (Elastic Container Service). Task definitions in ECS are versioned, and each revision can define a different set of container configurations. This allows us to manage changes and updates to your application in a controlled manner.

Next, we create a service. The service manages the task definitions and ensures the specified number of tasks are running and healthy. The task definition is deployed using a service.

![Screenshot (872)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/7d930ae1-7316-418c-8449-85a646c0d294)

![Screenshot (873)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/77dc5b40-fe6a-4bf1-8262-1827b04c4c78)

![Screenshot (875)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/d36fe60d-eb92-484e-96aa-64936b0745ed)

![Screenshot (876)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/1cd354e0-bccd-40c4-a1b4-fafac97fd3b2)

![Screenshot (877)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/116d1284-449f-40f0-a7e6-48d207bff89f)

![Screenshot (878)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/e9a569a9-99c0-46c6-b124-ce866687c57e)

![Screenshot (879)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/79180ca8-c1b7-492d-8dc0-f7bb9220e4a1)

![Screenshot (880)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/8eebe210-3dd7-40dc-be08-14595c62ee0f)

![Screenshot (912)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/42ca022d-3ab7-4a35-aeb5-74a033d50865)

![Screenshot (913)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/ab3532a5-e08d-4fd6-82d0-c2f66f91b6ec)

![Screenshot (914)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/9ef05525-9cd6-400f-9cec-2c548fb64050)

With the service deployed successfully and running with the latest version of the container on ECR, we now test the container application by copying the LB DNS name and accessing the application via a web browser

![Screenshot (885)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/d96908c1-f6fd-41d3-a89d-b61c9a03685e)

![Screenshot (886)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/c623b085-f7a2-4e07-9d6b-ff7866028928)

After this manual deployment, we then add a deploy stage after the build stage to the pipeline. This will enable deployment whenever there is a commit in the CodeCommit repository.

![Screenshot (889)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/97d427f4-8b29-4c2d-a050-294a43a5dde2)

![Screenshot (888)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/7c39eeba-7307-4bde-ab0a-138b96850db2)

![Screenshot (890)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/b7a27049-9329-4685-8c91-c3d4d0f8b2ad)

We then add an action group. For Action Provider, we select Amazon ECS. For Input artifacts, we select Build Artifact (this will be the imagedefinitions.json information about the container) and for Image Definitions file, we input imagedefinitions.json. We then save and confirm.

![Screenshot (894)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/f0824e18-d358-4b75-b0ef-d7b0ed39aa6a)

![Screenshot (910)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/b7cc4140-677a-4cc9-b0a3-7c45c4e93561)

![Screenshot (911)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/e89a5418-f6ba-4654-96f6-5700c6ed30d0)

![Screenshot (895)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/e0b506dd-ff4c-4563-9380-18055993cc7e)

Now, to test automation of the full deployment pipeline, we edit the Header.js by adding three dots to the end of the `h1` line text and commit the changes

![Screenshot (897)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/95d14f09-fd8e-42c2-964d-a7642009f892)

![Screenshot (899)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/ff5adc3d-d744-40ee-a619-fd6523c2d830)

We then observe the pipeline triggered

![Screenshot (900)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/2a5cda7f-ad8a-41db-a581-edf68ab5b61e)

![Screenshot (902)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/d3b28353-b43c-4e51-914c-b99991236b71)

![Screenshot (903)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/cbcebec4-14dd-49aa-9ac0-8b28a8dc2984)

![Screenshot (904)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/d9588608-f0de-4755-acb4-64d15f60ab3f)

![Screenshot (905)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/31142df0-da51-46f8-a9d6-ec8b78749064)

We can see the task using the new docker image ans the task is registered in the target group.

![Screenshot (906)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/6490cc39-3355-4daf-a969-26d1173853df)

![Screenshot (907)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/44a92f00-5a25-4784-a9c9-4fada3a0a19b)

![Screenshot (908)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/58ec6ae0-b540-4f8c-bed8-7100e3d7635d)

We can as well see the artifacts stored in S3 bucket and the docker image in the container registry.

![Screenshot (918)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/06989da1-837f-4f04-9e30-c1864f18fbc3)

![Screenshot (919)](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/ca67db67-a79f-4c75-bb3b-c4d4ed9b01af)

Hence, we have a full automated deployment by CodePipeline.

Congratulations!!!











































































































