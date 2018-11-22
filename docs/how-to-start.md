# HOW TO START?

This guide describes steps necessary to deploy basic infrastructure for Kubernetes-based environment
in project operations AWS accounts.


## PREREQUISITIES:

1. AWS accounts requested and created using IAM portal:

	* operation/default (with VPC and at least 1 private subnet per Availability Zone)
	* application/advanced (with VPC including 3 public and 3 private subnets, Nat Gateway)
	* VPC peering between operation and advanced AWS accounts mentioned above
	* access to both accounts with AWS console or aws-cli with highly-administrative roles (e.g. OwnFull)

2. Access to Kentrikos project at repository:

	* clone `aws-bootstrap` repository (alternatively just download it from Github using a browser):

	```
	git clone https://github.com/kentrikos/aws-bootstrap.git
	```

3. A __private__ SSH key allowing read-only access to required repos (please contact project admin of Kentrikos)


## STEPS:

1. Create IAM Policies

Using IAM's portal, create policies from `aws-bootstrap/iam/kops/` on both your operations and application account.

* For policy names use filenames without `.json` (e.g. `KOPS_MANAGEMENT_NODE_ec2`)

2. Create SSH Key

* Using AWS console (found under EC2 services/Key Pairs)
* Create (or import) new SSH key pair,

	> This key will be used in the steps below to ssh to your first bastion host. Please note its name and make sure you have private key stored securely on your laptop. This should be a .pem file

3. Create own product domain repo or fork from existing example  - tf-aws-product-domain-maps-env-test

	> See [howto-fork-repo.md](https://github.com/kentrikos/aws-bootstrap/blob/master/docs/howto-fork-repo.md) for more information on Naming Conventions)

4. Update Terraform variables and backend configuration.

* In Product domain repo - ```cd ops/eu-central-1/jenkins-core-infra```
* Update backend.tf - with s3 bucket, key and dynamodb state accordingly to naming-conventions document below
* Update variables.tf
	> See [naming-conventions.md](https://github.com/kentrikos/aws-bootstrap/blob/master/docs/naming-conventions.md) for more information on Naming Conventions)

5. Create TF state resources and Bastion Host (BH) on operations/transit account:

* open CloudFormation service in AWS web console
* use `aws-bootstrap/cfn/BH-TF-ops.yaml` template
* name your stack in meaningful way (e.g. `product-domain-maps-env-test-ops`, note that it will become EC2 instance name as well)
* choose your VPC
* choose your subnet (e.g. `ops-test-private-private_a`)
* select an SSH key pair you've created in step 2
* select and IP range from which connections will be allowed to the BH (do not allow all hosts e.g. 0.0.0.0/0)
* paste a private SSH key to Github (see PREREQUISITIES ssh key section)
* verify that URL to the repository points to the desired product domain and environment (edit the URL if necessary/make sure it contains the forked repo)

6. Deploy core-infra Jenkins from BastionHost
* Login to BH via SSH
* ensure your repository is cloned there already
* execute terraform:
```
cd terraform/YOUR_REPO/transit/transit/AWS_REGION/jenkins-core-infra
terraform init -input=false
terraform plan -out=tfplan -input=false
terraform apply -input=false tfplan
```
* note output with IP Address of Jenkins node and paste it into browser (i.e. http://10.1.2.3:8080) to get Jenkins dashboard

7. Create TF state resources and cross-account role on application account:
* open CloudFormation service in AWS web console
* use `aws-bootstrap/cfn/application-account.yaml` template
* name your stack in meaningful way (e.g. `product-domain-maps-env-test-app`)
* provide operations/transit account as a parameter (required to create trust relationship)
