# Example: using credential system to access CPG Staging bucket

## Add provided credentials to `~/.aws/credentials`

```
[cpg_staging]
aws_access_key_id = {ACCESS_KEY_ID} 
aws_secret_access_key = {SECRET_ACCESS_KEY} 
```

## Create file called `s3_credentials.sh`

This will get and activate temporary data credentials:
```
export AWS_PROFILE=cpg_staging
export AWS_REGION=us-east-1

# Run the AWS CLI command and capture the output
output=$(aws s3control get-data-access \
  --account-id 309624411020 \
  --target "s3://staging-cellpainting-gallery/*" \
  --permission READWRITE \
  --privilege Default \
  --region $AWS_REGION)

# Check if the command was successful
if [ $? -eq 0 ]; then
  # Parse the output to extract credentials
  AccessKeyId=$(echo "$output" | jq -r '.Credentials.AccessKeyId')
  SecretAccessKey=$(echo "$output" | jq -r '.Credentials.SecretAccessKey')
  SessionToken=$(echo "$output" | jq -r '.Credentials.SessionToken')

  # Export the credentials as environment variables
  export AWS_ACCESS_KEY_ID=$AccessKeyId
  export AWS_SECRET_ACCESS_KEY=$SecretAccessKey
  export AWS_SESSION_TOKEN=$SessionToken

  echo "AWS credentials have been set successfully."
else
  echo "Failed to get data access information."
  exit 1
fi
```

## Activate credentials
Let's say you are running `python run.py`:

Option 1: Activate credentials for the rest of the shell (Note: generated credentials only valid for 1-12 hours, flag can be set in script above)
```
source s3_credentials.sh
python run.py
```
Option 2: Create subshell to only activate credentials for command (recommended)
```
(source s3_credentials; python run.py)
```

