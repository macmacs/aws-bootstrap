# Naming convention:

## Top level names:
* if possible names for all other resources (e.g. s3 buckets or K8s cluster) should be computed from PRODUCT_DOMAIN_NAME and ENV_TYPE, this will allow
  for very simple input from user and also keep all resulting names predictable.

## S3 state buckets/keys:
* bucket name: `tf-state-bootstrap-AWS_ACCOUNT_NUMBER-AWS_ACCOUNT_TYPE-AWS_REGION` - e.g. `tf-state-bootstrap-123456789012-ops-eu-central-1` (S3 bucket names are global across all AWS accounts/regions)
* key: `tf/tf-aws-product-domain-PRODUCT_DOMAIN_NAME-env-ENV_TYPE/SUBCOMPONENT/terraform.tfstate` - e.g. `tf/tf-aws-product-domain-maps-env-test/jenkins/terraform.tfstate` (for main set of subcomponents deployed by infrastructure module use "env" for SUBCOMPONENT)
* please keep in mind S3 bucket name length limit (63 characters) and that multiple TF states (for different infra components) are kept under the same bucket (separated by keys)

## DynamoDB state lock table:
* `tf-state-lock-bootstrap` (since table names are not global)
* multiple TF states will be automatically locked using 1 table by using different items (named after S3 keys), so no need for separate tables for single account/region
* DynamoDB table name length limit = 255 characters which should be plenty

## AWS CLI PROFILES (optional):
* `project-AWS_ACCOUNT_NAME-AWS_ACCOUNT_NUMBER` - e.g.  `project-main-test-234567890123`
* awscli profiles are expected to be pre-configured on machine used interact with AWS using awscli
