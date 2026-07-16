# 🌍 Universal Helm Chart

![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)
![AWS ECR](https://img.shields.io/badge/Amazon_ECR-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)

This repository contains the **Universal DRY (Don't Repeat Yourself) Helm Blueprint**. It is the centralized infrastructure engine that powers the microservices within the [AWS-Infra-Gitops-Boilerplate](https://github.com/Parth2496Singh/AWS-Infra-Gitops-Boilerplate).

By decoupling the Helm chart from the main infrastructure repository, we achieve a true enterprise-grade architecture where Kubernetes templates are treated as immutable, version-controlled artifacts.

## 🏗️ How it Works

Instead of application developers writing 500+ lines of raw Kubernetes YAML for every single microservice, this chart acts as a dynamic "template machine". 

Depending on the configuration passed to it, this chart can instantly generate:
*   `Deployments`, `StatefulSets`, or `DaemonSets`
*   `Services` & `Ingress` (with AWS ALB routing)
*   `ServiceMonitors` (for automated Prometheus scraping)
*   `ConfigMaps` & `Secrets` (dynamically injected from `envConfig`)
*   Arbitrary `extraManifests` (for edge-cases like CronJobs or NetworkPolicies)

## 🚀 Automated Release Pipeline (OCI)

This repository utilizes an automated GitHub Actions CI/CD pipeline (`publish.yaml`) integrated natively with AWS via OIDC (keyless authentication).

When a Pull Request is merged into `main` and the `Chart.yaml` version is bumped:
1. GitHub Actions securely logs into **Amazon Elastic Container Registry (ECR)**.
2. It runs `helm package .` to compress the templates into a `.tgz` artifact.
3. It pushes the artifact to AWS ECR as an **OCI (Open Container Initiative)** package.

## 🔗 Integration with GitOps

The GitOps cluster does **not** read from this Git repository directly. 

Instead, Argo CD is configured to pull the immutable OCI artifact directly from AWS ECR. Applications in the main infrastructure repository use the **Umbrella Chart (Dependency) Pattern** to import this blueprint:

```yaml
# Example Chart.yaml in the GitOps Boilerplate
apiVersion: v2
name: payment-service
version: 1.0.0
dependencies:
  - name: common-microservice
    version: 1.0.1  # Matches a version released from this repository!
    repository: oci://<YOUR_AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/universal-helm-chart
```

## 🛡️ IAM Security

The pipeline pushing to ECR uses an AWS IAM Role strictly bound by the **Principle of Least Privilege**. The CloudFormation template used to provision this role (`iam/aws-oidc-helm-role.yaml`) only grants `AmazonEC2ContainerRegistryPowerUser` permissions, completely isolating this repository from the rest of the AWS environment.
