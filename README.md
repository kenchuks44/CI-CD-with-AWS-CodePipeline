# CI-CD-with-AWS-CodePipeline

Creating this project to demonstrate how to setup a Continuous Integration and Continuous Deployment pipeline using AWS CodePipeline to automate the delivery process of a React Nodejs Application.

The steps to accomplish this will involve implementing a full code pipeline incorportating commit, build and deploy steps. Firstly, we will configure Security & Create a CodeCommit Repo. Next, we configure CodeBuild to clone the repo, create a container image and store on ECR. Thereafter, we configure a CodePipeline with commit and build steps to automate build on commit. Finally, we create an ECS Cluster, Target Group , Application Load Balancer and configure the CodePipeline for deployment to ECS Fargate.

Below is how the target architecture will look like

![Arch6](https://github.com/kenchuks44/CI-CD-with-AWS-CodePipeline/assets/88329191/817c6f36-f1d0-4ce5-8c05-a9b9a8d1f968)
