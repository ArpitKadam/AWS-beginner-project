# AWS SageMaker Project

## Project Overview
This project demonstrates how to set up and use AWS SageMaker Studio for machine learning tasks, including creating an S3 bucket, configuring a SageMaker Studio notebook instance, and executing workflows. All processes were conducted entirely on AWS servers, utilizing the cloud-based capabilities of SageMaker Studio for efficient development and execution.

## Requirements

1. **AWS Account**: Ensure you have an active AWS account.
2. **IAM Role**: Set up an IAM role with the necessary permissions for SageMaker, S3, and related AWS services.
3. **SageMaker Studio**: Access to AWS SageMaker Studio to manage and execute Jupyter notebooks.
4. **Python Environment**: Familiarity with Python and libraries like `boto3` and `sagemaker`.

## Setup Process

### 1. Create an S3 Bucket
1. Log in to your AWS Management Console.
2. Navigate to the **S3 Service**.
3. Click on **Create Bucket**.
4. Provide a unique name for the bucket (e.g., `sagemakerprojectbucket`) and select a region.
5. Leave other settings as default and click **Create Bucket**.

Alternatively, you can create the bucket programmatically:

```python
import boto3

bucket_name = "sagemakerprojectbucket2".lower()
region = boto3.session.Session().region_name
s3 = boto3.resource('s3')

try:
    if region == "us-east-1":
        s3.create_bucket(Bucket=bucket_name)
    else:
        s3.create_bucket(
            Bucket=bucket_name,
            CreateBucketConfiguration={'LocationConstraint': region}
        )
    print("S3 bucket created successfully")
except Exception as e:
    print("S3 error:", e)
```

### 2. Set Up a SageMaker Studio Notebook
1. Navigate to the **SageMaker Service** in the AWS Console.
2. Launch **SageMaker Studio** from the console.
3. Create a new user or select an existing one to start SageMaker Studio.
4. Use the SageMaker Studio interface to create and manage Jupyter notebooks.
5. Ensure that your SageMaker Studio user has an IAM role with access to S3 and SageMaker.

### 3. Define the Execution Role
In your SageMaker Studio notebook, define the execution role programmatically:

```python
import sagemaker

sagemaker_role = sagemaker.get_execution_role()
print("Role:", sagemaker_role)
```
This role is used to grant the notebook permissions to interact with AWS resources.

## Execution Steps

1. **Import Required Libraries**:
   ```python
   import sagemaker
   import boto3
   from sagemaker.amazon.amazon_estimator import get_image_uri
   from sagemaker.session import s3_input, Session
   ```

2. **Set Up Data**:
   - Upload your dataset to the S3 bucket.
   - Configure the SageMaker session to use the uploaded data.

3. **Train the Model**:
   Use SageMaker’s prebuilt algorithms or your custom model for training.
   ```python
   # Example for retrieving the container image
   container = get_image_uri(region, 'linear-learner')

   # Set up the estimator
   estimator = sagemaker.estimator.Estimator(
       container,
       sagemaker_role,
       instance_count=1,
       instance_type='ml.m4.xlarge',
       output_path=f's3://{bucket_name}/output',
       sagemaker_session=Session()
   )

   # Configure and start training
   estimator.fit({'train': f's3://{bucket_name}/train'})
   ```

4. **Deploy the Model**:
   Deploy the trained model to an endpoint for inference.
   ```python
   predictor = estimator.deploy(
       initial_instance_count=1,
       instance_type='ml.m4.xlarge'
   )
   ```

5. **Test the Model**:
   Test the deployed model with sample data.
   ```python
   result = predictor.predict(sample_input)
   print("Prediction:", result)
   ```

## Results and Next Steps

- **Results**: Analyze the predictions or performance metrics from the model.
- **Next Steps**: Refine the model, retrain with updated data, or integrate the model with a broader application.

## Notes

- Ensure that your S3 bucket and SageMaker Studio notebook are in the same region.
- Clean up resources after the project to avoid incurring additional costs:
  - Delete the S3 bucket.
  - Stop or delete the SageMaker Studio notebook instance.
  - Delete the deployed endpoint.
