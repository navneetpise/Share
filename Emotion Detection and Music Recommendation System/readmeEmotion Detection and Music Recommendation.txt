Emotion Detection and Music Recommendation System
This project uses a Raspberry Pi with a camera module and a 7-inch screen to detect emotions and recommend music from YouTube based on the detected mood. The system utilizes OpenCV for face detection, a pre-trained deep learning model for emotion recognition, and the YouTube Data API to fetch relevant music recommendations.

Table of Contents
Requirements
Setup and Installation
Model Training and Pre-Trained Model
Running the Program
Detailed Workflow
Troubleshooting
Acknowledgements
Requirements
Raspberry Pi: Raspberry Pi 4 Model B or later recommended.
Camera Module: Raspberry Pi Camera Module V2 or compatible USB camera.
7-Inch Display: Official Raspberry Pi 7-inch touchscreen display or any compatible HDMI display.
Internet Connectivity: Required for fetching YouTube recommendations.
Software: Python 3.x, TensorFlow, Keras, OpenCV, and Google API client.
Setup and Installation
Step 1: System Update
Update your Raspberry Pi’s package list and upgrade installed packages:

bash
Copy code
sudo apt-get update
sudo apt-get upgrade
Step 2: Install Python and Dependencies
Install Python 3 and necessary libraries:

bash
Copy code
sudo apt-get install python3-pip python3-dev python3-opencv
pip3 install numpy keras tensorflow google-api-python-client
Step 3: Enable Camera
Enable the Raspberry Pi Camera Module:

Open the Raspberry Pi configuration tool:
bash
Copy code
sudo raspi-config
Navigate to Interface Options > Camera and enable it.
Reboot the Raspberry Pi:
bash
Copy code
sudo reboot
Step 4: Obtain YouTube API Key
Go to the Google Cloud Console.
Create a new project or select an existing one.
Enable the YouTube Data API v3.
Generate an API key and copy it for use in the script.
Step 5: Download the Pre-trained Emotion Model
Download the emotion_model.h5 file (you can train your model using facial expression datasets like FER2013 or use a pre-trained model) and place it in the project directory.

Model Training and Pre-Trained Model
Training Your Emotion Detection Model (Optional)
If you want to train your own model:

Dataset: Use datasets like FER2013 or CK+.
Model Structure: Use Convolutional Neural Networks (CNNs) for training. Preprocessing should include resizing images to 48x48 pixels and normalizing pixel values.
Training Script: Train using Keras/TensorFlow with an architecture such as VGG16 or a custom CNN.
Save the trained model as emotion_model.h5 and use it in this project.

Running the Program
Step 1: Prepare the Script
Copy the combined Python script (provided earlier) into a file named emotion_detection.py.
Replace 'YOUR_YOUTUBE_API_KEY' with your actual YouTube Data API key.
Step 2: Execute the Script
Run the script on the Raspberry Pi:

bash
Copy code
python3 emotion_detection.py
Step 3: Using the System
The program will start capturing video from the camera.
It will detect faces and emotions in real-time and display the detected emotion on the 7-inch screen.
Music recommendations will be printed based on the detected emotion, with URLs for YouTube.
Detailed Workflow
Face Detection: The system captures video frames from the camera and uses OpenCV’s Haar Cascade classifier to detect faces.

Emotion Detection: For each detected face, the image is preprocessed (grayscale, resized) and fed into the pre-trained emotion recognition model to predict the emotion.

Music Recommendation: The detected emotion is used to query the YouTube Data API for relevant music. Recommendations are updated continuously based on the current mood.

User Interface: Detected emotions and the recommended music list are displayed on the screen.

Troubleshooting
Camera Not Detected: Ensure the camera is correctly connected and enabled in the Raspberry Pi configuration settings.
Low FPS/Slow Performance: Use a lightweight model for emotion detection or reduce the frame size for faster processing.
YouTube API Errors: Check your API key, ensure the YouTube Data API v3 is enabled, and verify that there are no quota limits reached.
Missing Dependencies: Double-check that all required libraries are installed with correct versions.
Acknowledgements
The face detection model uses OpenCV's Haar Cascade classifier.
The emotion recognition model is a deep learning model trained using Keras and TensorFlow.
Music recommendations are fetched using YouTube Data API v3.
