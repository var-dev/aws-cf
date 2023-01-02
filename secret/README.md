# Secret

## A cf template to create an aws secrets storage with an user that can access the entries. 
### Parameters:
  * sName - second part of the SM secret ID; first is the CF stack name.
  * KmsKeyArn (key_arn/'') - allows to use a non default key.
  * GrantAdmin (yes/no) - if you want the user/role to have adm privileges.

### Create stack
```
aws cloudformation create-stack --on-failure DELETE --profile awsa --template-body file://secretsVault.yaml --stack-name secretStore5 --capabilities CAPABILITY_IAM
```
### Deploy stack
```
aws cloudformation deploy --profile awsa --stack-name secretStore5 --template-file secretsVault.yaml --capabilities CAPABILITY_IAM
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
aws cloudformation describe-stack-events --no-cli-pager --profile awsa --stack-name secretStore5
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
#### CMD
```
docker run --rm -it -v "%userprofile%\.aws:/root/.aws" amazon/aws-cli secretsmanager get-secret-value  --profile profile1 --no-cli-pager --secret-id secretStore1Test101 --output json --query SecretString
```
#### Powershell
```
docker run --rm -it -v "$home\.aws:/root/.aws" amazon/aws-cli secretsmanager get-secret-value  --profile profile1 --no-cli-pager --secret-id secretStore1Test101 --output text --query SecretString
```

### Put secret values from file
```
aws secretsmanager put-secret-value  --profile profile1 --no-cli-pager --secret-string file://secret-example.yaml --secret-id secretStore1Test101
aws secretsmanager put-secret-value  --profile profile1 --no-cli-pager --secret-string file://secret-example.json --secret-id secretStore1Test101
```

### Read single secret (YAML)
```
docker run --rm -it -v "$home\.aws:/root/.aws" amazon/aws-cli secretsmanager get-secret-value  --profile profile1 --no-cli-pager --secret-id secretStore1Test101 --output text --query SecretString | python -c "import yaml,sys;print(yaml.safe_load(sys.stdin)['asd'])"
```

### Assume role
```
aws sts assume-role --profile awsa  --role-session-name role1 --role-arn arn:aws:iam::546145655640:role/secretStore1-secretsVaultRole-E85NRMUHC8S4
```
### Update a temp profile in $home\.aws\config $home\.aws\credentials
### Creating the ENV variables will do it too
```
Credentials:
[profileA]
aws_access_key_id = ASIAX6KGRLNMF5ON5W7C
aws_secret_access_key = k34FCMyLKDakYkdfHNj000pvJdTKvLIsdbD7kLH5
aws_session_token = IQoJb3JpZ2luX2VjED0aCXVzLXdlc3QtMiJHMEUCIA+xK
```
### Verify that you assumed the IAM role by running this command:
```
aws sts get-caller-identity --profile profileA
```
### Read single secret (JSON) using the assumed role
```
docker run --rm -it -v "$home\.aws:/root/.aws" amazon/aws-cli secretsmanager get-secret-value  --profile profileA --no-cli-pager --secret-id secretStore1Test101 --output text --query SecretString | python -c "import json,sys;print(json.load(sys.stdin)['zxc'])"
```

### A way to put the sts assume-role to work
```
export $(printf "AWS_ACCESS_KEY_ID=%s AWS_SECRET_ACCESS_KEY=%s AWS_SESSION_TOKEN=%s" \
$(aws sts assume-role \
--role-arn arn:aws:iam::123456789012:role/MyAssumedRole \
--role-session-name MySessionName \
--query "Credentials.[AccessKeyId,SecretAccessKey,SessionToken]" \
--output text))
```
#### Same in Powershell
```
$Env:AWS_ROLE_ARN=$(aws cloudformation describe-stacks --profile awsa --stack-name secretStore5 --query "Stacks[0].Outputs[?OutputKey=='secretsVaultRoleArn'][].OutputValue" --output text)
$Env:AWS_ROLE_TEMP1=$(aws sts assume-role --profile awsa  --role-session-name role1 --role-arn $Env:AWS_ROLE_ARN --query "Credentials.[AccessKeyId,SecretAccessKey,SessionToken]" --output text)

results in the \t separated list: key\tsecret\tsessionkey 
```