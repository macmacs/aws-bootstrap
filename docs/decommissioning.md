# HOW TO REMOVE ENVIRONMENT?

This guide describes steps necessary to destoy basic infrastructure for Kubernetes-based environment
on a pair of AWS accounts (operations and application).


## STEPS:

1. [JenkinsX] Run Kubernetes_Destroy job to remove application cluster

2. [Jenkins Core-Infra] Run Kubernetes_Destroy job remove operation cluster

3. [Dev-host] Remove Jenkins Core-infra with terraform command

3.1 Login with ec2-user to dev-host:

3.2 [ec2-user@ip-10-6-39-55 jenkins-core-infra]$ terraform plan -input=false -destroy -var-file=../terraform.tfvars

3.3 [ec2-user@ip-10-6-39-55 jenkins-core-infra]$ terraform destroy -input=false -var-file=../terraform.tfvars

3.4 [ec2-user@ip-10-6-39-55 ~]$ cd ~/terraform/bmw-tf-aws-product-domain-maps-env-test/operations/eu-central-1/jenkins-core-infra

4. [Operation_aws_account console] Destroy CF stack


## TROUBLSHOOTING:

1. Problem with state lock for Terraform resources
Error: Error locking state: Error acquiring the state lock: ConditionalCheckFailedException: The conditional request failed
	status code: 400, request id: RBN5ECNCCTNB0JE4FR96LJNTS7VV4KQNSO5AEMVJF66Q9ASUAAJG
Lock Info:
  ID:        cb43d95b-8e14-0d9d-9368-d937ac30de3b
  Path:      tf-state-bootstrap-111860764813-ops-eu-central-1/tf/tf-aws-product-domain-maps-env-test/jenkins-core-infra/terraform.tfstate
  Operation: OperationTypeApply
  Who:       ec2-user@ip-10-6-39-55.eu-central-1.compute.internal
  Version:   0.11.10
  Created:   2018-12-06 10:48:25.822692885 +0000 UTC
  Info:


Terraform acquires a state lock to protect the state from being written
by multiple users at the same time. Please resolve the issue above and try
again. For most commands, you can disable locking with the "-lock=false"
flag, but this is not recommended.

Solution: terraform destroy -input=false -var-file=../terraform.tfvars -lock=false
