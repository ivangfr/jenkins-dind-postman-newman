# postman-newman-jenkins

## Goal

The goal of this project is to implement an **Automation Testing** to test a REST API. We will use [`Postman`](https://www.getpostman.com), [`Newman`](https://github.com/postmanlabs/newman) (the command line Collection Runner for Postman) and [`Jenkins`](https://jenkins.io). The REST API to be tested will be [`ReqRes`](https://reqres.in), a fake online REST API. 

#### Notes

A new image `docker.mycompany.com/postman-newman-jenkins:2.126` is build from the `Jenkins` base image `jenkins/jenkins:2.126`. _"We need to give the jenkins user sudo privileges in order to be able to run Docker commands inside the container. Alternatively we could have added the jenkins user to the Docker group, which avoids the need to prefix all Docker commands with ‘sudo’, but is non-portable due to the changing gid of the group"_[1]

## Start environment

#### Test Postman Collection in Host Machine

- Open a terminal and go to `/postman-newman-jenkins` root folder

- To run the `Postman` collection present in `\postman` folder without `Jenkins`, execute the following command. It will start `Newman` docker container
```
docker run -t --rm --name newman -v $PWD/postman:/etc/newman \
postman/newman_ubuntu1404:3.9.4 run ReqRes.postman_collection.json -g ReqRes.postman_globals.json
```

#### Docker Compose

- Export to `DOCKER_PATH` environment variable the docker path in host machine
```
export DOCKER_PATH=$(which docker)
```

- Go to `/postman-newman-jenkins/dev` folder and run
```
docker-compose up -d
```
> To stop and remove containers, networks, images, and volumes type:
> ```
> docker-compose down -v
> ```

- Wait a little bit so that `jenkins` container is `Up`.

- To check their status run
```
docker-compose ps
```

#### Jenkins

- Get the Jenkins installation password
```
docker logs jenkins
```

- Access Jenkins: http://localhost:9090

- Inform the installation password

- Select the default installation. It will take a few minutes to complete.

- Create an admin account and proceed normally with the setup.

- On the main Jenkins interface, click on `New item` on the menu.

- Enter a name `user-service-automation-testing`

- Select `Freestyle project`

- Click `OK`

- On the next screen, go to `Build` section

- Click on `Add build step` and select `Execute shell`

- Fill the `Command` field with the command bellow
```
sudo docker run --rm postman/newman_ubuntu1404:3.9.4 \
run "https://raw.githubusercontent.com/ivangfr/postman-newman-jenkins/master/postman/ReqRes.postman_collection.json" \
-g "https://raw.githubusercontent.com/ivangfr/postman-newman-jenkins/master/postman/ReqRes.postman_globals.json" \
--no-color --disable-unicode
```

#### Sources

[1] Running Docker in Jenkins (in Docker): https://container-solutions.com/running-docker-in-jenkins-in-docker