Step-by-Step Guide
1. Setting Up the Raspberry Pi
Hardware and Software Requirements
Raspberry Pi (any model with internet capability)
Python installed (usually pre-installed in Raspberry Pi OS)
Libraries: paho-mqtt, ssl
Install required Python libraries on your Raspberry Pi using the following command:

bash
Copy code
pip install paho-mqtt
Python Code for Raspberry Pi
This code sends sensor data (e.g., temperature and humidity) to AWS IoT Core using the MQTT protocol.

python
Copy code
import paho.mqtt.client as mqtt
import ssl
import json
import time

# Configuration
client_id = "RaspberryPiClient"
endpoint = "<YOUR_IOT_ENDPOINT>"  # Replace with your AWS IoT Core endpoint
root_ca = "<PATH_TO_YOUR_ROOT_CA>"  # Path to root-CA.crt file
private_key = "<PATH_TO_YOUR_PRIVATE_KEY>"  # Path to private.pem.key file
certificate = "<PATH_TO_YOUR_CERTIFICATE>"  # Path to certificate.pem.crt file
topic = "raspberrypi/sensorData"

# Example data
data = {
    "temperature": 25.6,
    "humidity": 60.5,
    "timestamp": time.time()
}

# MQTT Client Setup
mqtt_client = mqtt.Client(client_id)
mqtt_client.tls_set(
    root_ca,
    certfile=certificate,
    keyfile=private_key,
    cert_reqs=ssl.CERT_REQUIRED,
    tls_version=ssl.PROTOCOL_TLSv1_2
)
mqtt_client.connect(endpoint, 8883)

# Publishing data
mqtt_client.publish(topic, json.dumps(data))
print("Data published to AWS IoT Core")

mqtt_client.disconnect()
2. Configuring AWS Services
2.1 AWS IoT Core Configuration
Create a Thing in AWS IoT Core:

Go to the AWS IoT Core console.
Click Manage > Things > Create Things > Create single thing.
Name your Thing (e.g., RaspberryPiThing).
Create Certificates:

After creating the Thing, create certificates for secure communication:
Choose Certificates > Create certificate > Create with an AWS IoT certificate.
Download the following files:
Device Certificate (certificate.pem.crt)
Private Key (private.pem.key)
Amazon Root CA 1 (AmazonRootCA1.pem)
Activate the certificate.
Attach a Policy:

Create a policy that allows the Raspberry Pi to publish messages:
Go to Secure > Policies > Create policy.
Add policy actions:
Action: iot:Publish
Resource ARN: arn:aws:iot:<REGION>:<ACCOUNT_ID>:topic/raspberrypi/sensorData
Effect: Allow
Attach the policy to the certificate.
Set Up IoT Rule:

Create an IoT rule to trigger Lambda when data is published:
Go to Act > Rules > Create.
Name your rule (e.g., RaspberryPiRule).
SQL Statement: SELECT * FROM 'raspberrypi/sensorData'.
Set the action to invoke the Lambda function created in the next steps.
2.2 AWS Lambda Configuration
Create a Lambda Function:

Go to the AWS Lambda console.
Click Create Function > Author from scratch.
Name the function (e.g., ProcessSensorData).
Runtime: Python 3.x.
Execution role: Create a new role with basic Lambda permissions.
IAM Role Configuration for Lambda:

After creating the Lambda function, modify the role:
Go to IAM > Roles > find your Lambda execution role.
Attach policies for DynamoDB and S3:
AmazonDynamoDBFullAccess
AmazonS3FullAccess
AWSIoTDataAccess (if necessary)
Lambda Code: This code processes incoming data and stores it in DynamoDB and S3.

python
Copy code
import json
import boto3
import time

# Initialize AWS clients
dynamodb = boto3.resource('dynamodb')
s3 = boto3.client('s3')

# DynamoDB table and S3 bucket
dynamodb_table_name = 'SensorData'
s3_bucket_name = 'raspberry-pi-data-bucket'

def lambda_handler(event, context):
    # Extract data from the event
    sensor_data = json.loads(event['body'])

    # Store data in DynamoDB
    table = dynamodb.Table(dynamodb_table_name)
    table.put_item(
        Item={
            'timestamp': str(time.time()),
            'temperature': sensor_data.get('temperature'),
            'humidity': sensor_data.get('humidity')
        }
    )

    # Save data to S3
    s3.put_object(
        Bucket=s3_bucket_name,
        Key=f"data_{int(time.time())}.json",
        Body=json.dumps(sensor_data)
    )

    return {
        'statusCode': 200,
        'body': json.dumps('Data stored successfully')
    }
2.3 DynamoDB Configuration
Create a DynamoDB Table:
Go to the DynamoDB console.
Click Create Table.
Table name: SensorData.
Primary key: timestamp (String).
Configure read/write capacity settings as needed.
2.4 S3 Configuration
Create an S3 Bucket:
Go to the S3 console.
Click Create Bucket.
Name the bucket (e.g., raspberry-pi-data-bucket).
Configure bucket permissions based on your needs (ensure access control).
IAM Configuration
IAM User and Role Configuration
Create IAM User:

Go to the IAM console.
Click Users > Add User.
Username: RaspberryPiUser.
Access type: Programmatic access.
Attach policies:
AWSIoTFullAccess
AmazonDynamoDBFullAccess
AmazonS3FullAccess
Create IAM Role for Lambda Execution:

Go to Roles > Create Role.
Trusted entity: AWS service > Lambda.
Attach required policies as mentioned earlier.
Testing the Complete Setup
Deploy the Raspberry Pi Code:

Ensure your Raspberry Pi has the correct endpoint, certificates, and paths configured.
Run the Python script to publish data.
Monitor AWS IoT Core:

Verify that messages are received in the AWS IoT Core.
Check AWS Lambda Logs:

Go to CloudWatch Logs and check the logs for your Lambda function.
Verify Data in DynamoDB and S3:

Check DynamoDB to ensure data is recorded.
Verify that S3 has stored the JSON files as expected.