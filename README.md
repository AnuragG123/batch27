To create an AWS Lambda function with Python 3.11 runtime using Terraform, you can modify the previous Terraform configuration to specify the runtime as `python3.11` for the Lambda function. Here's how you can adjust the Terraform configuration:

### Step 1: Update AWS Provider in Terraform

Ensure you have the AWS provider configured:

```hcl
provider "aws" {
  region = "us-east-1"  # Replace with your desired AWS region
}
```

### Step 2: Define IAM Role for Lambda Execution

Create the IAM role and policy as before:

```hcl
resource "aws_iam_role" "lambda_role" {
  name = "lambda-execution-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect    = "Allow"
        Principal = {
          Service = "lambda.amazonaws.com"
        }
        Action    = "sts:AssumeRole"
      }
    ]
  })
}

resource "aws_iam_policy" "lambda_policy" {
  name        = "lambda-policy"
  description = "Policy for Lambda function"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Action   = [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Resource = "*"
      },
      {
        Effect   = "Allow"
        Action   = [
          "s3:ListBucket",
          "s3:GetObject",
          "s3:DeleteObject"
        ]
        Resource = "*"  # Adjust to specific bucket ARNs if needed
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "lambda_attachment" {
  role       = aws_iam_role.lambda_role.name
  policy_arn = aws_iam_policy.lambda_policy.arn
}
```

### Step 3: Define Lambda Function

Update the Lambda function configuration to use Python 3.11 runtime:

```hcl
data "archive_file" "lambda_zip" {
  type        = "zip"
  source_dir  = "path/to/your/lambda/function"  # Path to your Lambda function code
  output_path = "lambda_function.zip"
}

resource "aws_lambda_function" "my_lambda_function" {
  function_name    = "my-lambda-function"
  filename         = data.archive_file.lambda_zip.output_path
  source_code_hash = data.archive_file.lambda_zip.output_base64sha256
  handler          = "lambda_function.lambda_handler"
  runtime          = "python3.11"  # Python 3.11 runtime
  role             = aws_iam_role.lambda_role.arn
  timeout          = 300  # Adjust as needed
  memory_size      = 128  # Adjust as needed

  environment {
    variables = {
      LOG_LEVEL = "INFO"
    }
  }
}
```

### Step 4: Create API Gateway (Optional)

If you want to create an API Gateway trigger for your Lambda function, you can follow the previous steps to define API Gateway resources and integrate them with your Lambda function.

### Step 5: Terraform Configuration

Combine all configurations into your `main.tf` file:

```hcl
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"  # Replace with your desired AWS region
}

resource "aws_iam_role" "lambda_role" {
  # Role configuration as defined earlier
}

resource "aws_iam_policy" "lambda_policy" {
  # Policy configuration as defined earlier
}

resource "aws_iam_role_policy_attachment" "lambda_attachment" {
  # Attachment configuration as defined earlier
}

data "archive_file" "lambda_zip" {
  # Lambda code archive configuration as defined earlier
}

resource "aws_lambda_function" "my_lambda_function" {
  # Lambda function configuration as defined earlier
}

# Optional: API Gateway configuration as defined earlier

# Optional: API Gateway integration, permissions, and deployment as defined earlier
```

### Step 6: Initialize and Apply Terraform Configuration

Initialize Terraform and apply the configuration:

```bash
terraform init
terraform apply
```

This Terraform configuration sets up an AWS Lambda function with Python 3.11 runtime, along with associated IAM roles and policies. Adjust paths, names, and other configurations as per your specific requirements before running `terraform apply`. If you need to add API Gateway integration, you can refer to the previous example and integrate it accordingly.
