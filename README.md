# jenkins-dind-postman-newman

The goal of this project is to implement an **Automation Testing** for a fake online REST API called [`ReqRes`](https://reqres.in). We will use:
- [`Jenkins`](https://jenkins.io), a self-contained, open source automation server which can be used to automate all sorts of tasks related to building, testing, and delivering or deploying software;
- [`Docker-in-Docker (dind)`](https://hub.docker.com/_/docker) to execute `Docker` commands inside `Jenkins`;
- [`Postman API Client`](https://www.postman.com/product/api-client/), for testing API calls;
- [`Newman`](https://github.com/postmanlabs/newman), a command-line collection runner for `Postman`.

## Prerequisites

- [`Postman API Client`](https://www.postman.com/product/api-client/)
- [`Docker`](https://www.docker.com/)
- [`Docker-Compose`](https://docs.docker.com/compose/install/)

## Test Postman Collection in Host Machine

- Open a terminal and make sure you are inside `jenkins-dind-postman-newman` root folder

- Execute the following command. I will run `Newman` docker container using the `Postman` collection present in `postman` folder.
  ```
  docker run -t --rm --name newman -v $PWD/postman:/etc/newman \
    postman/newman:5.3.1-alpine run ReqRes.postman_collection.json --globals ReqRes.postman_globals.json
  ```
  
- In `postman` folder, there are two JSON files that configure some test cases and environment variables used to run them. You can edit them by using `Postman`. 

## Running Jenkins using Docker-in-Docker

- In a terminal, make sure you are inside `jenkins-dind-postman-newman` root folder

- Run the command below
  ```
  docker-compose up -d
  ```
  > **Note 1:** A new image called `jenkins-blueocean:2.345-jdk11` is build from the `Jenkins` base image `jenkins/jenkins:2.345-jdk11`. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`
  
  > **Note 2:** If you prefer run `jenkins-docker` and `jenkins` Docker containers using `docker run` command instead, follow the steps described at [jenkins.io website](https://www.jenkins.io/doc/book/installing/docker/)

- Wait a bit so that `jenkins` container is `Up (healthy)`. To check it run
  ```
  docker-compose ps
  ```

## Configuring Jenkins

- In a terminal, get the `Jenkins` installation password by running the following command
  ```
  docker logs jenkins
  ```

  You should see something similar to
  ```
  *************************************************************
  *************************************************************
  *************************************************************
  
  Jenkins initial setup is required. An admin user has been created and a password generated.
  Please use the following password to proceed to installation:
  
  1b5964b8c7a648e584a994fbb293c05f
  
  This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
  
  *************************************************************
  *************************************************************
  *************************************************************
  ```

- Access `Jenkins` at http://localhost:8080

- Inform the installation password and click `Continue`

- Select `Install suggested plugins` (default installation) and wait for it to complete

- In the screen `Create First Admin User`, create an account for you informing a username, password, etc. Then, click `Save and Continue` button

- Keep the `Jenkins URL` as it is, i.e, `http://localhost:8080/`, and click `Save and Finish` button

- **Jenkins is ready!** Click `Start using Jenkins`

## Configuring Automation Project in Jenkins 

- If you are not in `Jenkins` website, access it at http://localhost:8080

- Provide the username and password that you created while [configuring Jenkins](#configuring-jenkins) 

- In the main `Jenkins` interface, click `New item` menu on the left

- Enter a name, for instance, `rest-api-automation-testing`

- Select `Freestyle project` and click `OK` button

- In the next screen, go to `Build` section. Click `Add build step` and select `Execute shell`

- Set the command below to the `Command` field
  ```
  docker run --rm postman/newman:5.3.1-alpine \
  run "https://raw.githubusercontent.com/ivangfr/postman-newman-jenkins/master/postman/ReqRes.postman_collection.json" \
  --globals "https://raw.githubusercontent.com/ivangfr/postman-newman-jenkins/master/postman/ReqRes.postman_globals.json" \
  --disable-unicode --color off
  ```

- Click `Save` button

- Your Jenkins project is created. You should see something like

  ![rest-api-jenkins-project](images/rest-api-jenkins-project.png)

- In order to run the project, click `Build Now` menu item on the left
  
  > **Note:** The test cases are a bit strict. They require that the response time must be below `1000` ms. So, depending on how fast `ReqRes` online REST API replies to you request, maybe some test cases will fail and, consequently, the build will fail.

- To see the execution results, click the red or green icons that appears inside `Build History` (section on the left) everytime you build the `Jenkins` project. You should get an output like
  ```
  Started by user Admin
  Running as SYSTEM
  Building in workspace /var/jenkins_home/workspace/rest-api-automation-testing
  [rest-api-automation-testing] $ /bin/sh -xe /tmp/jenkins6209473670817509056.sh
  + docker run --rm postman/newman:5.3.1-alpine run https://raw.githubusercontent.com/ivangfr/postman-newman-jenkins/master/postman/ReqRes.postman_collection.json --globals https://raw.githubusercontent.com/ivangfr/postman-newman-jenkins/master/postman/ReqRes.postman_globals.json --disable-unicode --color off
  Unable to find image 'postman/newman:5.3.1-alpine' locally
  5.3.1-alpine: Pulling from postman/newman
  fa637f525e1a: Pulling fs layer
  fa637f525e1a: Download complete
  fa637f525e1a: Pull complete
  Digest: sha256:8c6463863869941ab58a27655a6810fcdc01943cef82cfbda73bd6459e1751fa
  Status: Downloaded newer image for postman/newman:5.3.1-alpine
  newman
  
  ReqRes
  
  Root Get List of Users
    GET https://reqres.in/api/users [200 OK, 1.89kB, 296ms]
    Pass  Status code is 200
    Pass  Content-Type is present
    Pass  Response time is less than 1000ms
    Pass  Page Schema is valid
    Pass  Field values
  
  Root Get Single User
    GET https://reqres.in/api/users/2 [200 OK, 1.18kB, 65ms]
    Pass  Status code is 200
    Pass  Content-Type is present
    Pass  Response time is less than 1000ms
    Pass  User Schema is valid
    Pass  Field values
  
  Root Get Nonexistent User
    GET https://reqres.in/api/users/20 [404 Not Found, 865B, 141ms]
    Pass  Status code is 404
    Pass  Content-Type is present
    Pass  Response time is less than 1000ms
  
  Root Post User
    POST https://reqres.in/api/users [201 Created, 897B, 134ms]
    Pass  Status code is 201
    Pass  Content-Type is present
    Pass  Response time is less than 1000ms
    Pass  User Schema is valid
  
  Root Put User
    PUT https://reqres.in/api/users/2 [200 OK, 930B, 151ms]
    Pass  Status code is 200
    Pass  Content-Type is present
    Pass  Response time is less than 1000ms
    Pass  User Schema is valid
  
  Root Delete User
    DELETE https://reqres.in/api/users/2 [204 No Content, 762B, 126ms]
    Pass  Status code is 204
    Pass  Response time is less than 1000ms
  
  --------------------------------------------------------------------
  |                         |           executed |            failed |
  --------------------------+--------------------+--------------------
  |              iterations |                  1 |                 0 |
  --------------------------+--------------------+--------------------
  |                requests |                  6 |                 0 |
  --------------------------+--------------------+--------------------
  |            test-scripts |                 12 |                 0 |
  --------------------------+--------------------+--------------------
  |      prerequest-scripts |                  6 |                 0 |
  --------------------------+--------------------+--------------------
  |              assertions |                 23 |                 0 |
  --------------------------------------------------------------------
  | total run duration: 1316ms                                       |
  --------------------------------------------------------------------
  | total data received: 1.43kB (approx)                             |
  --------------------------------------------------------------------
  | average response time: 152ms [min: 65ms, max: 296ms, s.d.: 70ms] |
  --------------------------------------------------------------------
  Finished: SUCCESS
  ```

## Shutdown

- In a terminal, make sure you are inside `jenkins-dind-postman-newman` root folder

- To stop and remove docker-compose containers, network and volumes run
  ```
  docker-compose down -v
  ```

## Cleanup

To remove the Docker image create by this project, go to a terminal and, `jenkins-dind-postman-newman` root folder, run the following script
```
./remove-docker-images.sh
```

## How to keep this project updated

Check https://www.jenkins.io/doc/book/installing/docker/
