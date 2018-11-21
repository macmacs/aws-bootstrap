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

3. Create own product domain repo or fork from existing example  - tf-aws-product-domain-maps-env-test - (example: cust-tf-aws-rtti-maps-env-test)__

	> See [howto-fork-repo.md](https://github.com/kentrikos/aws-bootstrap/blob/master/docs/howto-fork-repo.md) for more information on Naming Conventions)

4. Update User Variables

* In Product domain repo - ```cd ops/eu-central-1/jenkins-core-infra```
* Update backend.tf - with s3 bucket, key and dynamodb state (unique string)
* Update variables.tf with Jenkins information

	> See [naming-conventions.md](https://github.com/kentrikos/aws-bootstrap/blob/master/docs/naming-conventions.md) for more information on Naming Conventions)

5. Create Bastion Host

* Using AWS console and CloudFormation deploy bastion host (BH) in __operations__ account (a simple EC2 instance that you will use later on to deploy other infra elements with Terraform)
* Use `aws-bootstrap/BH-TF-ops.yaml` template
* Give it any stack name (e.g. `product-domain-maps-env-test-tf-bh`, note that it will become EC2 instance name as well)
* Choose your VPC
* Choose your subnet (e.g. `ops-test-private-private_a`)
* Select an SSH key pair you've created in step 2
* Select and IP range from which connections will be allowed to the BH (do not allow all hosts e.g. 0.0.0.0/0)
* Paste a private SSH key to Github (see PREREQUISITIES ssh key section)
* Verify that URL to the repository points to the desired product domain and environment (edit the URL if necessary/make sure it contains the forked repo)

6. Clone Repository

* Login to dev host via SSH
* Clone repository with your branch (example: tf-aws-rtti-maps-env-test)

7. Execute Terraform

* Run Plan/Apply - Manual deployment via terraform ( This creates: S3 bucket and DynamoDB for terraform state, Jenkins incl. configuration, plugins and deployment tools)

8. Check Output - Receive IP Address of Jenkins node and paste it into browser (i.e. http://10.1.2.3:8080) to get Jenkins dashboard

* The last line is the IP address that works on port 8080 to access Jenkins Core Infra
