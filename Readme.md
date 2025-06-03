sudo docker run -d --name jenkins \
  -p 8080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  liatrio/jenkins-alpine



https://www.liatrio.com/resources/blog/building-with-docker-using-jenkins-pipelines

docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

