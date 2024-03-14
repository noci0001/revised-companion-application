# Troubleshooting

## DOCKER & DEVCONTAINERS TROUBLESHOOTING

### If you are having issues with reopening the project in a DevContainer 
_e.g. "Is Docker deamon not running"_
You can try fixing this initial issue by opening VSCode with sudo rights.
To acheive this, close your VSCode window and run this command in the terminal:

```
sudo code --user-data-dir="~/.vscode-root"
```

### If the DevContainer fails to open, 
  
make sure that docker and docker-desktop is installed correctly and running, with the following commands:
```
docker --version
sudo systemctl status docker
docker run hello-world
```

### If the problems persist, you might have a misconfiguration of your Docker.

You might also be in the wrong Docker context, in order to debug, list them:

```
docker context list
```

If you see multiple, try switching to them (try switching to the default one)
Example:
```
docker context use default
```

Once you have switched to a different context, try reopening in DevContainer.


## If nothing works
  
  you can try purging and reinstalling the entire Docker:

```
sudo apt-get purge docker-ce docker-ce-cli containerd.io
sudo rm -rf /etc/docker/ /var/lib/docker/
```

To avoid conflicts during the reinstallation, remove Docker repositories:

```
sudo rm /etc/apt/sources.list.d/docker.list
sudo apt-get update
```

Install Docker CLI:

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl enable docker
```

Now it's time for Docker Desktop:

Download latest Docker Desktop .deb package
Open it in the terminal and run:
```
sudo dpkg -i <docker-desktop>.deb
```

Final check to see if everything was reinstalled correctly:
```
docker --version
sudo systemctl status docker
docker run hello-world
```
