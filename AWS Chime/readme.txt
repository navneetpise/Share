# AWS Chime SDK Setup on Raspberry Pi with Mobile Browser Access

This project demonstrates how to set up AWS Chime SDK on a Raspberry Pi and connect it to a mobile phone browser for video and audio streaming. The project uses Node.js to create a backend server that interacts with AWS Chime SDK, enabling meetings and attendees management.

## Prerequisites

1. **Raspberry Pi**: Set up with Raspbian OS.
2. **AWS Account**: [Sign up here](https://aws.amazon.com) if you don't have one.
3. **AWS CLI**: Installed and configured on the Raspberry Pi.
4. **Node.js**: Installed on the Raspberry Pi.

## AWS Setup

### Step 1: AWS Configuration

1. **Create IAM Role and Policy**:
   - Navigate to the AWS IAM console.
   - Create a new role with permissions to use AWS Chime SDK.
   - Attach `AmazonChimeSDK` and `AmazonChimeSDKMediaPipelines` policies.

2. **AWS Chime SDK**:
   - Go to the [Amazon Chime SDK Console](https://console.aws.amazon.com/chime/home).
   - Create a new application and note the application ID and region.

3. **AWS Lambda Functions (Optional)**:
   - Set up Lambda functions if you need server-side logic for meeting and attendee management.
   - Use AWS API Gateway to expose Lambda functions as RESTful APIs.

## Raspberry Pi Setup

### Step 2: Raspberry Pi Configuration

1. **Update Raspberry Pi**:
   ```bash
   sudo apt update && sudo apt upgrade
Install Node.js:

bash
Copy code
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs
Install AWS CLI:

bash
Copy code
sudo apt install awscli
aws configure
Provide your AWS access key, secret key, region, and output format when prompted.
Install Required Libraries:

bash
Copy code
sudo apt-get install libasound2-dev
Project Setup
Step 3: Create a Node.js Application
Set Up Node.js Project:

bash
Copy code
mkdir chime-raspberry
cd chime-raspberry
npm init -y
npm install aws-sdk amazon-chime-sdk-js express
Create server.js File: Create a server.js file with the following content:

javascript
Copy code
const express = require('express');
const { ChimeSDKMeetingsClient, CreateMeetingCommand, CreateAttendeeCommand } = require('@aws-sdk/client-chime-sdk-meetings');

const app = express();
const port = 3000;

const chimeClient = new ChimeSDKMeetingsClient({ region: 'us-east-1' });

app.use(express.json());

app.post('/createMeeting', async (req, res) => {
    try {
        const meetingResponse = await chimeClient.send(new CreateMeetingCommand({ ClientRequestToken: `meeting-${Date.now()}` }));
        res.json({ Meeting: meetingResponse.Meeting });
    } catch (error) {
        res.status(500).send(error.message);
    }
});

app.post('/createAttendee', async (req, res) => {
    try {
        const { MeetingId, ExternalUserId } = req.body;
        const attendeeResponse = await chimeClient.send(new CreateAttendeeCommand({ MeetingId, ExternalUserId }));
        res.json({ Attendee: attendeeResponse.Attendee });
    } catch (error) {
        res.status(500).send(error.message);
    }
});

app.listen(port, () => {
    console.log(`Server running at http://localhost:${port}`);
});
Run the Server: Start the server on Raspberry Pi using:

bash
Copy code
node server.js
Step 4: Mobile Browser Setup
Create index.html File: Create an index.html file with the following content:

html
Copy code
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chime Meeting</title>
</head>
<body>
    <button id="joinMeeting">Join Meeting</button>

    <script>
        document.getElementById('joinMeeting').addEventListener('click', async () => {
            // Create Meeting
            const meetingResponse = await fetch('http://<Raspberry_Pi_IP>:3000/createMeeting', { method: 'POST' });
            const meeting = await meetingResponse.json();

            // Create Attendee
            const attendeeResponse = await fetch('http://<Raspberry_Pi_IP>:3000/createAttendee', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ MeetingId: meeting.Meeting.MeetingId, ExternalUserId: 'user@domain.com' })
            });

            const attendee = await attendeeResponse.json();
            console.log('Joined Meeting:', meeting, attendee);
        });
    </script>
</body>
</html>
Access the HTML Page on Mobile:

Host the index.html file using a simple HTTP server on the Raspberry Pi or upload it to a static website hosting service like AWS S3.
Access the page from your mobile browser using the Raspberry Pi’s IP address.
Testing and Troubleshooting
Ensure AWS credentials are correctly set up.
Check network configurations for access permissions.
Verify that the server on Raspberry Pi is running and accessible from the mobile browser.