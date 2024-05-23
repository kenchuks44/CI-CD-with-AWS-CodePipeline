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





























































