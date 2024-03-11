# Deploy React App to AWS EC2 using GitHub Actions
`https://www.youtube.com/watch?v=PH9gvPW5B5I&t=134s`

A tutorial to show how t0 create a typical CICD workflow with a react application using:
- Github actions
- Docker
- AWS EC2

# Create Ec2 instance and install Docker on it

1. Create a new EC2 instance on AWS. for this example we will create access keys, but we will be 
	connecting to our instance within the AWS console in the browser

2. we now need to install docker on our Ec2 instance using the following commands
	- sudo apt-get update
	- sudo apt-get install docker.io -y
	- sudo systemctl start docker
	- sudo docker run hello-world
	- sudo chmod 666 /var/run/docker.sock
	- sudo systemctl enable docker


3. check docker installed with below
	- docker --version

# Upload add to Github and create Dockerfile

1. in our app in vscode ensure app is working with `npm start`
2. now run: `npm run build`

3. create a `Dockerfile` in root of app and add code

//////////////////////////////////////////////////////////////////////////////////////////////////////

# Dockerfile notes:

Step 1: build the react app

`FROM node:alpine3.18 as build`: 
This line is getting the node.js based image node:alpine3.18 from DockerHub and using it throughout this 
Dockerfile. The `as build` part names this build stage `build`.

`WORKDIR /app`: 
Sets the working directory INSIDE the docker container to `/app`. All following instructions in our Dockerfile 
will be run from this directory.

`COPY package.json .`: 
Copies the package.json file from the local machine to the new docker image. The . means the current directory 
in the image which is `/app` as defined by `WORKDIR /app`.

`RUN npm install`: Runs the command `npm install` in our docker container. This command installs all our 
dependencies defined in the `package.json.` file.

`COPY . .`: 
Copies everything from our current directory on our local machine to the current directory in the Docker
image (which is `/app`).

`RUN npm run build`: 
This command will run the build script specified in our `package.json` file using npm in the Docker 
container. This script is generally used to transpile or compile the source code to a version that can 
be run on Node.js.

Step 2: Server with Nginx

`FROM nginx:1.23-alpine`: 
This line is getting the nginx based image ﻿`nginx:1.23-alpine` from DockerHub and using it throughout this 
stage of the Dockerfile. This is the base image for running our built React application.

`WORKDIR /usr/share/nginx/html`: 
Sets the working directory inside the docker container to `/usr/share/nginx/html`. This is the directory where 
nginx, by default, looks for content to serve.

`RUN rm -rf *`: 
Deletes all existing files in the current working directory `(/usr/share/nginx/html)` in the Docker container.
This operation is intended to keep the directory clean and prepared for the new content that's expected 
to be copied.

`.	COPY --from=build /app/build .`: 
Copies the built react application files from the build stage (as defined by `FROM node:alpine3.18 as build`), 
particularly from the directory `/app/build`, into the current directory in this stage, which 
is `/usr/share/nginx/html`

`EXPOSE 80`: This instructs Docker that the container listens on the network port at runtime. Here, it's 
exposing port `80`, the default port for HTTP.

`ENTRYPOINT [ "nginx", "-g", "daemon off;" ]`: 
Defines the default command that will be executed when the container starts. In this case, it is running nginx 
and with the `-g "daemon off;`" option, it specifies that nginx should run in the foreground (not as a daemon).
This is important when running it inside Docker because Docker containers exit when the main process 
runs in the background.


//////////////////////////////////////////////////////////////////////////////////////////////////////

# Create repo for our app at Docker hub

1. go to `docker hub` and create a repo for our app named reactjs-app which will be the repo for the 
	docker image of our app

# Create Secrets in Github repo and cicd.ymal in app

1. go to our repo in `github` > `settings` > `secrets and variable`s and create 2 secrets for Docker useranme
	and password entitled: `DOCKER_USERNAME` &  `DOCKER_PASSWORD`

2. in our repo create `.github` folder > `workflow` and create `cicd.yml` file and add code 

# Create runners in Github and add to Ec2 instance

10. Now in our github repo we will create our ec2 runner. Go to `actions` > `runners` and click the 
	`New self-hosted runner` button to show the `Runners > Download` section

11. Choose runner image as `linux`, select `x64` as Architecture and copy the first line in the `Download` section
	`mkdir actions-runner && cd actions-runner` and paste into our Ec2 instance browser terminal to create a
	a new directory

12. next copy the curl command  from the Github `Runners > Download` section and pase into the Ec2 instance browser 
	terminal which will download the latest runner package.

13. next copy the Extract command from `Runners > Download` section and paste into the Ec2 instance
	 browser terminal

# Create the runners in Ec2 instance

14. next copy the `./config.sh...` line from the  `Runners > Configure` section and choose the following:
	- `default`: presse Enter
	- `Enter the name of runner`: aws-ec2 (from the cicd.yml)
	- `Enter any addtional lables...`: aws-ec2 (from the cicd.yml)

You should now see the messages below: 
	√ Runner successfully added	
	√ Runner connection is good

for the `Enter name of work folder` prompt press Enter to accept the default`

15. now copy the `./run.sh` command from the Github `Runners > Configure` section and past into the
	Ec2 instance browser terminal

	you should hopefully see the `√ Connected to GitHub` message

16. in Github click `Actions` and then `Runners` to see the newly created Runner

# Commit our changes and deploy to our instance

1. in teminal in vscode run `git status`, then git `add .` then `git commmit -m "deploy to ec2"`
	then `git push` OK
