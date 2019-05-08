A Drupal8 build for lib.lsu.edu

## Git

### Windows10:

Install git from:  https://git-scm.com/download/win

  CRUCIAL -- "Checkout as-is, commit as-is".  Otherwise Windows will use \n\r line endings which will break the code in the containers and result in a "/usr/bin/env: 'bash\r': No such file or directory" error.

  If you already have this error post-install, reset the git lineendings config to "\n", with:
  
    git config --system core.autocrlf false
    git config --system core.whitespace cr-at-eol
    

  Also, I select "use git only from git bash" and then open git bash as my command prompt.


  Then in git bash, 

  from whatever directory you want the repo folder pulled to:

  `git clone --recursive https://github.com/lsulibraries/gnatty`
  
  `cd gnatty`

## Install docker-compose version 1.22 or greater

### linux:

```
sudo apt remove docker docker-engine docker.io
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install docker-ce -y
sudo usermod -aG docker $USER
```
```
Log out your computer & log back in.
```
```
docker run hello-world
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### Windows10:

    - install docker app from https://docs.docker.com/docker-for-windows/install/   
        - you may need to create a free docker account
        - "Use linux containers"
        - Hyper-V and Containers must be enabled in your Windows install.  Docker will attempt to enable them.
        - you may need to restart your computer.
    - go to docker toolbar icon, right-click, 'Settings'. in Settings panel, do 'Shared Drive', enable drive sharing on the C drive (or whichever drive the repo was cloned to).  (This setting allows docker access to "Drupal8Scaffoldings" folder, but not any other folder on C.) 
    - open PowerShell or gitbash (but not PowerShell ISE)
        - try `docker -v` and `docker-compose -v` to see whether they both installed
        - try `docker run hello-world` (this will pull an image from the repo & run it)


### osX:

    - to be added

## To run the containers

#### In the file ".env", change MYSQL_USER, MYSQL_PASSWORD, SSH_PASSWORD, and MYSQL_ROOT_PASSWORD to something.

Change the passwords at least.  Otherwise, Jimmy Random in Accounting can navigate on his computer to http://{ your ip address }:5001 and access the whole drupal database.  Or ssh into the drupal8 instance.  Changing the passwords in ".env" will control all the passwords on your instance.  Production will get secret values.

```
docker-compose up --build
```

### make www-data:www-data owner of drupal_sync folder

only needed on first run

```
docker-compose exec webapp chown -R www-data:www-data /drupal_sync
```


See the app at localhost:5000 and select the "Use existing configuration" option

See a database viewer at localhost:5001

## To stop the containers

```
ctrl-C
-or-
docker-compose stop 
```

Note:  On each webapp build, the MySQL container will need a few seconds to prepare.  You'll see when it's ready, when localhost:5001 gives a page.  Until the database is built, the server will give bad responses.  Wait a few seconds on first build.

You'll soon see a base install Drupal at localhost:5000.
And you'll see the drupal code inside webapp container mirrored on your computer at ./drupal_app/
When you do a drupal sync from webapp container, those files will be mirrored at ./drupal_sync/

## To wipe the drupal project and start clean:

You may delete the repo folder, then git clone it again.
Or you may:

```
docker-compose down
docker volume prune
docker system prune
docker-compose up --build
```

## To enter a bash prompt on the drupal container:

`docker-compose exec webapp bash`

Note you can substitute any container's nickname.  You can also substitute any command.  

Example:  If you want to run "ls /etc" on the nginx container, `docker-compose exec nginx ls /etc`

## To connect an IDE, Putty, Sublime, etc to the drupal_app folder:

Configure your program to sftp connect to server:localhost, port:5022, user:root, password:{whatever SSH_PASSWORD you set in .env}
If your code changes aren't applying to the running drupal, you may need to `docker-compose restart nginx` or `docker-compose restart webapp`

## To export config changes to drupal_sync/:

`docker-compose exec webapp drush config-export`

## To import config settings from drupal_sync/:

`docker-composer exec webapp drush config-import`

# some docker commands 

    ps    {show containers}

        -a      {including stopped ones}

    network     

        ls      {list}

        prune   {remove inactive ones}

    volume

        ls      {list}

        prune   {remove inactive ones}

                - this will destroy the container's persistent data


docker-compose

    up      {start the containers}

        -d  {in detached mode}
        --build {make the docker container if it doesn't exist}

    down    {kill & remove the containers}

    exec container_name /bin/bash {or other program from inside the container}

    logs container_name     {show logs}
