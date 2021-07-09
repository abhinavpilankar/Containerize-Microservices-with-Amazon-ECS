# Containerize-Microservices-with-Amazon-ECS

Amazon Elastic Container Service (Amazon ECS) is a highly scalable, fast container management service that makes it easy to orchestrate and provisioning of Docker containers on a cluster.
ECS Cluster: is a group of EC2 instances which has ecs-agent software available on it and.
Task Definition: is a blue print of your application that describes how a docker container should launch
Service: Defines long running tasks of the same Task Definition. This can be one or many running containers all using the same Task Definition.
Task: is a running container as per the template defined in the Task Definition
Let’s understand the step-by-step process to containerize a microservice with Amazon ECS
Step1: Create a microservice
Step2: Create Dockerfile
Step3: Create image & run microservice on Docker
Step4: Push Docker image to ECR
Step5: Create ECS cluster
Step6: Create Task Definition & add container information
Step7: Create Service to run the Task definition

Step1: **Create a microservice**
Create microservice with programming language of your choice. Java, Nodejs, Python etc.
Here we are creating a sample python based simple ‘Hello World’ microservice app. Lets write its code in index.py using flask which is a small HTTP server for python apps
index.py
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello World!"
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=int("5000"), debug=True)
    
Step2:** Create Dockerfile**
Create a text file called Dockerfile in your application’s root and paste in the following code.
Dockerfile
FROM python:alpine3.7
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
EXPOSE 5000
ENTRYPOINT ["python","./index.py"]
Where FROM directive is to tell Docker that which base image is to take from Docker Hub.
COPY directive moves the application into the container image
WORKDIR sets the working directory
RUN directive is calling PyPi(pip) to install dependencies available in file: “requirements.txt”
EXPOSE directive is to expose a port to be used by flask
ENTRYPOINT command is to execute the actual application scrtpt
requirements.txt
flask
Check if Docker is available on machine
docker --version
Steps to install Docker on Linux ec2 (if not already available)
install docker on ec2:
sudo yum update -y
sudo yum install -y docker
sudo service docker start
sudo usermod -a -G docker ec2-user
exit
==> login again
docker info
docker --version
So by now we have created index.py, requirements.txt, Dockerfile. Now lets create the Docker image for our application and run the same.

Step3: **Create image & run microservice on Docker**
docker build -t helloworld .
docker run --name python-app -p 5000:5000 helloworld
Go to the Security Group of your ec2 instance and Open Custom TCP port 5000 by editing inbound rules
Check the working Application at ec2 public DNS, followed by port e.g.
ec2 url: http://ec2-52-90-90-238.compute-1.amazonaws.com:5000/
use ctrl+c to exit.

Step4: **Push Docker image to ECR**
Check, is awscli is available at ec2 by: aws s3 ls
if not, install and configure it as below
pip install awscli
aws configure
AWSAccessKeyId=<>
AWSSecretKey=<>
Now, Create repository named ‘helloworld’ from ECR dashboard
Follow ECR commands to push the image in ECR
follow ECR commands:
1.(aws ecr get-login --no-include-email --region us-east-1)
2. skip image built as we have already build the image in previous steps
3. docker tag <image-name>:latest 16xxxxxxxxxx.dkr.ecr.us-east-1.amazonaws.com/helloworld:latest
4. docker push 16xxxxxxxxxx.dkr.ecr.us-east-1.amazonaws.com/helloworld:latest
Note the repository url. In this example, repo url is:
16xxxxxxxxxx.dkr.ecr.us-east-1.amazonaws.com/helloworld:latest
  
 Step5: ****Create ECS cluster****
An Amazon ECS cluster is a logical grouping of tasks or services. If you are running tasks or services that use the EC2 launch type, a cluster is also a grouping of container instances. To create one, Go to the ECS dashboard and Create ECS Cluster named ‘hellocluster’: (linux+Networking). Consider the following configurations while creating and refer snapshots below:
t2-small, 512 MB, choose keypair, choose VPC, choose SG (as of EC2)


Step6: **Create Task Definition & add container information**
Task definition is to be created to tell your ECS cluster that what task cluster is going to run. It is a blue print of your application. Click on Create New Task Definition named ‘hellotask’ and refer snapshot below

Now Click on Add Container button under Container definitions:
Assign Memory Hard limit as 512 and provide port mappings as 5000:5000 as Host port and Container port respectively.

Now, your Task definition is created and ‘Active’

Now, run your application via inline EC2 url:

