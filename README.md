# Creating CX Oracle AWS Lambda Layer Package

1. IAM Role
     - First, you need to Create an IAM Role for Amazon EC2 and attach the AWS managed permissions "AWSLambdaFullAccess" and "AmazonS3FullAccess". 
     - You will need this role to interact with AWS services using the AWS CLI later on in this post.
2. EC2 Instance
     - Next, Launch an Amazon EC2 Instance with Amazon Linux 2 AMI from AWS Lambda Runtimes and attach the recently created IAM Role to the instance [https://docs.aws.amazon.com/quickstarts/latest/vmlaunch/step-1-launch-instance.html].
     - If you already have an Amazon EC2 instance you can Attach the IAM Role to the existing Instance. 
     - The reason for using Amazon Linux 2 is that it comes with Python 3.7 available in the repositories.
3. Now you can connect to your Linux Instance using an SSH Client and start building the AWS Lambda Layer.

    a. Install and configure the AWS CLI [https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html].
    
    b. Update the existing system packages and install Python 3.7
  
        $ sudo yum update -y
        $ sudo yum install python3 -y
        
    c. Create a working directory and structure it by creating the "python" and "lib" folders which are required for a Python Layer.
  
        $ mkdir ~/my_layer
        $ cd ~/my_layer
        $ mkdir python/ lib/
        
    d. Install the "cx_Oracle" module using "pip-3.7" under "python/" directory. The function code will access it without additional configuration.
  
        $ pip-3.7 install cx_Oracle -t python/ 
        
    e. Download the latest version of "Oracle Client" libraries and extract the contents to the "lib/" directory [https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html]

        $ wget https://download.oracle.com/otn_software/linux/instantclient/193000/instantclient-basic-linux.x64-19.3.0.0.0dbru.zip -O oracle.zip
        $ unzip -j oracle.zip -d lib/

    f. Install the "libaio" library and copy it to the "lib/" folder of the layer. This library is preinstalled in Amazon Linux 2, if you are using it ycan skip the installation.
  
        $ (optional) sudo yum install libaio -y
        $ cp /lib64/libaio.so.1 lib/libaio.so.1

    g. Create a ZIP archive with "python/" and "lib/" folders.
  
        $ zip -r -y layer.zip python/ lib/

    h. Copy the local archive to an Amazon S3 bucket. Make sure the bucket exists first. I created a bucket called awsexamplebucket.

        $ aws s3 cp layer.zip s3://awsexamplebucket/layer.zip --region us-west-2

    i. Create an AWS Lambda Layer using the uploaded archive as the source [https://docs.aws.amazon.com/cli/latest/reference/lambda/publish-layer-version.html].

        $ aws lambda publish-layer-version --layer-name cx_Oracle --description "My cx_Oracle layer" --content S3Bucket=awsexamplebucket,S3Key=layer.zipcompatible-runtimes python3.7 --region us-west-2
