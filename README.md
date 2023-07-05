resource "aws_redshift_cluster" "redshift" {

  // Redshift cluster configuration



  // IAM role for Redshift cluster

  iam_roles = [aws_iam_role.redshift_role.arn]

}



resource "aws_iam_role" "redshift_role" {

  name = "redshift-role"



  // Attach necessary policies to the role

  // For example:

  assume_role_policy = <<EOF

{

  "Version": "2012-10-17",

  "Statement": [

    {

      "Effect": "Allow",

      "Principal": {

        "Service": "redshift.amazonaws.com"

      },

      "Action": "sts:AssumeRole"

    }

  ]

}

EOF



  // Attach policies to the role

  // For example:

  policy {

    policy = jsonencode({

      "Version": "2012-10-17",

      "Statement": [

        {

          "Effect": "Allow",

          "Action": [

            "redshift:DescribeClusters",

            "redshift:DescribeCluster*",

            // Add other necessary actions for Redshift

          ],

          "Resource": "*"

        }

      ]

    })

  }

}



output "redshift_cluster_id" {

  value = aws_redshift_cluster.redshift.id

}



resource "aws_secretsmanager_secret" "secrets_manager" {

  // Secrets Manager secret configuration



  // IAM policy for Secrets Manager secret

  policy = data.aws_iam_policy_document.secrets_manager_policy.json

}



data "aws_iam_policy_document" "secrets_manager_policy" {

  statement {

    actions = [

      "secretsmanager:GetSecretValue",

      "secretsmanager:DescribeSecret",

      // Add other necessary actions for Secrets Manager

    ]



    resources = [

      aws_secretsmanager_secret.secrets_manager.arn,

    ]

  }

}



output "secrets_manager_secret_id" {

  value = aws_secretsmanager_secret.secrets_manager.id

}





resource "aws_lambda_function" "lambda" {

  // Lambda function configuration



  // IAM role for Lambda function

  role = aws_iam_role.lambda_role.arn



  // ... Other Lambda function configurations

}



