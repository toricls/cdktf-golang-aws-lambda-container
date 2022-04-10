# cdktf-golang-aws-lambda-container

An example cdktf project that uses Golang, to deploy containerized application on AWS Lambda 📦

For reference:
- [CDK for Terraform official doc](https://www.terraform.io/cdktf) by HashiCorp
- [Introduction to CDK for Terraform](https://learn.hashicorp.com/tutorials/terraform/cdktf) by HashiCorp
- [Introducing the Cloud Development Kit for Terraform (Preview)](https://aws.amazon.com/blogs/developer/introducing-the-cloud-development-kit-for-terraform-preview/) by AWS

## Prerequisites

- CDK for Terraform (`cdktf`) v0.10.1
- Docker
- Go

## Prepping

We assume you already have Docker and Go installations in your laptop, so let's install the `cdktf` command.

```shell
# https://learn.hashicorp.com/tutorials/terraform/cdktf-install
# See the official docs above to install cdktf command
# Here is an example for macOS with Homebrew
❯ brew install cdktf

❯ cdktf --version
0.10.1
```

Let's download dependencies and generate code.

```shell
# This may take several minutes to complete.
❯ cdktf get
⠧ downloading and generating modules and providers...

Generated go constructs in the output directory: generated

The generated code depends on jsii-runtime-go. If you haven't yet installed it, you can run go mod tidy to automatically install it.

# Install jsii to let Go code to interact with CDK
❯ go mod tidy
```

### Pull and push "Lambda-enabled" container image to Amazon ECR

Because AWS Lambda only support Amazon ECR private repositories today, you need to copy a sample container image from Amazon ECR Public (or bring your own Lambda-enabled container image) to your own Amazon ECR private repository.

```shell
# Pull public container image that supports AWS Lambda
❯ docker pull public.ecr.aws/toricls/go-hello-world:latest

# Make sure you're using expected AWS profile or AWS credentials

# Create your Amazon ECR private repository if you don't have yet
# We use Oregon region here but of course you can choose your preferred AWS region instead
❯ AWS_REGION=us-west-2
❯ aws ecr create-repository --repository-name go-hello-world
❯ ACCOUNT_ID=$(aws sts get-caller-identity --query 'Account' --output text)
❯ docker tag public.ecr.aws/toricls/go-hello-world:latest "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/go-hello-world:latest"
❯ docker push "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/go-hello-world:latest"
```

## Deploy

The cdktf code in this repository requires two environment variable to deploy - `TF_VAR_AWS_REGION` and `TF_VAR_CONTAINER_IMG_ARCH`.

- `TF_VAR_AWS_REGION` - Choose AWS region where you used in the previous steps for the container image
- `TF_VAR_CONTAINER_IMG_ARCH` - This value depends on your current environment in general, so just choose the architecture of your container image you pushed in the previous steps. If you're not familiar with, then choose "arm64" if you're using Apple Sillicon (e.g. M1) Mac, and choose "x86_64" instead if you're on Intel Mac. This should work in most cases.

Note that `cdktf` command here requires AWS profile or AWS credentials to create resources in your AWS account.

```shell
❯ TF_VAR_AWS_REGION="us-west-2" TF_VAR_CONTAINER_IMG_ARCH="arm64" cdktf deploy --auto-approve
...
```

## Cleanup

```shell
❯ cdktf destroy --auto-approve
...
```

## See the generated Terraform template

`cdktf` generates and stores the Terraform template under auto-generated `cdktf.out` directory.

```shell
❯ cat ./cdktf.out/stacks/cdktf-go-example/cdk.tf.json
```

If you want to see the template without deploying actual resources, then use `cdktf synth` command as:

```shell
❯ TF_VAR_AWS_REGION="us-west-2" TF_VAR_CONTAINER_IMG_ARCH="arm64" cdktf synth

Generated Terraform code for the stacks: cdktf-go-example

❯ cat ./cdktf.out/stacks/cdktf-go-example/cdk.tf.json
```

## Contribution

1. Fork ([https://github.com/toricls/cdktf-golang-aws-lambda-container/fork](https://github.com/toricls/cdktf-golang-aws-lambda-container/fork))
1. Create a feature branch
1. Commit your changes
1. Rebase your local changes against the master branch
1. Create a new Pull Request

## Licence

[MIT](LICENSE)

## Author

[Tori](https://github.com/toricls)
