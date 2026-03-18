# Coder CodeBuild Container Image

Pre-baked container image for the Coder EKS deployment CodeBuild job.

## What's included

| Tool       | Source                                      |
|------------|---------------------------------------------|
| AWS CLI v2 | awscli-exe-linux-x86_64                     |
| kubectl    | kubernetes-release (stable)                 |
| eksctl     | weaveworks/eksctl (latest)                  |
| Helm       | helm v3.16.4 (configurable via build arg)   |
| Terraform  | hashicorp/terraform (latest)                |
| Coder CLI  | coder.com/install.sh (configurable)         |
| Python 3.12| Amazon Linux 2023 package                   |
| git, jq    | Amazon Linux 2023 packages                  |

## Build args

Override at build time to pin versions:

```
docker build \
  --build-arg HELM_VERSION=3.16.4 \
  --build-arg CODER_VERSION=2.30.2 \
  --build-arg KUBECTL_VERSION=v1.32.0 \
  --build-arg TERRAFORM_VERSION=1.10.3 \
  -t coder-codebuild:latest .
```

## Deploying the pipeline

1. Zip the `docker/` directory contents and upload to S3:
   ```bash
   cd infrastructure
   zip -r codebuild-image-source.zip docker/
   aws s3 cp codebuild-image-source.zip s3://YOUR_BUCKET/codebuild-image-source.zip
   ```

2. Deploy the CloudFormation stack (the ECR Public alias is auto-discovered — no need to know it upfront):
   ```bash
   aws cloudformation deploy \
     --template-file codebuild_image_pipeline.yaml \
     --stack-name coder-codebuild-image \
     --parameter-overrides \
       SourceBucket=YOUR_BUCKET \
     --capabilities CAPABILITY_NAMED_IAM
   ```

3. Grab the image URI from the stack outputs:
   ```bash
   aws cloudformation describe-stacks \
     --stack-name coder-codebuild-image \
     --query 'Stacks[0].Outputs'
   ```

4. Trigger the first build:
   ```bash
   aws codebuild start-build --project-name coder-codebuild-image-builder
   ```

## Using the image in coder_deployment.yaml

Replace the `Image` property in the BuildProject Environment with the
`ECRPublicRepositoryUri` output value:

```yaml
Image: public.ecr.aws/YOUR_ALIAS/coder-codebuild:latest
```

The pre_build phase then simplifies to just generating the cluster config — no more downloading binaries.
