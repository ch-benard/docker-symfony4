# DOCKER-SYMFONY4

Docker environment for a Symfony4 project with Nginx, PHP 7.3 and Postgresql 9.6

## Prerequisites

To use DOCKER-SYMFONY4, you need a recent version of git, docker and docker-compose that works on your computer. There are a lot of tutorial on the internet that can help you installing git, docker and docker-compose. I personally have these tools running on my PC that runs Ubuntu 18.04 and on my laptop with CloudReady, a nice ChromeOS fork.

## How to create a SF4 project from scratch with DOCKER-SYMFONY4 ?

1. Clone the docker-symfony4 repository
*git clone https://github.com/ch-benard/docker-symfony4*

2. Start docker containters

*cd [location where you cloned the project]/docker-symfony4*

*docker-compose up -d*

3. Connect to the sf4-php-fpm container where we will create our symfony4 application skeleton
*docker-compose exec php-fpm bash*

4. Create the skeleton for the symfony project
(inside sf4-php-fpm bash)
*composer create-project symfony/skeleton [our_new_project]*

5. Move the content of the skeleton into the root of the application
When the *composer create-project* command is lauched, we cannot create our new application into the root folder directly because the installer deletes the content of the folder we're cloning into. Since it’s too risky, this option is not allowed. But unless we change the config of the working directory inside the *docker-composer.yml*, we need our new project to be in the root folder. The trick is then to move the content of the newly built skeleton of our application afterward from the [our_new_project] directory to the root directory (/application).

(inside sf4-fphp-fpm bash)
mv /application/[our_new_project]/* /application
mv /application/[our_new_project]*/.* /application

6. Remove the empty [our_new_project] directory
(inside sf4-php-fpm bash)
*rm -Rf /application/[our_new_project]*

7. Install the commonly required components
(inside sf4-php-fpm bash)

*cd /application*
*composer require annotations*
*composer require --dev profiler*
*composer require twig*
*composer require orm*
*composer require form*
*composer require form validator*
*composer require maker-bundle*

Feel free to add the ones you need.

8. Creating some sample code in Symfony 4 project.
Now lets create a controller to test a sample route to make sure everything works.

(inside sf4-php-fpm bash)
*cd /application*
*bin/console make:controller WelcomeController*

Now, type the following URL. The port is the one we set up in the docker-compose.yml – If you check the dictionary of the ngix config, you can see that port 8000 maps the 80, which is the usual webserver port.

http://localhost:8000/welcome

9. Sync de database.
Finally, to sync the database, you need to update the .env file with the variables we set on the postgres image.

.env file that has been generated when requiring the orm package in Symfony4.

DATABASE_URL=pgsql://pgusr:pgpwd@127.0.0.1:5432/pgdb
So if you check the *docker-compose.yml* file, you’ll see the credentials under the pgsql configuration. So change the file to

DATABASE_URL=pgsql://pgusr:pgpwd@pgsql:5432/pgdb

Finally, let’s restart the containers

(from the host)
you need to cd first to where your docker-compose.yml file lives
*docker-compose down*
*docker-compose up -d*
*docker-compose exec php-fpm bash*

Now inside the bash, you should be able to create the schema

(inside php-fpm bash)
*bin/console doc:sch:crea*

You can connect an external database client such as pgadmin or dbeaver.

For the credentials remember to put again the ones we set in the mysql image.

Host: 0.0.0.0
Username: dbusr
Password: dbpwd
Port: 5432

## Docker compose cheatsheet

**Note:** you need to cd first to where your docker-compose.yml file lives.

* Start containers in the background: `docker-compose up -d`
* Start containers on the foreground: `docker-compose up`. You will see a stream of logs for every container running.
* Stop containers: `docker-compose stop`
* Kill containers: `docker-compose kill`
* View container logs: `docker-compose logs`
* Execute command inside of container: `docker-compose exec SERVICE_NAME COMMAND` where `COMMAND` is whatever you want to run. Examples:
      * Shell into the PHP container, `docker-compose exec php-fpm bash`
      * Run symfony console, `docker-compose exec php-fpm bin/console`
      * Open a mysql shell, `docker-compose exec mysql mysql -uroot -pCHOSEN_ROOT_PASSWORD`

## Docker general cheatsheet

**Note:** these are global commands and you can run them from anywhere.

* To clear containers: `docker rm -f $(docker ps -a -q)`
* To clear images: `docker rmi -f $(docker images -a -q)`
* To clear volumes: `docker volume rm $(docker volume ls -q)`
* To clear networks: `docker network rm $(docker network ls | tail -n+2 | awk '{if($2 !~ /bridge|none|host/){ print $1 }}')`

Disclaimer: This project has been generated on phpdocker.io and ispired by [Joey Masip Romeu's work](https://github.com/joeymasip/docker-symfony4)
