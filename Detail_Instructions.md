# Step by Step Instructions

## Create Cloud9 environment

> AWS Cloud9 is a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just a browser. It includes a code editor, debugger, and terminal. Cloud9 comes prepackaged with essential tools for popular programming languages, including JavaScript, Python, PHP, and more, so you don’t need to install files or configure your development machine to start new projects. Since your Cloud9 IDE is cloud-based, you can work on your projects from your office, home, or anywhere using an internet-connected machine. Cloud9 also provides a seamless experience for developing serverless applications enabling you to easily define resources, debug, and switch between local and remote execution of serverless applications. With Cloud9, you can quickly share your development environment with your team, enabling you to pair program and track each other's inputs in real time.

1. Go to the AWS Management Console, click **Services** then select [**Cloud9**](http://console.aws.amazon.com/cloud9/) under Developer Tools.
2. Click **Create environment**.
3. Enter **_BlueGreenEnvironment_** into **Name** and optionally provide a **Description**.
4. Click **Next step**.
5. You may leave **Environment settings** at their defaults of launching a new **t2.micro** EC2 instance which will be paused after **30 minutes** of inactivity.
6. Click **Next step**.
7. Review the environment settings and click **Create environment**. It will take several minutes for your environment to be provisioned and prepared.
8. Once ready, your IDE will open to a welcome screen. Below that, you should see a terminal prompt. You can run AWS CLI commands in here just like you would on your local computer. Verify that your user is logged in by running this command.

     ```console
     Admin:~/environment $ aws sts get-caller-identity
     ```

     ![Cloud9](./images/bg-1.png)

     We will be using Cloud9 IDE for our development.

## Configure Code Commit Git Credential Helper

In the terminal prompt, run the following commands to enable git to use the credential helper with the AWS credential profile.

```console
Admin:~/environment $ git config --global credential.helper '!aws codecommit credential-helper $@'
Admin:~/environment $ git config --global credential.UseHttpPath true
```

## Create an AWS CodeCommit Repository

1. Open the [AWS CodeCommit console](http://console.aws.amazon.com/codecommit)
2. On the Welcome page, choose Get Started Now. (If a **_Dashboard_** page appears instead, choose **_Create repository_**.)
3. On the **Create repository** page, in the **Repository name** box, type **_BlueGreenWebApp_**.
4. In the **Description** box, type **_BlueGreenWebApp repository_**.
5. Click **Create repository** to create an empty AWS CodeCommit repository.
6. On the **Connection Steps** screen, review Connection steps. We completed step 2 from the previous section. Select the **Clone URL** drop-down list to copy the Clone HTTPS command.
7. Go to your Cloud9 IDE.  In the terminal prompt paste clone command you previouly copied.

     ```console
     Admin:~/environment $ git clone <REPOSITORY_URL>
     ```

     ![Cloned Repo](./images/bg-2.png)

8. Configure Git user in Cloud9 environment.

     ```console
     Admin:~/environment $ git config --global user.email you@example.com
     Admin:~/environment $ git config --global user.name "Your Name"
     ```

9. Inside BlueGreenWebApp folder, download the Sample Web App Archive by running the following command from IDE terminal and unzip the archvie.

     ```console
     Admin:~/environment $ cd BlueGreenWebApp
     Admin:~/environment/BlueGreenWebApp (master) $ wget https://github.com/andreascg/aws-cicd-bluegreen/raw/master/WebApp.zip
     Admin:~/environment/BlueGreenWebApp (master) $ unzip WebApp.zip
     Admin:~/environment/BlueGreenWebApp (master) $ mv WebApp/* .
     Admin:~/environment/BlueGreenWebApp (master) $ rm -rf WebApp*
     ```

     Your IDE environment should look like this.

     ![Project](./images/bg-3.png)

10. Stage your change by running **_git add_**. You can use **_git status_** to review the changes.

     ```console
     Admin:~/environment/BlueGreenWebApp (master) $ git add .
     Admin:~/environment/BlueGreenWebApp (master) $ git status
     ```

11. Commit your change by running **_git commit_** to commit the change to the local repository then run **_git push_** to push your commit the default remote name Git uses for your AWS CodeCommit repository (origin). Enter your git credential.

     ```console
     Admin:~/environment/BlueGreenWebApp (master) $ git commit -m "Initial Commit"
     Admin:~/environment/BlueGreenWebApp (master) $ git push
     ```

     ![Project](./images/bg-4.png)

     **_💡 Tip_** After you have pushed files to your AWS CodeCommit repository, you can use the AWS CodeCommit console to view the contents. For more information, see [Browse the Contents of a Repository](http://docs.aws.amazon.com/codecommit/latest/userguide/how-to-browse.html).

## Create Infrastructure

In this step, we will be using CloudFormation template to create infrstructure used for this lab.  Review template.yml.

1. In Cloud9, create CloudFormation stack by running this command. If the command execute with no issues, you should see the StackId return back.

     ```console
     Admin:~/environment/BlueGreenWebApp (master) $ aws cloudformation create-stack --stack-name BlueGreenEnvironment --template-body file://template.yml --capabilities CAPABILITY_IAM
     ```

2. Go to [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation) to view its progress.  Once complete, go to Outputs Tab and observe the Cloudformation output value. Browse the URL of your ALB, in your favorite browser.

     ![ALB](./images/bg-11.png)

     **_💡 Tip_** While resources are being provisioned, go back to Cloud9 and inspect `BlueGreenWebApp/template.yml` to understand what resources are being spun up.

## Configure CodeBuild

> AWS CodeBuild is a fully managed continuous integration service that compiles source code, runs tests, and produces software packages that are ready to deploy. With CodeBuild, you don’t need to provision, manage, and scale your own build servers. CodeBuild scales continuously and processes multiple builds concurrently, so your builds are not left waiting in a queue. You can get started quickly by using prepackaged build environments, or you can create custom build environments that use your own build tools. With CodeBuild, you are charged by the minute for the compute resources you use.

1. Go to [CodeBuild Console](https://console.aws.amazon.com/codebuild) and click Create build project. Enter your build project information.

     **_Project configuration_**

     * **Project Name:** BlueGreenWebAppBuild  
     * **Description:** NodeJS WebApp build

     **_Source_**

     * **Source provider:** AWS CodeCommit  
     * **Repository:** BlueGreenWebApp   _Note:_ this is your source reporsitory that you have created earlier.
     * **Branch:** Master

     **_Environment_**

     In this step, we configure the build environment.

     * **Environment image:** Managed image  
     * **Operating system:** Ubuntu
     * **Runtime(s):** Standard
     * **Image:** aws/codebuild/standard:4.0
     * **Image version:** Always use the latest image for this runtime version
     * **Environment type:** Linux
     * **Service role:** New service role  
     * **Role name:** codebuild-BlueGreenWebAppBuild-service-role  (Automatically filled)  

     **_Buildspec_**

     * **Build specifications:** Use a buildspec file  
     * **Buildspec name:** empty   _Note:_ We will be using buildspec.yml which is in the project. Because we are using default name, we can leave this field empty.

     **_Artifacts_**

     * **Type:** Amazon S3  Note: We will store build output in S3 bucket.  
     * **Bucket Name:** build-artifact-bluegreenbucket-us-east-1-xxxxxxxxxxx   Note: This bucket was created with CloudFormation template and can be copied fro the Outputs section of the stack.  
     * **Name:** BlueGreenWebAppBuild.zip  
     * **Artifacts packaging:** Zip  

     Click **Create build project**

2. In your Build Project, click **Start build**. Leave everything with default value, click **Start build**
3. Observe the build process and logs. When completed, click **Build details** tab and navigate to the **Artifacts** section.

     **_💡 Tip_** While the process is running, go back to Cloud9 and review `BlueGreenWebApp/buildspec.yml` to get an understanding onto what commands are being executed as part of the Build action. You can find more information about its purpose and reference in this [link](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html).

4. Congratulations! You succesfully made your first build.

## Configure CodeDeploy

> AWS CodeDeploy is a fully managed deployment service that automates software deployments to a variety of compute services such as Amazon EC2, AWS Lambda, and your on-premises servers. AWS CodeDeploy makes it easier for you to rapidly release new features, helps you avoid downtime during application deployment, and handles the complexity of updating your applications. You can use AWS CodeDeploy to automate software deployments, eliminating the need for error-prone manual operations. The service scales to match your deployment needs, from a single Lambda function to thousands of EC2 instances.

1. Go to [CodeDeploy Console](https://console.aws.amazon.com/codedeploy), click **Create application**. Enter Application configuration and click **Create application**

     * **Application name:** BlueGreenWebApp  
     * **Compute platform:** EC2/On-premises  

2. In your CodeDeploy Application, BlueGreenWebApp, on Deployment groups tab, click **Create Deployment group**. Configure the deployment as follow:

     **_Deployment group name_**

     * **Enter a deployment group name:** BlueGreenWebApp_DeploymentGroup  
     * **Service role:** BlueGreenEnvironment-DeployTrustRole-xxxxxxxx    Note: This role was created as a part of CloudFormation.

     **_Deployment type_**

     * **Choose how to deploy your application:** Blue/green

     **_Environment configuration_**

     * **Specify the Amazon EC2 Auto Scaling groups or Amazon EC2 instances where the current application revision is deployed:** Automatically copy Amazon EC2 Auto Scaling group  
     * **Choose the Amazon EC2 Auto Scaling group where the current application revision is deployed:** BlueGreenASGroup  _Note:_ This AS was created as a part of CloudFormation.

     **_Deployment settings_**

     * **Choose whether traffic reroutes to the replacement environment immediately or waits for you to start the rerouting process manually:** Reroute traffic immediately  
     * **Choose whether instances in the original environment are terminated after the deployment is succeeds, and how long to wait before termination:** Terminate the original      instances in the deployment group: 5 Minutes  
     * **Deployment configuration:** CodeDeployDefault.AllAtOnce  

     **_Load balancer_**

     * **Application Load Balancer or Network Load Balancer**
     * **Choose a load balancer:** BlueGreenTG    _Note:_ This role was created as a part of CloudFormation.  

     Click **Create deployment group**

3. Under the deployment group, click **Create deployment**.  Configure the deployment as followed:

     **_Deployment settings_**

     * **Deployment group:** BlueGreenWebApp_DeploymentGroup  
     * **Revision type:** My application is stored in Amazon S3  
     * **Revision location:** s3://build-artifact-bluegreenbucket-REGION-1-XXXXXXXXXXXX/BlueGreenWebAppBuild.zip   Note: This is the location of the build artifact from your CodeBuild project.
     * **Revision file type:** .zip  

     Leave everything as the default value.

     **_💡 Tip_** Your application artifacts contain a file called `appspec.yml` which is used by AWS CodeDeploy to determine what to install onto your instances or what lifecycle events hooks to run upon deployment events, such as how to start and stop your application. Take a look at this file to understand what actions are taking place to successfully deploy your application.

     You can find more information following this [link](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html#appspec-reference-server).

Click **Create deployment**

1. Under the deployment, observe Deployment status. Wait until the deployment has completed and browse to your ALB endpoint to see the application deployed.

![ALB](./images/bg-7.png)

## Create CI/CD with CodePipeline

> AWS CodePipeline is a fully managed continuous delivery service that helps you automate your release pipelines for fast and reliable application and infrastructure updates. CodePipeline automates the build, test, and deploy phases of your release process every time there is a code change, based on the release model you define. This enables you to rapidly and reliably deliver features and updates. You can easily integrate AWS CodePipeline with third-party services such as GitHub or with your own custom plugin. With AWS CodePipeline, you only pay for what you use. There are no upfront fees or long-term commitments.

You are going to configure a CodePipeline to use CodeBuild and CodeDeploy previously created.

1. Go to [CodePipeline Console](https://console.aws.amazon.com/codepipeline) and click **Create Pipeline**.  Configure your Pipeline as followed:

     * **Pipeline name:** BlueGreenWebApp_Pipeline
     * **Service role:** New service role  
     * **Role name:** AWSCodePipelineServiceRole-us-east-1-BlueGreenWebApp_Pipeline (Automatically filled)  
     Enable **Allow AWS CodePipeline to create a service role so it can be used with this new pipeline**  
     * **Artifact Store:** Default location  

     **_Source_**

     * **Source provider:** AWS CodeCommit

     **_AWS CodeCommit_**

     * **Choose a repository:** BlueGreenWebApp  
     * **Branch name:** master  
     * **Change detection options:** Amazon CloudWatch Events(recommended)  

     **_Build_**

     * **Build provider:** AWS CodeBuild

     **_AWS CodeBuild_**

     * **Project name:** BlueGreenWebAppBuild  

     **_Deploy_**

     * **Deploy provider:** AWS CodeDeploy  

     **_AWS CodeDeploy_**

     * **Application name:** BlueGreenWebApp  
     * **Deployment group:** BlueGreenWebApp_DeploymentGroup  

     Click **Next** and **Create pipeline**.

2. Observe your existing commit going through CodePipeline.  

**_💡 Tip_** Now that we have a pipeline defining the different stages and actions to build and deploy our application, we don't need our Build action to download the code from AWS CodeCommit as this is already done by the Source stage. This change cannot be performend through the AWS web console, so execute the following command from Cloud9:

```console
Admin:~/environment/BlueGreenWebApp (master) $ aws codebuild update-project --name BlueGreenWebAppBuild --source type=CODEPIPELINE --artifacts type=CODEPIPELINE
```

## Deploy your new code

1. Go back to your Cloud9 IDE.
2. Navigate to BlueGreenWebApp and public folder. Open index.html.
3. Make a change to the file and save.

     ```html
     <div class="message">
          <a class="twitter-link" href="http://twitter.com/home/?status=I%20created%20a%20project%20with%20AWS%20CodeStar!%20%23AWS%20%23AWSCodeStar%20https%3A%2F%2Faws.amazon.com%2Fcodestar"><img src="img/tweet.svg" /></a>
          <div class="text">
               <h1>Congratulations!</h1>
               <h2>You just created a Node.js web application V2 from the CICD workshop!</h2>
          </div>
     </div>
     ```

4. Commit the change and push to the remote repository.

     ```console
     Admin:~/environment/BlueGreenWebApp (master) $ git add .
     Admin:~/environment/BlueGreenWebApp (master) $ git commit -m "Changes from the CICD workshop"
     Admin:~/environment/BlueGreenWebApp (master) $ git push
     ```

5. Go back to CodePipeline Console and observe the progress.
6. Once completed, browse to your ALB endpoint.

## (Optional) Test AutoScaling

We're going to stress test the ALB endpoint to see the AutoScaling Group adding more instances to handle the new load.

1. In the terminal prompt of the Cloud9 environment, run the following command

     ```console
     Admin:~/environment $ ab -c 100 -t 3600 -n 10000000 http://<ALB ENDPOINT>/index.html
     ```

2. Navigate to the [EC2 AutoScaling Console](https://console.aws.amazon.com/ec2autoscaling/home)
3. Select the AutoScaling Group created by CodeDeploy
4. Check in **_Activity_** or **_Monitoring_** if new instances are being created

**Congratulations! You have completed the lab.**
