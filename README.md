# Flask Hello World with Docker and Amazon ECR

This repository contains a small Flask application that listens on port `5000`.
The commands below show how to build a container image, publish it to Amazon
Elastic Container Registry (ECR), and run it locally.

## Prerequisites

Install and configure the following tools:

- [Git](https://git-scm.com/downloads)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- An AWS account with permission to use IAM, ECR, and STS

Docker Desktop must be running before you build or run the image.

## 1. Fork, clone, and open the repository

Fork [shyamnair488/flask-hello-world-ecr](https://github.com/shyamnair488/flask-hello-world-ecr)
to your GitHub account, then run the following commands in PowerShell:

```powershell
git clone https://github.com/shyamnair488/flask-hello-world-ecr.git
Set-Location .\flask-hello-world-ecr
code .
```

If you cloned your fork, use your own GitHub username and repository URL.

## 2. Check the installed tools

```powershell
aws --version
docker --version
git --version
```

## 3. Create an AWS access key securely

For learning, use an IAM user or role that has only the permissions required
for this exercise. Avoid using the AWS account root user and avoid granting
permanent administrator access when a narrower policy is sufficient.

1. Open the [AWS IAM Console](https://console.aws.amazon.com/iam/).
2. Go to **Users**, select your IAM user, and open **Security credentials**.
3. Under **Access keys**, select **Create access key**.
4. Select the **Command Line Interface (CLI)** use case.
5. Copy or download the access key ID and secret access key immediately.

The secret access key is shown only when it is created. Never commit it to this
repository, paste it into source code, or share it in an issue or chat.

## 4. Configure and test the AWS CLI

Run:

```powershell
aws configure
```

Use values for your own account. The following are sample values only:

```text
AWS Access Key ID [None]: AKIAEXAMPLE000000000
AWS Secret Access Key [None]: example-secret-access-key
Default region name [None]: us-east-1
Default output format [None]: json
```

Confirm that the credentials work:

```powershell
aws sts get-caller-identity
```

Example response shape:

```json
{
  "UserId": "AIDAEXAMPLE",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/example-user"
}
```

Use your own region consistently for all subsequent commands. This guide uses
`us-east-1` as an example.

## 5. Set the ECR image values

Set these PowerShell variables once. Replace the account ID and repository name
with values from your AWS account:

```powershell
$env:AWS_REGION = "us-east-1"
$env:AWS_ACCOUNT_ID = "123456789012"
$env:ECR_REPOSITORY = "flask-hello-world-ecr"
$env:IMAGE_TAG = "latest"
$env:ECR_REGISTRY = "$env:AWS_ACCOUNT_ID.dkr.ecr.$env:AWS_REGION.amazonaws.com"
$env:IMAGE_URI = "$env:ECR_REGISTRY/$env:ECR_REPOSITORY`:$env:IMAGE_TAG"
```

For example, the image URI will look like:

```text
123456789012.dkr.ecr.us-east-1.amazonaws.com/flask-hello-world-ecr:latest
```

## 6. Create the ECR repository

Create the repository once:

```powershell
aws ecr create-repository `
  --repository-name $env:ECR_REPOSITORY `
  --region $env:AWS_REGION
```

If the repository already exists, AWS returns a repository-exists error; in
that case, continue with the login step.

## 7. Log in to ECR

Authenticate Docker to the registry:

```powershell
aws ecr get-login-password --region $env:AWS_REGION |
  docker login --username AWS --password-stdin $env:ECR_REGISTRY
```

## 8. Build and push the image

Build a multi-platform image and push it directly to ECR. Docker Buildx is
included with current Docker Desktop installations.

```powershell
docker buildx build `
  --platform linux/amd64,linux/arm64 `
  --tag $env:IMAGE_URI `
  --push .
```

Verify that the image is available in ECR:

```powershell
aws ecr describe-images `
  --repository-name $env:ECR_REPOSITORY `
  --region $env:AWS_REGION
```

To see local images:

```powershell
docker images
```

## 9. Run the image locally

Pull the image for the current machine and map local port `5000` to the
container's port `5000`:

```powershell
docker pull $env:IMAGE_URI
docker run --detach --name flask-hello-world-ecr --publish 5000:5000 $env:IMAGE_URI
docker container ls
```

Open [http://localhost:5000](http://localhost:5000) or test it from PowerShell:

```powershell
Invoke-WebRequest http://localhost:5000
```

Expected response:

```text
Hello from Flask! Using Docker and AWS ECR for deployment.
```

View logs and stop the container when finished:

```powershell
docker logs flask-hello-world-ecr
docker stop flask-hello-world-ecr
docker rm flask-hello-world-ecr
```

## Useful ECR commands

List repositories:

```powershell
aws ecr describe-repositories --region $env:AWS_REGION
```

Delete a repository and all images in it only when you are sure it is no longer
needed:

```powershell
aws ecr delete-repository `
  --repository-name $env:ECR_REPOSITORY `
  --region $env:AWS_REGION `
  --force
```

## Project files

- `app.py` - Flask application
- `requirements.txt` - pinned Python runtime dependency
- `Dockerfile` - container image definition
- `deployment.yaml` - optional Kubernetes Deployment and Service example
