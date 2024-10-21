## Docker

### install docker engine in ec2
1. create ec2
2. create key pair
3. create security group
4. inbount rule > source - myIP
5. add inbound rule > ALL TCP, Anywher IP4

documentation: https://docs.docker.com/engine/install/ubuntu/

1. Set up Docker's apt repository.
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
2. Install the Docker packages.
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

3. Verify that the Docker Engine installation is successful by running the hello-world image.
```
sudo docker run hello-world
```

(or)

```
systemctl status docker
```

list all docker images
```
docker images
```
it will get this error:  
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Head "http://%2Fvar%2Frun%2Fdocker.sock/_ping": dial unix /var/run/docker.sock: connect: permission denied\

because by deafult root user can connect to docker deamon. if we want to accesss, we need to user to docker group

documentation :https://docs.docker.com/engine/install/linux-postinstall/

```
sudo usermod -aG docker ubuntu
```
it will add ubuntu user to group

note: logout and login

Verify that the Docker Engine installation is successful by running the hello-world image.
```
sudo docker run hello-world
```

to check the containers
```
docker ps -a
```



