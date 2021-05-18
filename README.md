# angular-nginx-docker_ng-conf_2021

## Run an Angular app locally with Docker and Nginx

>Lightning talk presentation given at ng-conf.com's 2021 online conference.

# Ng-Conf 2021 - Presentation Script

## Title page

>Hi! My name is Drew Shirts and I'll be sharing how to run an Angular app locally with Docker and Nginx.

## Tools

>There are a few tools we'll be using for this presentation: Node, NPM, Visual Studio Code, Docker, and Angular CLI.

## Node.js and NPM

>We won't be covering the install of Node and NPM in this tutorial, but you can download both together at [nodejs.org](https://nodejs.org).

## VS Code

>I'll be using [VS Code](https://code.visualstudio.com/download) as the editor of choice for this tutorial.

## Docker Desktop

>Installing [Docker Desktop](https://www.docker.com/products/docker-desktop) will provide you with the CLI and the UI executables.
>Once downloaded, validate the install with:
>
>```zsh
>docker --version
>```

## Angular CLI

>You can install the Angular CLI using this global npm command.
>
>```zsh
>npm install -g @angular/cli
>```

## Angular App

>With these tools in place, let's create an Angular app with the default settings.
>
>```zsh
>ng new angular-nginx-docker --defaults
>```

## Angular App Created

>After creating the default project, it will run npm install for you. Depending on your internet connection, it can take a while.
>
>```zsh
>drewshirts ng-conf-2021 %ng new angular-nginx-docker --defaults
>CREATE angular-nginx-docker/README.md (1027 bytes)
>CREATE angular-nginx-docker/.editorconfig (274 bytes)
>CREATE angular-nginx-docker/.gitignore (631 bytes)
>CREATE angular-nginx-docker/angular.json (3647 bytes)
>CREATE angular-nginx-docker/package.json (1210 bytes)
>CREATE angular-nginx-docker/tsconfig.json (538 bytes)
>CREATE angular-nginx-docker/tslint.json (3185 bytes)
>CREATE angular-nginx-docker/.browserslistrc (703 bytes)
>CREATE angular-nginx-docker/karma.conf.js (1437 bytes)
>CREATE angular-nginx-docker/tsconfig.app.json (287 bytes)
>CREATE angular-nginx-docker/tsconfig.spec.json (333 bytes)
>CREATE angular-nginx-docker/src/favicon.ico (948 bytes)
>CREATE angular-nginx-docker/src/index.html (304 bytes)
>CREATE angular-nginx-docker/src/main.ts (372 bytes)
>CREATE angular-nginx-docker/src/polyfills.ts (2830 bytes)
>CREATE angular-nginx-docker/src/styles.css (80 bytes)
>CREATE angular-nginx-docker/src/test.ts (753 bytes)
>CREATE angular-nginx-docker/src/assets/.gitkeep (0 bytes)
>CREATE angular-nginx-docker/src/environments/environment.prod.ts (51 bytes)
>CREATE angular-nginx-docker/src/environments/environment.ts (662 bytes)
>CREATE angular-nginx-docker/src/app/app.module.ts (314 bytes)
>CREATE angular-nginx-docker/src/app/app.component.css (0 bytes)
>CREATE angular-nginx-docker/src/app/app.component.html (25725 bytes)
>CREATE angular-nginx-docker/src/app/app.component.spec.ts (982 bytes)
>CREATE angular-nginx-docker/src/app/app.component.ts (224 bytes)
>CREATE angular-nginx-docker/e2e/protractor.conf.js (904 bytes)
>CREATE angular-nginx-docker/e2e/tsconfig.json (274 bytes)
>CREATE angular-nginx-docker/e2e/src/app.e2e-spec.ts (671 bytes)
>CREATE angular-nginx-docker/e2e/src/app.po.ts (274 bytes)
>✔ Packages installed successfully.
>    Successfully initialized git.
>```

## Angular App Validation

>Now, let's validate the angular app runs correctly with ng-serve.
>
>```zsh
>cd angular-nginx-docker
>ng serve
>```

## Angular App Server Started

>Once the build and compilation complete, it will start the local server on localhost, port 4200. Let's open up the browser and take a look ([http://localhost:4200](http://localhost:4200)).
>
>```zsh
>drewshirts ng-conf-2021 %cd angular-nginx-docker 
>drewshirts angular-nginx-docker %ng serve
>Compiling @angular/core : es2015 as esm2015
>Compiling @angular/common : es2015 as esm2015
>Compiling @angular/platform-browser : es2015 as esm2015
>Compiling @angular/platform-browser-dynamic : es2015 as esm2015
>✔ Browser application bundle generation complete.
>
>Initial Chunk Files | Names         |      Size
>vendor.js           | vendor        |   2.43 MB
>polyfills.js        | polyfills     | 128.79 kB
>main.js             | main          |  56.00 kB
>runtime.js          | runtime       |   6.15 kB
>styles.css          | styles        | 119 bytes
>
>                    | Initial Total |   2.62 MB
>
>Build at: 2021-05-17T23:30:20.409Z - Hash: b217119e71be0af757fa - Time: 22435ms
>
>** Angular Live Development Server is listening on localhost:4200, open your browser on http://localhost:4200/ **
>
>
>✔ Compiled successfully.
>✔ Browser application bundle generation complete.
>
>Initial Chunk Files | Names  |      Size
>styles.css          | styles | 119 bytes
>
>4 unchanged chunks
>
>Build at: 2021-05-17T23:30:20.905Z - Hash: 6730662d7e09eac1a3b0 - Time: 191ms
>
>✔ Compiled successfully.
>```

## Docker Setup: Dockerfile

> Create a file named ‘Dockerfile’. The init-cap and no extension are on purpose.
>
>```zsh
>touch Dockerfile
>```

## Docker Setup: .dockerignore

>Create a file named '.dockerignore' and add the folder name 'node_modules’. This allows us to ignore our dependencies there just like we do with a .gitignore file.
>
>```zsh
>echo "node_modules" > .dockerignore
>```

## Docker setup: Dockerfile Node Stage

>We’ll be using a two-stage process for building out our Dockerfile.  Our first stage will add a node image, copy all the Angular files to our working directory, run npm install to add project dependencies, and finally will build the app with npm.
>
>```zsh
># Create the node stage with the name "builder"
>FROM node:latest as builder
># Set the working directory
>WORKDIR /app
># Copy files from current directory to working directory
>COPY . .
># Run npm install and build all assets
>RUN npm i && npm run ng build
>```

## Dockerfile Setup: DockerFile Nginx Stage

>Our second stage for our Dockerfile will use an nginx image, set the working directory for nginx assets, remove any default assets from the nginx image, copy our node static assets from the builder image we just created, and specify the entrypoint for our Docker container to run nginx. You can add an additional configuration file for nginx to match your local settings with those of lower test lanes and production, but for this exercise, I’ll keep it simple.
>
>```zsh
># Create the nginx stage for serving content
>FROM nginx:alpine
># Set the working directory to nginx asset directory
>WORKDIR /usr/share/nginx/html
># Remove default nginx static assets
>RUN rm -rf ./*
># Copy static assets from builder stage
>COPY --from=builder /app/dist/angular-nginx-docker .
># Containers run nginx with global directives and daemon off
>ENTRYPOINT ["nginx", "-g", "daemon off;"]
>```

## Build Docker Image

>With these two stages added to our Dockerfile, we are ready to build an image called 'angular-nginx'
>
>```zsh
>docker build -t angular-nginx-docker .
>```

## Docker Build Results

>The build completed will look like this.
>
>```zsh
>drewshirts angular-nginx-docker %docker build -t angular-nginx-docker .
>[+] Building 113.9s (14/14) FINISHED                                                                                                                                                                                                                                              
> => [internal] load build definition from Dockerfile                                                                                                                                                                                                                         0.0s
> => => transferring dockerfile: 691B                                                                                                                                                                                                                                         0.0s
> => [internal] load .dockerignore                                                                                                                                                                                                                                            0.0s
> => => transferring context: 53B                                                                                                                                                                                                                                             0.0s
> => [internal] load metadata for docker.io/library/nginx:alpine                                                                                                                                                                                                              0.0s
> => [internal] load metadata for docker.io/library/node:latest                                                                                                                                                                                                               2.0s
> => [stage-1 1/4] FROM docker.io/library/nginx:alpine                                                                                                                                                                                                                        0.0s
> => [internal] load build context                                                                                                                                                                                                                                            0.1s
> => => transferring context: 6.61kB                                                                                                                                                                                                                                          0.0s
> => [builder 1/4] FROM docker.io/library/node:latest@sha256:fc7a47442a743e34050576adea835cd0fec7f3f75039c9393010b1735d42cef9                                                                                                                                                24.0s
> => => resolve docker.io/library/node:latest@sha256:fc7a47442a743e34050576adea835cd0fec7f3f75039c9393010b1735d42cef9                                                                                                                                                         0.0s
> => => sha256:5478c2b58457ff0b83877f891f774ce08669160d8186a5256b2f47e123b5261f 2.21kB / 2.21kB                                                                                                                                                                               0.0s
> => => sha256:6a8007b5489a26e7af109abe7c1518cc1661ffd1e09e256362fe348dcbac1185 7.80kB / 7.80kB                                                                                                                                                                               0.0s
> => => sha256:d960726af2bec62a87ceb07182f7b94c47be03909077e23d8226658f80b47f87 50.43MB / 50.43MB                                                                                                                                                                             5.5s
> => => sha256:8962bc0fad55b13afedfeb6ad5cb06fd065461cf3e1ae4e7aeae5eeb17b179df 10.00MB / 10.00MB                                                                                                                                                                             2.2s
> => => sha256:fc7a47442a743e34050576adea835cd0fec7f3f75039c9393010b1735d42cef9 1.21kB / 1.21kB                                                                                                                                                                               0.0s
> => => sha256:e8d62473a22dec9ffef056b2017968a9dc484c8f836fb6d79f46980b309e9138 7.83MB / 7.83MB                                                                                                                                                                               1.6s
> => => sha256:65d943ee54c1fa196b54ab9a6673174c66eea04cfa1a4ac5b0328b74f066a4d9 51.84MB / 51.84MB                                                                                                                                                                             9.5s
> => => sha256:532f6f7237092ebd79f21ccd3cf050138b31abeed1b29bac39cfdb30786a615b 192.35MB / 192.35MB                                                                                                                                                                          13.5s
> => => sha256:f8463f32765bd59845c3a0c887354f883b071a0c307469d62a409988d1fcace7 4.20kB / 4.20kB                                                                                                                                                                               5.7s
> => => extracting sha256:d960726af2bec62a87ceb07182f7b94c47be03909077e23d8226658f80b47f87                                                                                                                                                                                    3.2s
> => => sha256:0f0e08571b47c031b8b48e996c6f9ab7e38f8597553524f6fcc24efa46a1b3ae 34.38MB / 34.38MB                                                                                                                                                                             9.0s
> => => sha256:827b2d18feb8df0f6d84131c4d2b1a59208cd7df5d197eebcb026441b4fc6eeb 2.30MB / 2.30MB                                                                                                                                                                               9.4s
> => => extracting sha256:e8d62473a22dec9ffef056b2017968a9dc484c8f836fb6d79f46980b309e9138                                                                                                                                                                                    0.4s
> => => sha256:d9d64980bd3c71e6e660d8e32a58b6e4a9dd16b972e2c9b90085726c421428ce 281B / 281B                                                                                                                                                                                   9.6s
> => => extracting sha256:8962bc0fad55b13afedfeb6ad5cb06fd065461cf3e1ae4e7aeae5eeb17b179df                                                                                                                                                                                    0.4s
> => => extracting sha256:65d943ee54c1fa196b54ab9a6673174c66eea04cfa1a4ac5b0328b74f066a4d9                                                                                                                                                                                    3.2s
> => => extracting sha256:532f6f7237092ebd79f21ccd3cf050138b31abeed1b29bac39cfdb30786a615b                                                                                                                                                                                    7.8s
> => => extracting sha256:f8463f32765bd59845c3a0c887354f883b071a0c307469d62a409988d1fcace7                                                                                                                                                                                    0.1s
> => => extracting sha256:0f0e08571b47c031b8b48e996c6f9ab7e38f8597553524f6fcc24efa46a1b3ae                                                                                                                                                                                    1.6s
> => => extracting sha256:827b2d18feb8df0f6d84131c4d2b1a59208cd7df5d197eebcb026441b4fc6eeb                                                                                                                                                                                    0.2s
> => => extracting sha256:d9d64980bd3c71e6e660d8e32a58b6e4a9dd16b972e2c9b90085726c421428ce                                                                                                                                                                                    0.0s
> => [builder 2/4] WORKDIR /app                                                                                                                                                                                                                                               0.1s
> => [builder 3/4] COPY . .                                                                                                                                                                                                                                                   0.1s
> => [builder 4/4] RUN npm i && npm run ng build                                                                                                                                                                                                                             87.6s
> => CACHED [stage-1 2/4] WORKDIR /usr/share/nginx/html                                                                                                                                                                                                                       0.0s
> => CACHED [stage-1 3/4] RUN rm -rf ./*                                                                                                                                                                                                                                      0.0s
> => CACHED [stage-1 4/4] COPY --from=builder /app/dist/angular-nginx-docker .                                                                                                                                                                                                0.0s
> => exporting to image                                                                                                                                                                                                                                                       0.0s 
> => => exporting layers                                                                                                                                                                                                                                                      0.0s 
> => => writing image sha256:af7561fc75d33832d291e07f8e1023614d1fbf895b05c984c63512b22270cb3f                                                                                                                                                                                 0.0s 
> => => naming to docker.io/library/angular-nginx-docker                                                                                                                                                                                                                      0.0s
>
>Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them
>drewshirts angular-nginx-docker %
>```

## Docker Validation

>When the build is complete, you can validate the image was created with “docker
image ls”. It will show all the Docker images you have on your computer.
>
>```zsh
>drewshirts angular-nginx-docker %docker image ls
>REPOSITORY                                            TAG             IMAGE ID       CREATED         SIZE
>angular-nginx-docker                                  latest          af7561fc75d3   3 weeks ago     26.1MB
>docker101tutorial                                     latest          a82dbe2fa137   10 months ago   26.5MB
>node                                                  12-alpine       06a4a7b5263d   11 months ago   89.3MB
>python                                                alpine          8ecf5a48c789   11 months ago   78.9MB
>nginx                                                 alpine          7d0cdcc60a96   11 months ago   21.3MB
>docker/getting-started                                latest          3c156928aeec   13 months ago   24.8MB
>hub.docker.churchofjesuschrist.org/approved/openjdk   11              243e95d792e3   18 months ago   605MB
>drewshirts angular-nginx-docker %
>```

## Docker Startup

>Building complete, enter when ready. Now we can start a container, serving up our app on localhost, port 8080 ([http://localhost:8080](http://localhost:8080)).
This command won't provide any output until you load the application in the browser.
>
>```zsh
>docker run --rm -it -p 8080:80 angular-nginx-docker
>```

## Docker Startup Validation

>After the container is up and running you can see which containers are up:
>
>```zsh
>docker container ls
>```

## Docker Startup Validation

>When you make changes to your Angular app, you will need to create a new docker
image. Use these when the container is still running. The hash at the end of these
commands is the sha256 hash created when the container is built.
>
>```zsh
>docker container stop af756
>docker container rm af756
>```

## Thank You

>Thank you for your time today! I’m glad I’ve had this opportunity to share. It has helped me learn a lot about using Docker and Nginx. I plan on using this with my current projects at work.
