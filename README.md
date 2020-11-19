# smart-door-rekognition
This smart recognizes new visitors and send OTP registration link in emails to the visitors. Returning visitors get direct OTP.

![Demo](https://github.com/sailikhithk/smart-door-rekognition/blob/main/frontend/opendoor.png)

# Architecture

![architecture](https://github.com/sailikhithk/smart-door-rekognition/blob/main/images/Screenshot%20from%202020-11-18%2019-16-11.png)

**Smart Door Rekognition System**

Name: Tanya Shah, Rutwik Chinchole, Ishan Tickoo, Sai Likhith Kanuparthi
NetID: tsah20, ruc197, it732, slk522

**S3 Bucket link:** 
* http://smartdoor-asg2.s3-website-us-east-1.amazonaws.com/index.html?fileName=frame_5c89c5c6-29fe-11eb-bc75-8671d7826d76.jpeg
* https://smartdoor-asg2.s3.amazonaws.com/virtualdoor.html

**GitHub Release:** 
18th Nov 2020

**Overview:**
1. For a given visitor (ex. new visitor), your system should be able to depict their face and email you to allow or deny them access. You will need to extract the      photo from individual fragments of the video stream. 
2. If allowed access, you should be able to capture their information through a hosted web page. You should then send them an SMS message with a valid OTP that is      only valid for a maximum of 5 minutes.
3. The OTP should be valid only once and guaranteed unique across the different visitors.
4. For a returning visitor (ex. the old user), your system should automatically send them an SMS message with a valid OTP.
5. Given a valid OTP, a visitor should be able to input it and receive a personalized greeting.
6. All other functionality should be working as described above.

**Steps:**

1. Visitor Vault
    - a. Create a S3 bucket (B1) to store the photos of the visitors.
    - b. Create a DynamoDB table “passcodes” (DB1) that stores temporary access codes to your virtual door and a reference to the visitor it was assigned to.
     - i. Use the TTL feature of DynamoDB to expire the records after 5 -1 minutes.
    - c. Create a DynamoDB table “visitors” (DB2) that stores details about the visitors that your Smart Door system is interacting with.
     - i. Index each visitor by the FaceId detected by Amazon Rekognition2 (more in the next section), alongside the name of the visitor and their phone number.            When storing a new face, if the FaceId returned by Rekognition already exists in the database, append the new photo to the existing photos array.
           Use the following schema for the JSON object:
              
              {
            
                “faceId”: “{UUID}”,
                “name”: “Jane Doe”,
                1 https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html
                2 https://aws.amazon.com/rekognition/
                “phoneNumber”: “+12345678901”,
                “photos”: [
                            {
                            “objectKey”: “my-photo.jpg”,
                            “bucket”: “my-photo-bucket”,
                            “createdTimestamp”:
                            “2018-11-05T12:40:02”
                            }
                          ]
            }
2. Analyze
      - a. Create a Kinesis Video Stream (KVS1), that will be used to capture and 3 stream video for analysis.
            i. Download the KVS Producer SDK GStreamer plugin
                i. We recommend you use the Docker image to run it, if you’re not comfortable with compiling the library yourself.
            ii. Get an IP camera or simulate one on your device to create an  RTSP video stream.
            iii. Run one of the GStreamer commands outlined in the GStreamer documentation to stream your RSTP source to Kinesis Video Streams.
      - b. Subscribe Rekognition Video to the Kinesis Video Stream (KVS1). 
      - c. Output the Rekognition Video analysis to a Kinesis Data Stream (KDS) and trigger a Lambda function (LF1) for every event that Rekognition Video outputs.
      - d. For every known face detected by Rekognition, send the visitor an SMS message to the phone number on file. The text message should include a PIN or a              One-Time Passcode (OTP) that they can use to open the virtual door.
          i. Store the OTP in the “passcodes” table (DB1), with a 5 minute expiration timestamp.
            A known face entails a face detected by Rekognition, whose FaceId can be found in the “visitors” DynamoDB table (DB1).
      - e. For every unknown face detected by Rekogniton, send an SMS to th “owner” (i.e. yourself or a team member) a photo of the visitor. The text 9
        message should also include a link to approve access for the visitor.
       - i. If clicked, the link should take you to a simple web page (WP1) that collects the name and phone number of the visitor via a web form.
            i. Submitting this form should create a new record in the “visitors” table (DB2), indexed by the FaceId identified by
                Rekognition. Note that you will have to build your own API to send information from the form to the backend. Its design
                and implementation is left up to you.
       - ii. Generate a OTP as in step (d) above and store it in the “passcodes” table (DB1), with a 5 minute expiration timestamp.
       - iii. Send the visitor an SMS message to the phone number on file. The text message should include the OTP.
3. Authorize
    - a. Create a second web page (WP2), the “virtual door”, that prompts the user to input the OTP.
     - i. If the OTP is valid, greet the user by name and present a success message.
     - ii. If the OTP is invalid, present a “permission denied” message.
    - b. Note that you will have to build your own API to capture and validate the OTP. Its design and implementation is left up to you.
