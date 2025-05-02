### Overview

In this project, I set up a CI/CD pipeline using **GitHub Actions** to automatically build and push a Docker image of a Node.js application to **Amazon ECR**, and then deploy it to an **EC2 instance** using **AWS SSM (Systems Manager)**.

Before setting up automation, I tested the app manually to ensure everything worked as expected. This included running the app locally, creating the necessary AWS infrastructure (EC2 and ECR).

---

### Pre-Automation Manual Steps

Before writing the pipeline:

1. **Tested the Node.js App Locally:**
   I ensured the app worked on my machine and ran without issues in a Docker container.

2. **Created AWS Infrastructure:**

   * **Amazon EC2 Instance:** Set up a Ubuntu-based EC2 server, installed Docker and Docker Compose, and exposed the necessary ports.
   * **Amazon ECR Repository:** Created a private ECR repo named `node-app` to host my Docker image.
   * **IAM Credentials:** Created AWS IAM credentials to authenticate and give GitHub access to push the image created to ECR and run a deploy script on EC2. This access was restricted to EC2 and ECR.

3. **Wrote a Simple `docker-compose.yml` File:**
   On the EC2 instance, I wrote a Docker Compose file to define how the app should run in a container.

4. **Created a `deploy.sh` Script:**
   This script on the EC2 instance pulls the latest image from ECR and restarts the container. It's triggered remotely in the automated workflow.

---

### GitHub Actions CI/CD Workflow

```yaml
name: Deploy Node.js App to EC2 via ECR

on:
  push:
    branches: [main]
```

This workflow is triggered every time a push is made to the `main` branch.

---

#### Job 1: `Build Image`

This job builds and pushes the Docker image to ECR.

```yaml
jobs:
  build:
    name: Build Image
    runs-on: ubuntu-latest
    steps:
```

##### Step-by-step:

1. **Check Out the Code:**

   ```yaml
   - name: Check out code
     uses: actions/checkout@v2
   ```

   This step pulls the latest code from the GitHub repository.

2. **Configure AWS Credentials:**

   ```yaml
   - name: Configure AWS credentials
     uses: aws-actions/configure-aws-credentials@v1
     with:
       aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
       aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
       aws-region: eu-central-1
   ```

   This step authenticates the GitHub Action with AWS using stored secrets.

3. **Log in to ECR:**

   ```yaml
   - name: Login to Amazon ECR
     id: login-ecr
     uses: aws-actions/amazon-ecr-login@v1
   ```

   Logs into Amazon ECR so Docker can push images there.

4. **Build Docker Image:**

   ```yaml
   - name: Build Docker Image
     env:
       ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
       ECR_REPOSITORY: node-app
       IMAGE_TAG: latest
     run: |
       docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
   ```

   This step builds the Docker image from the project code but does not push it yet.

5. **Scan Docker Image for Vulnerabilities:**

   ```yaml
   - name: Scan Docker image for vulnerabilities
     uses: crazy-max/ghaction-container-scan@v2
     with:
       image: ${{ steps.login-ecr.outputs.registry }}/node-app:latest
   ```

   This step scans the Docker image for known vulnerabilities before pushing it to ECR. It ensures that the image is secure and free from any known vulnerabilities.

6. **Push Docker Image to ECR:**

   ```yaml
   - name: Push Docker image to Amazon ECR
     env:
       ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
       ECR_REPOSITORY: node-app
       IMAGE_TAG: latest
     run: |
       docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
   ```

   If the scan passes without issues, this step pushes the Docker image to ECR.

---

#### Job 2: `Deploy to EC2`

After successfully building and pushing the image, this job deploys the image to an EC2 instance.

```yaml
  Deploy:
    name: DEPLOY TO EC2
    runs-on: ubuntu-latest
    needs: build
    steps:
```

##### Step-by-step:

1. **Trigger EC2 via AWS SSM:**

   ```yaml
   - name: AWS SSM STEP
     uses: peterkimzz/aws-ssm-send-command@v1.1.1
     id: ssm
     with:
       aws-region: ${{ secrets.AWS_REGION }}
       aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
       aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
       instance-ids: ${{ secrets.INSTANCE_ID }}
       working-directory: /home/ubuntu
       command: |
         cd /home/ubuntu/node-app
         bash deploy.sh
   ```

   This step uses AWS Systems Manager to SSH into the EC2 instance **without needing a key pair** and runs a command that navigates to the app directory and executes `deploy.sh`. That script pulls the latest Docker image from ECR and restarts the service.

2. **Send Slack Notification on Deployment Success:**

   ```yaml
   - name: Send Slack notification on success
     uses: slackapi/slack-github-action@v1.24.0
     with:
       payload: |
         {
           "text": "✅ *Deployment Successful!* Your Node.js app is now live on EC2.",
           "channel": "#slack-s14"
         }
     env:
       SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
   ```

   This step sends a notification to a Slack channel, informing the team that the deployment was successful.

---

### Summary

* ✅ Manually tested the Node.js app and Docker setup.
* ✅ Created an EC2 instance and an ECR repository.
* ✅ Verified image build and deployment with Docker Compose manually.
* ✅ Automated everything with GitHub Actions using two jobs:

  * One to build and push the Docker image to ECR.
  * Another to trigger deployment on EC2 via AWS SSM.
* ✅ **Security Scan:** Integrated a vulnerability scan before pushing the Docker image to ECR.
* ✅ **Slack Notification:** A notification is sent to Slack upon successful deployment.

---

### Security Notes

* All sensitive data (AWS credentials, EC2 instance ID, Slack webhook URL) are securely stored as **GitHub Actions secrets**.
* **AWS SSM** is used to securely run commands on the EC2 instance without exposing SSH keys.
* **Docker Image Scan** is integrated into the pipeline to check for vulnerabilities in the Docker image before pushing it to ECR, ensuring that only secure images are deployed.
* A **Slack notification** is sent to inform the team of a successful deployment, keeping everyone updated on the status.

---

### Future Security Enhancements

If I had more time, I would have implemented additional security checks within the pipeline to ensure that the code and images are free from vulnerabilities. Some improvements could include:

* **Code Scanning and Vulnerability Detection:** I would integrate security tools like **Snyk** or **OWASP Dependency-Check** to scan the application code and dependencies for vulnerabilities before building the Docker image.
* **Static Analysis:** Integrating a static code analysis tool (like **SonarQube**) could help catch security issues and maintain high code quality before the deployment process.

---
