
import boto3
import json
from datetime import datetime, timedelta

def lambda_handler(event, context):
    print(event)
    # Extract bucket name and prefix from the event
    bucket_name = event['bucket']
    prefix = event['prefix']
    older_than_years = event.get('older_than_years', 8)

    delete_old_folders(bucket_name, prefix, older_than_years)

def delete_old_folders(bucket_name, prefix, older_than_years):
    s3 = boto3.client('s3')

    # Calculate cutoff date
    cutoff_date = datetime.now() - timedelta(days=older_than_years*365)

    # List objects in the bucket with the given prefix
    response = s3.list_objects_v2(Bucket=bucket_name, Prefix=prefix)

    # Iterate through objects and delete those older than cutoff date
    if 'Contents' in response:
        for obj in response['Contents']:
            key = obj['Key']
            last_modified = obj['LastModified'].replace(tzinfo=None)

            if last_modified < cutoff_date:
                print(f"Deleting object: {key}")
                s3.delete_object(Bucket=bucket_name, Key=key)

# Example event JSON:
{
   "bucket": "anuragoldbucket123",
    "prefix": "converted/",
    "older_than_years": 8
}
