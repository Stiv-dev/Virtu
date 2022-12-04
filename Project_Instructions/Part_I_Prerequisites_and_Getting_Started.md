# Overview - Udagram Application

**Udagram** - an Instagram 'like' application, allows users to register and log into a web client, and post photos to a feed.

This section introduces the project followed by instructions on how to set up your local environment remote dependencies to be able to configure and run the starter project.

## Components

At a high level, the project has 2 main components:

1. Frontend Web App - Angular web application built with Ionic Framework
2. Two Backend RESTful APIs - Node-Express application

## Project Goal

In this project you will:

- Set up each microservice to be run in its own Docker container
- Deploy udagram to the Kubernetes cluster

# Local Prerequisites

You should have the following tools installed in your local machine:

- Git
- Node.js
- PostgreSQL
- Ionic CLI
- Docker
- AWS CLI

## Git

Git is used to interface with GitHub.

> Windows users: Once you download and install Git for Windows, you can execute all the bash, ssh, git commands in the Git bash terminal. On the other hand, Windows users using Windows Subsystem for Linux (WSL) can follow all steps as if they are Linux users.

### Instructions

Install [Git](https://git-scm.com/downloads) for your corresponding operating system.

## Node.js

### Instructions

Install Node.js using [these instructions](https://nodejs.org/en/download/). I recommend a version between 14 and 16

This installer will install Node.js as well as NPM on your system. Node.js is used to run JavaScript-based applications and NPM is a package manager used to handle dependencies.

### Verify Installation

```bash
node -v
```

```bash
npm -v
```

## PostgreSQL

Using PostgreSQL involves a server and a client. The server hosts the database while the client interfaces with it to execute queries.

- Use [the official postgres docker image](https://hub.docker.com/_/postgres) from dockerhub
- Use [PostBird](https://github.com/Paxa/postbird) as client

### Instructions

## Ionic CLI

Ionic Framework is used to make cross-platform applications using JavaScript. It is used to help build and run Udagram.

Use [these instructions](https://ionicframework.com/docs/installation/cli) to install Ionic Framework with `npm`.

#### Verify Installation

```bash
# v6.0 or higher
ionic --version
```

## Docker

Docker is needed to build and run containerized applications.

### Instructions

Follow the instructions for [Docker Desktop](https://docs.docker.com/desktop/#download-and-install) to install Docker.

## AWS CLI

You will be using AWS CLI to interface programmatically with AWS.

### Instructions

Follow [these instructions](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) to set up AWS CLI.

After it's installed, you will need to configure an AWS access profile locally so that our local environment knows how to access your AWS account:

1. Create an IAM user with admin privileges on the AWS web console. Copy its Access Key.
2. Configure the access profile locally using your Access Key:

```bash
aws configure
```

### Verify Installation

```bash
# aws-cli/2.0.0 or greater
aws --version
```

A Youtube video is available [here](https://youtu.be/5aHsovI2DEk)

# Project Prerequisites

To run this project, you are expected to have:

1. An S3 bucket
2. A PostgreSQL database

## S3 Bucket

The project uses an AWS S3 bucket to store image files.

### Instructions

1. Navigate to S3 from the AWS console.
2. Create a public S3 bucket with default configurations (eg. no versioning, disable encryption).
3. In your newly-created S3 bucket, go to the **Permissions** tab and add an additional bucket policy to enable access for other AWS services (ie. Kubernetes).

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "Stmt1625306057759",
         "Principal": "*",
         "Action": "s3:*",
         "Effect": "Allow",
         "Resource": [
           "arn:aws:s3:::[bucket_name]",
           "arn:aws:s3:::[bucket_name]/*"
         ]
       }
     ]
   }
   ```

   > Add the CORS configuration to allow the application running outside of AWS to interact with your bucket. You can use the following configuration:

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["POST", "GET", "PUT", "DELETE", "HEAD"],
    "AllowedOrigins": ["*"],
    "ExposeHeaders": []
  }
]
```

> Once the policies above are set and you are no longer testing locally, you can disable public access to your bucket.

## PostgreSQL Database

### Verify Connection

Test the connection from your local PostgreSQL client.
Assuming the endpoint is: `127.0.0.1` `localhost` or , you can run:

```bash
docker run --name [your-container-naime] -e POSTGRES_PASSWORD=[your-database-password] -e POSTGRES_DB=[your-database-name] -e POSTGRES_USER=[your-database-name] -p 5432:5432  -d postgres

Only if you want to test using terminal
psql -h localhost -U [your-username] [your-database-name] -p
# Provide the database password when prompted
```

- for commande line users only:
  If your connection is succesful, your terminal should print ` "postgres=>"`.

You can play around with some `psql` commands found [here](https://www.postgresql.org/docs/13/app-psql.html).

Afterwards, you can enter `\q` to quit.

# Project Configuration

Once the local and remote prerequisites are set up, you will need to configure the application so that they can connect and utilize them.

## Fork and Clone the Project

If you have not already done so, you will need to fork and clone the project so that you have your own copy to work with.

```bash
git clone https://github.com/<YOUR_GITHUB_USERNAME>/esiea-4th-vcc-project.git

cd esiea-4th-vcc-project/
```

## Configuration Values

The application will need to connect to the AWS PostgreSQL database and S3 bucket that you have created.

You do **not** want to hard-code the configuration details into the application code. The code should not contain sensitive information (ie. username and password).

For this reason, you will follow a common pattern to store our credentials inside environment variables. You'll explain how to set these values in Mac/Linux environments and Windows environments followed by an example.

### Set Environment Variables in Mac/Linux

#### Instructions

1. Use the `set_env.sh` file present in the `project/` directory to configure these values on your local machine. This is a file that has been set up for your convenience to manage your environment.
2. Prevent this file from being tracked in `git` so that your credentials don't become stored remotely:

   ```bash
   # Stop git from tracking the set_env.sh file
   git rm --cached set_env.sh

   # Prevent git from tracking the set_env.sh file
   echo *set_env.sh >> .gitignore
   ```

3. Running the command `source set_env.sh` will set your environment variables.
   > Note: The method above will set the environment variables temporarily. Every time you open a new terminal, you will have to run `source set_env.sh` to reconfigure your environment variables

#### Verify Configurations

1. Set the configuration values as environment variables:

```bash
source set_env.sh
```

2. Verify that environment variables were set by testing one of the expected values:

```bash
echo $POSTGRES_USERNAME
```

### Set Environment Variables in Windows

Set all the environment variables as shown in the `set_env.sh` file either using the **Advanced System Settings** or d GitBash/WSL terminal.

Below is an example. Make sure that you replace the values with ones that are applicable to the resources that you created in AWS.

```bash
setx POSTGRES_USERNAME postgres
setx POSTGRES_PASSWORD abcd1234
setx POSTGRES_HOST mypostgres-database-1.c5szli4s4qq9.us-east-1.rds.amazonaws.com
setx POSTGRES_DB postgres
setx AWS_BUCKET test-udagram-dev-wc # replace with your bucket name
setx AWS_REGION us-east-1
setx AWS_PROFILE default
setx JWT_SECRET hello
setx URL http://localhost:8100
```
