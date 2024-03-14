# TROUBLESHOOTING

## GENERAL TROUBLESHOOTING

### There was an error creating your Release: tag name can't be blank, tag name is not well-formed, published releases must have a valid tag.
> During the process of Publishing the Release, make sure to add a meaningful tag or you will be blocked by the following error: 
 
### CI MULTIARCH RELEASE WORKFLOW FAILING
> If any of the workflow (CI, Multiarch, Release) fail there is likely a misconfiguration of the permissions in your github repo.
> Firstly, make sure that the repo is private.
> Once you have excluded that problem, you can go to your GitHub settings, tokens and then inspect the token assigned to this repo.
> Retry the run the workflow.
> If it is still not working, the problem is caused by wrongly set permissions in you Github. To fix this, go to your repo Settings > Action > General and if you scroll at the bottom you can check Read and Write permissions.

### If you are getting a 401 Unauthorized error when trying to obtain the image of your application
>You must first navigate to Code -> Package -> Package Settings
>Scroll to the bottom and within the Danger Zone change the visibility if it is set to private to public. 
>kanto-cm create will now correctly create the container from your application.

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

### If nothing works
  
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
