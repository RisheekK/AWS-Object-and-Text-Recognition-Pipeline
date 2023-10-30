# AWS-Object-and-Text-Recognition-Pipeline

Github link: https://github.com/RisheekK/AWS-Object-and-Text-Recognition-Pipeline

<p><b>Goal:</b> The purpose of this individual assignment is to learn how to use the Amazon AWS cloud platform and how to develop an AWS application that uses existing cloud services. Specifically, you will learn: (1) how to create VMs (EC2 instances) in the cloud; (2) how to use cloud storage (S3) in your applications; (3) how to communicate between VMs using a queue service (SQS); (4) how to program distributed applications in Java on Linux VMs in the cloud; and (5) how to use a machine learning service (AWS Rekognition) in the cloud.</p>

<b>Description:</b> You have to build an image recognition pipeline in AWS, using two EC2 instances, S3, SQS, and Rekognition. The assignment must be done in Java on Amazon Linux VMs. For the rest of the description, you should refer to the figure below:

![image](https://github.com/RisheekK/AWS-Object-and-Text-Recognition-Pipeline/assets/86208506/f9969692-0a17-4875-8e4d-b8bfe6c4ae57)

You have to create 2 EC2 instances (EC2 A and B in the figure), with Amazon Linux AMI, that will work in parallel. Each instance will run a Java application. Instance A will read 10 images from an S3 bucket that we created (https://njit-cs-643.s3.us-east-1.amazonaws.com) and perform object detection in the images. When a car is detected using Rekognition, with confidence higher than 90%, the index of that image (e.g., 2.jpg) is stored in SQS. Instance B reads indexes of images from SQS as soon as these indexes become available in the queue, and performs text recognition on these images (i.e., downloads them from S3 one by one and uses Rekognition for text recognition). Note that the two instances work in parallel: for example, instance A is processing image 3, while instance B is processing image 1 that was recognized as a car by instance A. When instance A terminates its image processing, it adds index -1 to the queue to signal to instance B that no more indexes will come. When instance B finishes, it prints to a file, in its associated EBS, the indexes of the images that have both cars and text, and also prints the actual text in each image next to its index.

## Initial Setup
1) Log in to your AWS Academy account using the credentials provided by your professor and navigate to your course. Modules > Launch AWS Learner Lab allows you to access the vocareum learner lab.
2) Launch the AWS lab and console, and copy the aws_access_key_id, aws_secret_access_key, and aws_session_token from AWS Details and paste them into your /.aws/credentials credentials file. The region (e.g., us-east-1 for North Virginia) and output format (e.g., json) are stored in the 'config' file. 
3) Additionally, download the labsuser.pem file under SSH Key to authenticate later before accessing your EC2 instances via your console.
4) Once your lab is up and running, visit your AWS Management panel, search for EC2 under Services, and then proceed with the instructions below to establish your two instances.

## Creating EC2 Instances
1) Once on the EC2 page, click the 'Launch Instance' button and input the name of the EC2 instances you wish to launch.
2) Select 'Amazon Linux 2 AMI (HVM) - Kernel 5.10, SSD Volume Type' under AMI.
3) Choose t2.micro as the instance type.
4) Create a new key pair of type RSA and format under Key Pair (login).ppk (if using PuTTY for SSH, this can also be produced later using PuTTYgen from.pem) or.pem (for OpenSSH). Don't forget to keep this key somewhere secure. (For example, EC2_Key.ppk or.pem, which will be transformed later)
5) Under Network settings, make the following changes to create security groups.<br>
    -> SSH with Source type My IP<br>
    -> HTTP with Source type My IP<br>
    -> HTTPS with Source type My IP<br>
6) You may just leave the options under Configure Storage and Advanced Details alone.
7) Select Number of Instances as 2 on the right side, under the summary, to launch two EC2 instances concurrently.
8) To view your instances, click Launch Instance.

![image](https://github.com/RisheekK/AWS-Object-and-Text-Recognition-Pipeline/assets/86208506/d3cc415a-95f5-4517-8dda-f2924b71312f)<br>

## Configuring JAVA Applications and Maven Packages
1) After we've set up the instances, we'll need to write the source code for two separate apps. The first is to recognize things, and the second is to recognize words. Read the project description for more information.
2) Once you've finished writing the source code for both programs (ObjectRecognition.java and TextRecognition.java), we'll need to bundle them into executable JAR files.
3) We will now setup and create the AWS SDK for the application using Apache Maven. If you do not have Maven installed, you may download the binaries and extract the files from http://maven.apache.org/. To access maven's mvn command, add the path to the bin folder inside the system variable PATH. 
4) To begin creating a Maven package, open the terminal and enter the following command:

![image](https://github.com/RisheekK/AWS-Object-and-Text-Recognition-Pipeline/assets/86208506/507993e0-f22b-4f4e-8336-582a53ff7669)<br>

5) at order to configure and utilize AWS SDK, we must define dependencies at the root of your project, in the pom.xml file.<br>
   As demonstrated below, you may add dependencies by using the dependency> tag beneath dependencies>.<br>

![image](https://github.com/RisheekK/AWS-Object-and-Text-Recognition-Pipeline/assets/86208506/feb7a94e-ebe5-47cc-874f-3efb4b6ca0cf)<br>

6) After we've setup the dependencies, we'll produce the JAR file.
7) Use the commands'mvn clean package' and 'mvm clean install' to package the program.
8) Repeat the preceding procedures for both applications. 
9) We can now test our apps by cd-ing into the target folder containing the JAR file and typing 'java -jar YourApplication.jar'. Version numbers are sometimes assigned to.jar files. You can specify whether or not to include the name in the pom.xml build> section and re-package.
10) If you encounter import/package issues while running the program, in my instance, we may utilize automation tools such as Apache Maven, which includes plugins. This generates a new jar with all requirements. The structure may be seen in the screenshot below. <br>

![image](https://github.com/RisheekK/AWS-Object-and-Text-Recognition-Pipeline/assets/86208506/ed18ba55-1e95-4f14-b974-86263728e041)<br>

11) Do not forget to configure the AWS credentials in your local terminal, in order to use AWS services in your application, using the access_key, secret_access_key, and session_token that you copied and pasted into your /.aws/credentials file.
12) If everything goes smoothly, you may move on to the following stage, where you'll upload these JAR files to your EC2 instances. 

## SSH Access to EC2 Instances (PuTTY for Windows)
1) Check to see whether you have PuTTY installed; if not, go to https://www.putty.org/ to get it. To use the 'pscp' command, we'll require PuTTY.
2) Place the ObjectRecognition.jar and TextRecognition.jar files on your EC2_A and EC2_B instances, respectively.
3) Open a terminal (cmd) and type the following command to upload the JAR file to your EC2 instance.<br>
<br>pscp -i C:\Users\rishe\Desktop\NJIT-courses\Cloud\Projects\AWS-Object-and-Text-Recognition-Pipeline\EC2_key.ppk AWS-Object-Recognition-1.0-SNAPSHOT-jar-with-dependencies.jar ec2-user@3.90.208.192:~<br>
This will upload and save the JAR file to the EC2 instance's root directory. Repeat the process for the other application.

4) We will now SSH into the EC2 instances one at a time in separate windows. Follow the procedures below to SSH into a single instance, such as EC2_A.
5) Launch Putty and input the public IP address of your EC2 instance in the 'Session' category.
6) Browse and pick the PPK Private key file (EC2_Key.ppk) under the "Connection">"SSH">"Auth">"Credentials" section.
7) Click Open to begin the session.
8) The user will now be asked to sign in. "ec2-user" is the default username for Amazon Linux and Amazon AMI instances.<br>

![image](https://github.com/RisheekK/AWS-Object-and-Text-Recognition-Pipeline/assets/86208506/5cc04918-8a20-4acb-a310-5231f8e1fdbc)<br>
9) Repeat steps 5–8 for the second Instance.
10) After connecting to EC2 instances using PuTTY on Windows, use the following instructions in order to install essential software, including Java, AWS CLI, and the AWS SDK for Java.<br>
    -> 'sudo yum update'<br>
    -> 'sudo yum install java-1.8.0-openjdk'<br>
    -> 'sudo yum install python-pip'<br>
    -> 'pip install awscli'<br>
    -> 'aws configure' to configure the AWS credentials and access the resources.<br>

## Running the Instances with the application
1) Finally, once you've configured and installed everything on both instances, you may find your JAR file in the root directory of your instances.
2) To execute the program, use 'java -jar yourjarfile.jar'.
3) Run both programs in parallel in two different windows.

## ObjectRecognition on Instance-A
1) This application receives photos from a previously constructed S3 bucket:- https://njit-cs-643.s3.us-east-1.amazonaws.com/njit-cs-643.s3.us-east-1.amazonaws
2) Images that satisfy the object "Car" and have a confidence of greater than 90 are added to the application's SQS queue.
3) We can see that 6 photos met the requirement and were added to the queue. <br>

![image](https://github.com/RisheekK/AWS-Object-and-Text-Recognition-Pipeline/assets/86208506/11f72f19-b433-4cef-a05b-0b627f970d53)<br>

4) The queue is also generated in SQS under Services in the AWS Lab<br>

![image](https://github.com/RisheekK/AWS-Object-and-Text-Recognition-Pipeline/assets/86208506/7cbeac9b-86ac-4ebe-a021-11559032a4c8)<br>

5) The messages can also be seen through polling.  Six photos are delivered, and another is "-1" to indicate that the program has completed image processing. (for the purpose of termination)<br>
![image](https://github.com/RisheekK/AWS-Object-and-Text-Recognition-Pipeline/assets/86208506/d786cf2e-7639-4319-9dd2-bbe200451294)<br>

## TextRecognition on Instance-B
1) As photos are submitted to the SQS queue from Instance-A (Object recognition), they are retrieved and processed for text.
2) The output of the photos' words is saved in "filename.txt" in the same directory, which may be viewed to see the final result.
   The processes and final results are depicted below.<br>

![image](https://github.com/RisheekK/AWS-Object-and-Text-Recognition-Pipeline/assets/86208506/35e44f4a-440f-4f18-935d-e7998d1da1fd)<br>





