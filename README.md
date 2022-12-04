# esiea-4th-vcc-project

Project d'étude 4è année - ESIEA-INTECH Virtualisation, Conteneurisation et Cloud

![Udagram Architecture](udagram_architecture.png?raw=true "Udagram Architecture")

# Udagram, an Instagram like application

Udagram is a simple application that allows users to register and log into a web client, post photos to the feed, and process photos using an image filtering microservice.

![Udagram Application](udagram.png?raw=true "Udagram Application")

The project is split into two parts:

1. Frontend - Angular web application built with Ionic Framework
2. Backend RESTful APIs - Node-Express application

## Getting Started

> _tip_: it's recommended that you start with getting the backend API running since the frontend web application depends on the API.

### Prerequisite

1. The project depends on the Node Package Manager (NPM). You will need to download and install Node from [https://nodejs.com/en/download](https://nodejs.org/en/download/). This will allow you to be able to run `npm` commands.
2. Environment variables will need to be set. These environment variables include database connection details that should not be hard-coded into the application code.

#### Environment Script

A file named `set_env.sh` has been prepared as an optional tool to help you configure these variables on your local development environment.

I do _not_ want your credentials to be stored in git. After pulling this `starter` project, run the following command to tell git to stop tracking the script in git but keep it stored locally. This way, you can use the script for your convenience and reduce risk of exposing your credentials.
`git rm --cached set_env.sh`

Afterwards, you can prevent the file from being included in your solution by adding the file to our `.gitignore` file.

### 1. Database

Create a PostgreSQL database locally using docker postgres image from dockerhub. The database is used to store the application's metadata.

- You will need to use password authentication for this project. This means that a username and password is needed to authenticate and access the database.
- The port number will need to be set as `5432`. This is the typical port that is used by PostgreSQL so it is usually set to this port by default.

Once your database is set up, set the config values for environment variables prefixed with `POSTGRES_` in `set_env.sh`.

- If you set up a local database, your `POSTGRES_HOST` is most likely `localhost`

### 2. S3

Create an AWS S3 bucket. The S3 bucket is used to store images that are displayed in Udagram.
Follow [this guide](https://youtu.be/5aHsovI2DEk) on how to create an S3 bucket and configure an IAM user for this project

Set the config values for environment variables in `set_env.sh`.

### 3. Backend API

Launch the backend APIs locally. The APIs are the application's interface to S3 and the database.

- To download all the package dependencies, run the command from the directory `udagram-api-feed/` or `udagram-api-user/`:
  ```bash
  npm install .
  ```
- To run the application locally, run:
  ```bash
  npm run dev
  ```
- You can visit `http://localhost:8080/api/v0/feed` in your web browser to verify that the application is running. You should see a JSON payload. Feel free to play around with Postman to test the API's.

### 4. Frontend App

Launch the frontend app locally.

- To download all the package dependencies, run the command from the directory `udagram-frontend/`:
  ```bash
  npm install .
  ```
- Install Ionic Framework's Command Line tools for us to build and run the application:
  ```bash
  npm install -g ionic
  ```
- Prepare your application by compiling them into static files.
  ```bash
  ionic build
  ```
- Run the application locally using files created from the `ionic build` command.
  ```bash
  ionic serve
  ```
- You can visit `http://localhost:8100` in your web browser to verify that the application is running. You should see a web interface.

## Tips

1. The `.dockerignore` file is included for your convenience to not copy `node_modules`. Copying this over into a Docker container might cause issues if your local environment is a different operating system than the Docker image (ex. Windows or MacOS vs. Linux).
2. `set_env.sh` is really for your backend application. Frontend applications have a different notion of how to store configurations. Configurations for the application endpoints can be configured inside of the `environments/environment.*ts` files.
3. In `set_env.sh`, environment variables are set with `export $VAR=value`. Setting it this way is not permanent; every time you open a new terminal, you will have to run `set_env.sh` to reconfigure your environment variables. To verify if your environment variable is set, you can check the variable with a command like `echo $POSTGRES_USERNAME`.
