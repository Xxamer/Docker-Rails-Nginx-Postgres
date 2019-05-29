# NGINX + RAILS PRODUCTION + PUMA + DOCKER
## 1. First Steps

In the Rails application we go to the `Gemfile.lock` file and delete in the end of the file the lines of `BUNDLE WITH {Bundler version }.`

![](https://i.imgur.com/iLvqjve.png)

We add Docker files that will create a postgresql container, one from Rails with our production mode application and one from Nginx that will redirect requests to the Rails server through port 80 by default or we can designate one on the `Docker-compose.yml` file.

The default path is:
`Docker-compose.yml` at the root of the Rails project.
A folder named `Docker` with two folders "`web`" and "`app`" inside, app for setting up the Rails application and Web for Nginx.

Once this is done we can configure the files first or move the application to the server and edit it using ftp.

| ![Imgur ](https://i.imgur.com/1Oa1pRF.png ) | ![Imgur ](https://i.imgur.com/L890aHO.png ) ||||
| -------------------------------------------| -------------------------------------------| --- | --- | --- |

## 2. Editing the docker files
### Docker-compose.yml

In the file ‘Docker-compose.yml’ we define the name of the containers and their dependencies, in the Nginx container we define the port by which we will be able to connect externally.

Note: The names of the containers and the port of Nginx must be different in order to have multiple servers in parallel.

![Imgur](https://i.imgur.com/uOVuAMg.png)

### app/dockerfile

In this file we define the Ruby installation with which we created the Rails project, in my case it is version 2.5.1. The Alpine version means that it is a "light" version to reduce the size of the container.

Then we installed the necessary dependencies to start the project of Rails and the Bundler to be able to install the gems.

We create a variable called "RAILS_ROOT" where we will define the route where the Rails project is hosted

Then we define the environment of the application as "production", add the gems to the working directory and run the Bundle to start the puma server.

![Imgur](https://i.imgur.com/dYqqRY3.png)

#### Note: If the Rails application is developed in api mode we should comment the line of ‘RUN Bundle exec Rake assets:precompile’

### web/dockerfile

Here we download the nginx image and install the necessary updates and tools to get it started, then we set the path where Nginx will search for files and save the settings

![Imgur](https://i.imgur.com/24kLVid.png)

### web/nginx.conf

At the beginning of the file we define where the Rails application is, putting the name of the container and the default port is 3000.

Then we define what the server is going to be, in my case it is the server ip but if we have a DNS it would be the name (p.j www.example.com)

Deny requests to files that should not be accessible, such as logs and files with `.rb` extension

For the rest we enable the CORS to access from any external address.

We define a long waiting times in case the server has heavy traffic or has to solve a lot of data

![Imgur](https://i.imgur.com/dFiRS4v.png)

Once this is done, we go to the folder where the Rails project and Docker files are in the server,and there we run "Docker-compose build" to start downloading and creating the images. When they are downloaded we will run "Docker-compose up" to start Docker images and with this now we have connection to the Rails application via nginx.

![Imgur](https://i.imgur.com/oPe7NV6.png)



## Connection of Rails Application with database

By default the image of postgres brings the user "postgres" with password empty so we will leave it so unless we define another user in the Docker-compose, in the "host" we will specify the name of the container

![Imgur](https://i.imgur.com/Un8LrJI.png)



## Errors encountered

In case the Rails application is not available, the Nginx server will send an "502" error 

![Imgur](https://i.imgur.com/5FGmgHC.png)


## Error: Missing ‘secret key base’ for ‘production’ environment
This error occurs when you start an application in Rails in production mode and do not find the "secret key", to fix it you use the command `rails rake secret`, the string you return will be placed in the config/application.Rb file in a field called `config.secret_key_base= string of Rake secret`

![Imgur](https://i.imgur.com/GTUl0uD.png)

![Imgur](https://i.imgur.com/aeDaE5N.png)