AWS Cloud Computing Midterm Guide 🚀
A comprehensive, step-by-step guide covering six key tasks for an AWS cloud computing lab midterm. This guide provides all necessary steps, commands, and code snippets for each task.

Table of Contents
Hosting Static Websites on AWS S3 and EC2
EC2 Setup and MySQL Database Management
Web Application Deployment using AWS Elastic Beanstalk
Serverless Computing – S3 and Lambda Integration
EC2 Auto Scaling using Launch Templates and Scaling Policies
S3 Bucket File Management and Public Access Configuration
1. Hosting Static Websites on AWS S3 and EC2
This section covers hosting a static website using two different methods: AWS S3 for a serverless approach and an EC2 instance for a traditional server-based approach.

Part A: Hosting on AWS S3 📁
This method is serverless, cost-effective, and highly scalable for static content.

Create an S3 Bucket:

Navigate to the S3 console in AWS.
Click Create bucket.
Give it a globally unique name (e.g., my-unique-midterm-website).
Under Block Public Access settings for this bucket, uncheck Block all public access and acknowledge the warning.
Click Create bucket.
Upload Your Website Files:

Click on your new bucket and upload your index.html and error.html files.
Enable Static Website Hosting:

Go to the Properties tab of your bucket.
Scroll to Static website hosting and click Edit.
Select Enable, set index.html as the index document and error.html as the error document, and save changes.
Add a Bucket Policy for Public Access:

Go to the Permissions tab and edit the Bucket policy.
Paste the following JSON, replacing my-unique-midterm-website with your bucket name.
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-unique-midterm-website/*"
        }
    ]
}
Save the policy. Your website is now live at the endpoint URL found in the Static website hosting section.
Part B: Hosting on an EC2 Instance 🖥️
This method gives you a full virtual server for more control.

Launch an EC2 Instance:

Navigate to the EC2 console and click Launch instances.
Choose an AMI like Amazon Linux 2 and an instance type like t2.micro.
Create and download a new key pair.
Configure Security Group:

In the Network settings, create a security group that allows inbound traffic on SSH (Port 22) from your IP and HTTP (Port 80) from anywhere (0.0.0.0/0).
Launch and Connect:

Launch the instance and connect to it using SSH with the key pair you downloaded.
Install and Start a Web Server:

Once connected, run these commands to install and start the Apache web server:
# Update packages
sudo yum update -y

# Install Apache
sudo yum install -y httpd

# Start and enable Apache service
sudo systemctl start httpd
sudo systemctl enable httpd
Create Your Website File:

Create a simple index.html file in Apache's web root:
echo "<h1>Hello from my EC2 Web Server!</h1>" | sudo tee /var/www/html/index.html
Access your site by pasting the instance's Public IPv4 address into your browser.
2. EC2 Setup and MySQL Database Management
This task involves launching an EC2 instance and setting up a MySQL server on it, complete with a database, stored procedures, and triggers.

Launch and Connect to EC2:

Launch an Amazon Linux 2 EC2 instance as described above.
In the security group, allow SSH (Port 22) and MySQL/Aurora (Port 3306) from My IP.
Install MySQL Server:

Connect via SSH and run the following commands:
# Install MySQL repository
sudo wget [https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm](https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm)
sudo rpm -ivh mysql80-community-release-el7-3.noarch.rpm
sudo yum update -y

# Install MySQL server
sudo yum install -y mysql-community-server
Start and Secure MySQL:

Start the MySQL service: sudo systemctl start mysqld.
Find the temporary root password: sudo grep 'temporary password' /var/log/mysqld.log.
Run the security script and follow the prompts to set a new password: sudo mysql_secure_installation.
Create Database and Tables:

Log in to MySQL: mysql -u root -p.
Run the following SQL commands:
CREATE DATABASE company;
USE company;

CREATE TABLE Employees (
    emp_id INT PRIMARY KEY AUTO_INCREMENT,
    emp_name VARCHAR(100),
    salary DECIMAL(10, 2)
);

CREATE TABLE SalaryAudit (
    audit_id INT PRIMARY KEY AUTO_INCREMENT,
    emp_id INT,
    old_salary DECIMAL(10, 2),
    new_salary DECIMAL(10, 2),
    change_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
Create a Stored Procedure and Trigger:

While in the MySQL prompt, run these commands:
-- Stored Procedure to add a new employee
DELIMITER $$
CREATE PROCEDURE AddEmployee(IN name VARCHAR(100), IN initial_salary DECIMAL(10, 2))
BEGIN
    INSERT INTO Employees (emp_name, salary) VALUES (name, initial_salary);
END$$
DELIMITER ;

-- Trigger to log salary updates
DELIMITER $$
CREATE TRIGGER before_employee_update
BEFORE UPDATE ON Employees
FOR EACH ROW
BEGIN
    IF OLD.salary <> NEW.salary THEN
        INSERT INTO SalaryAudit(emp_id, old_salary, new_salary)
        VALUES(OLD.emp_id, OLD.salary, NEW.salary);
    END IF;
END$$
DELIMITER ;
Test Your Work:

Call the procedure: CALL AddEmployee('Charlie', 80000.00);
Update a salary to test the trigger: UPDATE Employees SET salary = 65000.00 WHERE emp_name = 'Alice';
Verify the results with SELECT * FROM Employees; and SELECT * FROM SalaryAudit;.
3. Web Application Deployment using AWS Elastic Beanstalk
Elastic Beanstalk (EB) automates the deployment and scaling of web applications.

Prepare Your Application:

Create a folder flask-app containing two files:
application.py:
from flask import Flask
application = Flask(__name__)
@application.route('/')
def hello_world():
    return 'Hello, World! This is my Elastic Beanstalk App.'
requirements.txt:
Flask==2.0.1
Create a zip file containing these two files (not the parent folder).
Create an EB Application:

Navigate to the Elastic Beanstalk console and click Create application.
Enter an Application name.
Configure and Deploy:

Choose Web server environment and select the Python platform.
Select Upload your code and upload the zip file you created.
Click Create application. EB will provision the environment, which takes a few minutes.
Once the environment health is "Ok", click the URL at the top of the dashboard to see your live app.
4. Serverless Computing – S3 and Lambda Integration
Create an event-driven workflow where uploading a file to S3 automatically triggers a Lambda function.

Create an S3 Bucket:

Create a new S3 bucket (e.g., my-lambda-trigger-bucket) with default settings (public access blocked).
Create an IAM Role for Lambda:

Go to the IAM console -> Roles -> Create role.
Select AWS service for the trusted entity and Lambda for the use case.
Attach the AWSLambdaS3ExecutionRole permission policy.
Name the role (e.g., LambdaS3AccessRole) and create it.
Create the Lambda Function:

Navigate to the Lambda console and click Create function.
Select Author from scratch, give it a name, and choose a Python runtime.
Under Permissions, choose Use an existing role and select the LambdaS3AccessRole you just created.
Add the Lambda Code:

In the code editor, paste the following Python code and click Deploy.
import json

def lambda_handler(event, context):
    bucket_name = event['Records'][0]['s3']['bucket']['name']
    file_key = event['Records'][0]['s3']['object']['key']
    print(f"File '{file_key}' was uploaded to bucket '{bucket_name}'.")
    return {
        'statusCode': 200,
        'body': json.dumps(f'Successfully processed {file_key}!')
    }
Add the S3 Trigger:

In the function overview, click Add trigger.
Select S3 as the source, choose your bucket, and set the event type to All object create events.
Test the Integration:

Upload any file to your S3 bucket.
Check the function's logs in CloudWatch (via the Monitor tab) to confirm it was triggered successfully.
5. EC2 Auto Scaling using Launch Templates and Scaling Policies
Auto Scaling ensures your application can automatically scale in and out to meet demand.

Create a Launch Template:

Navigate to EC2 -> Launch Templates -> Create launch template.
Give it a name (e.g., MyWebAppTemplate).
Choose an AMI (Amazon Linux 2), instance type (t2.micro), and a key pair.
Select a security group that allows HTTP traffic.
Expand Advanced details and paste the following into the User data field to set up a web server on launch:
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from ASG Instance</h1>" > /var/www/html/index.html
Click Create launch template.
Create an Auto Scaling Group (ASG):

Navigate to EC2 -> Auto Scaling Groups -> Create Auto Scaling group.
Name the ASG and select the launch template you just created.
Choose a VPC and at least two subnets for high availability.
Set the Desired capacity to 2, Minimum to 1, and Maximum to 4.
Configure a Scaling Policy:

Under Scaling policies, select Target tracking scaling policy.
Set the Metric type to Average CPU utilization and the Target value to 50.
Finish creating the Auto Scaling group.
Verify:

The ASG will launch two instances. If you stress the CPU on one instance (e.g., using the stress tool), the ASG will detect the high average CPU usage and automatically launch a new instance to handle the load.
6. S3 Bucket File Management and Public Access Configuration
This task covers the fundamentals of S3 file storage and public access control.

Create an S3 Bucket:

Create a new S3 bucket with a globally unique name.
Keep the default setting Block all public access checked initially.
Upload a File and Confirm Private Access:

Upload any file to the bucket.
Click the file, copy its Object URL, and paste it into your browser. You will see an AccessDenied error message.
Allow Public Access:

Go to the bucket's Permissions tab.
Edit the Block public access (bucket settings) and uncheck Block all public access. Save the changes.
Add a Bucket Policy:

In the Bucket policy section, click Edit.
Paste the following JSON policy, replacing your-bucket-name with your actual bucket name.
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*"
    }
  ]
}
Save the policy.
Verify Public Access:

Refresh the browser tab with the object URL. The file should now be visible. ✅
