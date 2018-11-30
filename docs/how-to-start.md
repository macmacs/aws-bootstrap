# HOW TO START?

This guide describes steps necessary to deploy basic infrastructure for Kubernetes-based environment
on a pair of AWS accounts (operations and application).


## PREREQUISITIES:

1. AWS accounts created (e.g. by requesting through internal IT procedures):

	* operations/default (with VPC and at least 1 private subnet per Availability Zone)
	* application/advanced (with VPC including 3 public and 3 private subnets and NAT Gateway(s))
    * please ensure at least /26 subnets everywhere
	* VPC peering between the 2 accounts (operations/advanced, including DNS resolution)
	* access to both accounts with AWS console or aws-cli with highly-administrative roles

3. git and ssh clients on your laptop/workstation

4. Access to Kentrikos project's public git repositories:

	```
	git clone https://github.com/kentrikos/aws-bootstrap.git
	```
    (altrernatively use web browser to download repo from GitHub)

5. Configuration repository (typically private as it contains environment-specific values such as AWS account numbers, VPC IDs, etc.):
    * e.g. located in your private GitHub repo or Bitbucket, please contact your admin
    * __private__ SSH key allowing read-only access



## NOTES:

* some Jenkins jobs may require manual confirmation (e.g. for 'terraform apply' stage), please hover your mouse over the paused stage to see confirmation button


## STEPS:

1. Create IAM Policies for core-infra jenkins and cross-account role:

* use jsons from `aws-bootstrap/iam/kops/` on both your operations and application account (using either you internal procedures or directly with AWS console/IAM). For policy names use filenames without `.json` (e.g. `KOPS_MANAGEMENT_NODE_ec2`).

2. Create SSH key pair

* using AWS console (EC2 services/Key Pairs) create (or import existing) new SSH key pair

	> This key will be used in the steps below to ssh to your first bastion host. Please note its name and make sure you have private key stored securely on your laptop. This should be a .pem file

3. Create your own configuration (product domain) repo or fork from existing example (e.g. tf-aws-product-domain-demo-env-test into tf-aws-product-domain-demo2-env-test).

	> See [howto-fork-repo.md](https://github.com/kentrikos/aws-bootstrap/blob/master/docs/howto-fork-repo.md) for more information on Naming Conventions)

4. Update Terraform variables and backend configuration:

* in product domain repo - ```cd ops/REGION/jenkins-core-infra```
* update backend.tf - with s3 bucket, key and dynamodb state accordingly to naming-conventions document below
* update variables.tf
	> See [naming-conventions.md](https://github.com/kentrikos/aws-bootstrap/blob/master/docs/naming-conventions.md) for more information on Naming Conventions)
* review other parts of the repo and update variables.tf files where necessary (remember that product domain name must be unique per AWS account)

5. Create TF state resources and Bastion Host (BH) on operations account:

* open CloudFormation service in AWS web console
* use `aws-bootstrap/cfn/operations-account.yaml` template
* name your stack in meaningful way (e.g. `demo-test-operations`, note that it will become prefix for EC2 instance name as well)
* choose your VPC
* choose your subnet (e.g. `ops-test-private-private_a`)
* select an SSH key pair you've created in step 2
* select and IP range from which connections will be allowed to the BH (do not allow all hosts e.g. 0.0.0.0/0)
* paste a private SSH key to Github (see PREREQUISITIES ssh key section)
* verify that URL to the repository points to the desired product domain and environment (edit the URL if necessary/make sure it contains the forked repo)
* note private IP/DNS of BH for next step

6. Deploy core-infra Jenkins from BH

* login to BH via SSH
* ensure your repository is cloned there already
* execute terraform:
```
cd terraform/YOUR_REPO/ops/AWS_REGION/jenkins-core-infra
terraform init
terraform plan -out=tfplan -input=false
terraform apply -input=false tfplan
```
* note TF outputs with URL, login and password for Jenkins dashboard (to be open in web browser)

7. Create TF state resources and cross-account role on application account

* open CloudFormation service in AWS web console
* use `aws-bootstrap/cfn/application-account.yaml` template
* name your stack in meaningful way (e.g. `demo-test-application`)
* provide operations account as a parameter (required to create trust relationship)
* ensure ARN of cross-account role created matches variable(s) in your configuration repository

8. Create IAM Policies for Kubernetes cluster instances (masters and nodes) on both accounts:

* use core-infra Jenkins job "Create_IAM_policies" (if you are not allowed to create policies directly it will just create json files for you that you can use in your internal procedures)

9. Deploy Kubernetes cluster on operations account using core-infra Jenkins:

* open web dashboard
* FIXME: due to https://github.com/jenkinsci/ssh-credentials-plugin/pull/33 you need to update credentials manually (go to Manage Jekins/Configure Credentials/Credentials/git/Update and enter new line at the end of private key - just hit Enter and press Save)
* run "Kubernetes_Install" job

10. Deploy jx on K8s cluster in operations account (using core-infra Jenkins):

* run "Install_JX" job
* in "Manual create domain" go to AWS console/Route53 and create alias to jx ingress LB (FIXME: improve instructions)

11. Manually configure jx after it is installed (using URL and credentials for web dashboard from previous step, FIXME: this should be automated):

* configure HTTP proxy settings for plugins (Manage Jenkins/Manage Plugins/Advanced - please note 'Server' is without 'http://' and port and 'No Proxy' is one entry per line)
* go to 'Available' plugins tab, update the list ('Check now') and install (without restart) the following plugins: 'SSH Agent', 'AWS Parameter Store Build Wrapper'
* refresh the 'Installing Plugins/Upgrades' to verify plugins got installed
* manually create new pipeline job (from Jenkins main dashboard: 'New item/Pipeline', name it 'Kubernetes_Install_On_Application')
* copy&paste Pipeline script from `jenkins-bootstrap-pipelines` repository, `/application/kubernetes/install/Jenkinsfile`
* update top `parameters` section with defaults apropriate for your environment
* save the job
* add private SSH key for accessing your configuration repository (from Jenkins main dashboard: 'Credentials/System/Global/Add Credentials/SSH Username with private key', Username: 'git', ID: 'bitbucket-key', enter key directly with 1 empty line at the end)

12. Deploy Kubernetes cluster on "application" account:

* run jx job from previous step (double check parameters)
