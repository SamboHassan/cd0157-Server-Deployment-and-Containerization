# Deploying a Flask API

This is the project starter repo for the course Server Deployment, Containerization, and Testing.

In this project you will containerize and deploy a Flask API to a Kubernetes cluster using Docker, AWS EKS, CodePipeline, and CodeBuild.

The Flask app that will be used for this project consists of a simple API with three endpoints:

- `GET '/'`: This is a simple health check, which returns the response 'Healthy'.
- `POST '/auth'`: This takes a email and password as json arguments and returns a JWT based on a custom secret.
- `GET '/contents'`: This requires a valid JWT, and returns the un-encrpyted contents of that token.

The app relies on a secret set as the environment variable `JWT_SECRET` to produce a JWT. The built-in Flask server is adequate for local development, but not production, so you will be using the production-ready [Gunicorn](https://gunicorn.org/) server when deploying the app.

## Prerequisites

- Docker Desktop - Installation instructions for all OSes can be found <a href="https://docs.docker.com/install/" target="_blank">here</a>.
- Git: <a href="https://git-scm.com/downloads" target="_blank">Download and install Git</a> for your system.
- Code editor: You can <a href="https://code.visualstudio.com/download" target="_blank">download and install VS code</a> here.
- AWS Account
- Python version between 3.7 and 3.9. Check the current version using:

```bash
#  Mac/Linux/Windows 
python --version
```

You can download a specific release version from <a href="https://www.python.org/downloads/" target="_blank">here</a>.

- Python package manager - PIP 19.x or higher. PIP is already installed in Python 3 >=3.4 downloaded from python.org . However, you can upgrade to a specific version, say 20.2.3, using the command:

```bash
#  Mac/Linux/Windows Check the current version
pip --version
# Mac/Linux
pip install --upgrade pip==20.2.3
# Windows
python -m pip install --upgrade pip==20.2.3
```

* Terminal
  - Mac/Linux users can use the default terminal.
  - Windows users can use either the GitBash terminal or WSL.
- Command line utilities:
  - AWS CLI installed and configured using the `aws configure` command. Another important configuration is the region. Do not use the us-east-1 because the cluster creation may fails mostly in us-east-1. Let's change the default region to:

  ```bash
  aws configure set region us-east-2  
  ```

  Ensure to create all your resources in a single region.
  - EKSCTL installed in your system. Follow the instructions [available here](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html#installing-eksctl) or <a href="https://eksctl.io/introduction/#installation" target="_blank">here</a> to download and install `eksctl` utility.
  - The KUBECTL installed in your system. Installation instructions for kubectl can be found <a href="https://kubernetes.io/docs/tasks/tools/install-kubectl/" target="_blank">here</a>.

## Initial setup

1. Fork the <a href="https://github.com/udacity/cd0157-Server-Deployment-and-Containerization" target="_blank">Server and Deployment Containerization Github repo</a> to your Github account.
1. Locally clone your forked version to begin working on the project.

```bash
git clone https://github.com/SudKul/cd0157-Server-Deployment-and-Containerization.git
cd cd0157-Server-Deployment-and-Containerization/
```

1. These are the files relevant for the current project:

```bash
.
├── Dockerfile 
├── README.md
├── aws-auth-patch.yml #ToDo
├── buildspec.yml      #ToDo
├── ci-cd-codepipeline.cfn.yml #ToDo
├── iam-role-policy.json  #ToDo
├── main.py
├── requirements.txt
├── simple_jwt_api.yml
├── test_main.py  #ToDo
└── trust.json     #ToDo 
```

## Project Steps

Completing the project involves several steps:

1. Write a Dockerfile for a simple Flask API
2. Build and test the container locally
3. Create an EKS cluster
4. Store a secret using AWS Parameter Store
5. Create a CodePipeline pipeline triggered by GitHub checkins
6. Create a CodeBuild stage which will build, test, and deploy your code

For more detail about each of these steps, see the project lesson in your classroom.

## Running the container locally

To run the containerized application using the production-ready [Gunicorn](https://gunicorn.org/) server and test the endpoints locally. Open a terminal and run the following command passing the path to a local file that stores your `JWT_SECRET` and `LOG_LEVEL` environment variables.

 ```bash
docker run --name containerName --env-file=pathToenv_file -p 80:8080 imageName
```

Sample run:

 ```bash
docker run --name my-container --env-file=/home/soft-dev/Desktop/Server-Deploy-project/env_file.txt -p 80:8080 udacity-flask-app
```

The env_file used for deploying to the gunicorn server locally is a simple text file that stores the environment variables in a key-value pairs as shown below:

 ```bash
JWT_SECRET='abc123abc1234'
LOG_LEVEL=DEBUG
```

## To check the endpoints

 ```bash
# Flask server running inside a container
curl --request GET 'http://localhost:80/'

# Calls the endpoint 'localhost:80/auth' with the email/password as the message body. 
# The return JWT token assigned to the environment variable 'TOKEN' 
export TOKEN=`curl --data '{"email":"abc@xyz.com","password":"WindowsPwd"}' --header "Content-Type: application/json" -X POST localhost:80/auth  | jq -r '.token'`

echo $TOKEN

# Decrypt the token and returns its content
curl --request GET 'http://localhost:80/contents' -H "Authorization: Bearer ${TOKEN}" | jq .
```
