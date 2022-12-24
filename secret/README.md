# Secret

## A cf template to create an aws secrets storage with an user that can access the entries. 

### Create stack
```
aws cloudformation create-stack --on-failure DELETE --profile awsa --template-body file://secretsVault.yaml  --stack-name secretStore5 --capabilities CAPABILITY_IAM
```
### Deploy stack
```
aws cloudformation deploy --profile awsa --template-file secretsVault.yaml  --stack-name secretStore5  --capabilities CAPABILITY_IAM
```
### Update stack
```
aws cloudformation update-stack --profile awsa --stack-name secretStore5 --template-body file://secretsVault.yaml --capabilities CAPABILITY_IAM
```
### Delete stack
```
aws cloudformation delete-stack --profile awsa --stack-name secretStore5
```

### Describe stack
```
aws cloudformation describe-stack-events --profile awsa --stack-name secretStore5
aws cloudformation describe-stacks --profile awsa --stack-name secretStore5
```

### Query describe stack
```
aws cloudformation describe-stacks --profile awsa --stack-name secretStore5 --query "Stacks[?StackName=='secretStore5'][].Outputs[?OutputKey=='my_output_key'].OutputValue" --output text

aws cloudformation list-exports --profile awsa --no-cli-pager --output text --query "Exports[?Name=='secretStore5SmUsrId'][].Value"

aws cloudformation describe-stacks --profile awsa --stack-name secretStore5 --output text --query "Stacks[?StackName=='secretStore5'][].Outputs[?OutputKey=='secretsVaultUsrAccessKeyId'][].OutputValue"

aws cloudformation describe-stacks --profile awsa --stack-name secretStore5 --output text --query "Stacks[?StackName=='secretStore5'][].Outputs[?OutputKey=='secretsVaultUsrSecretKey'][].OutputValue"
```

### Create an access profile
```
aws configure --profile profile1
```

### Query the secrets manager using aws-cli
```
docker run --rm -it -v "%userprofile%\.aws:/root/.aws" amazon/aws-cli secretsmanager get-secret-value  --profile profile1 --no-cli-pager --secret-id secretsVault-bVMjveIsvBvb --output json --query SecretString

docker run --rm -it -v "$home\.aws:/root/.aws" amazon/aws-cli secretsmanager get-secret-value  --profile profile1 --no-cli-pager --secret-id secretsVault-bVMjveIsvBvb --output json --query SecretString
```