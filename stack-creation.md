# Creating a new Appsody stack

[Appsody](https://appsody.dev) is a new open source project designed to simplify starting cloud native application development. The fundamental object is a stack, which builds a pre-configured Docker image ready to be deployed in a cloud environment. Images can include any amount of customized content, and allow stack builders to decide which parts are fixed (stack image) and which parts stack users can modify/extend (templates).

## The role of a stack in the development process

The fundamental goal of stacks is to simplify the life of the developer trying to build an application using a specific set of technologies or development pattern. While there are an increasing number of publicly available stacks, many enterprises are likely to want to build their own set of stacks that match their specific requirements and standards they want to lay down for the development of cloud native applications within their organization.

If a new stack needs to be created, then this tutorial will take you through the steps of doing this. However, before leaping into that, it is worth stepping back and refreshing the design requirements on stacks. A stack is designed to support the developer in two ways of working:

1. **Rapid Local Development Mode**  
In this mode, the stack provides everything to enable the development of a new application on the local machine, with the application *always* being run in a (local) containerized Docker environment. The fact that application development uses containerization from the start (as opposed to development solely in the user space of the local machine) minimizes the chances of introducing subtle issues in the containerization process, and removes the need for a developer to install the core technology components that will underpin their application. The requirements on the stack, therefore, are to have all the dependencies for the particular technologies involved pre-built into the Docker image, and also to dynamically compliment these with whatever dependencies are added explicitly by the developer for their code. Rapid local development mode consists of the appsody CLI (hooked into a local IDE if required), communicating with a local Docker container that is running the application under development. Furthermore, local development is accelerated by enabling application code to be held on the local file system, whilst being mounted in the Docker container, so that a local change can automatically trigger a restart of the application.

2. **Build and Deploy Mode**  
In this mode, the stack enables the appsody CLI to build a self-contained Docker image that includes both the core technologies in the stack plus the application code that has been developed, along with the combined dependencies of both. The resulting image can then be deployed manually or programmatically to any platform that supports Docker images (such as a local or public Kubernetes cluster).

## Stack structure

Given the fact that a single stack is designed to enable both the above methods of working (under the control of the Appsody software), there is a standard structure that all stacks follow. The structure below represents the source structure of a stack:

```bash
my-stack
├── README.md
├── stack.yaml
├── image/
|   ├── config/
|   |   └── app-deploy.yaml
|   ├── project/
|   |   ├── [files that provide the technology components of the stack]
|   |   └── Dockerfile
│   ├── Dockerfile-stack
|   └── LICENSE
└── templates/
    ├── my-template-1/
    |       └── [example files as a starter for the application, e.g. "hello world"]
    └── my-template-2/
            └── [example files as a starter for a more complex application]

```

The structure above is then processed when you build a stack, to generate a Docker image for the stack, along with tar files of each of the templates, which can then all be stored/referenced in a local or public appsody repo. The appsody CLI can then access such a repo, to use the stack to initiate local development.

## Let's create a new stack!

The first thing in creating a new stack is to construct a scaffold of the above structure. Luckily appsody provides a sample stack that has this, and much more. Taking a copy of this will get us started. Here are the steps to do this:

First, clone the appsody stacks repository
```bash
$ git clone https://github.com/appsody/stacks.git
```
You will now have the full stacks source structure in your local environment,
```bash
$ cd stacks
$ ls -l
-rw-r--r--   1 henrynash  staff  11357 25 Jul 12:32 LICENSE
-rw-r--r--   1 henrynash  staff   2897 19 Aug 12:48 README.md
-rw-r--r--   1 henrynash  staff   6465  4 Sep 13:35 RELEASE.md
-rw-r--r--   1 henrynash  staff    510 25 Jul 12:32 TECHNICAL_REQUIREMENTS.md
drwxr-xr-x  15 henrynash  staff    480 12 Sep 13:13 ci
drwxr-xr-x   7 henrynash  staff    224  4 Sep 13:35 experimental
drwxr-xr-x   9 henrynash  staff    288  4 Sep 13:35 incubator
-rw-r--r--   1 henrynash  staff   2861 11 Sep 14:26 index.yaml
drwxr-xr-x   3 henrynash  staff     96 25 Jul 12:32 samples
drwxr-xr-x   3 henrynash  staff     96 25 Jul 12:32 stable
```

Stacks are classified as being `stable`, `incubating` or `experimental` (as described [here](https://appsody.dev/docs/stacks/stacks-overview)). If you list any of those three directories in your source you will see the source for stacks in each of those classifications. There is also the `samples` directory, that contains a `sample-stack`. This is something you can copy and rename to get you started. Given that we are experimenting, let's copy and rename this into the experimental directory.

```bash
$ cp -r samples/sample-stack experimental
$ cd experimental
$ mv sample-stack mystack
```

In fact, the sample stack we have just copied is more than just a scaffold - it is actually a stack you can build and run (it doesn't do much, by design, but it will ensure you have your appsody stack building environment working before you start making changes). Building a stack will create a stack image, that can then be used by the appsody CLI to initiate a project using that stack. Before you start, make sure you have a Docker environment set up (a local docker environment is fine, and indeed recommended). A stack image is, in fact, just a Docker image - and there is a Docker file (`Dockerfile-stack`) within the sample stack structure you copied. While you can run a Docker build directly using this Docker file, it is better to use the appsody ci build scripts - since they will help you install the new stack into an appsody repo (which is then accessible directly by the appsody CLI). To build our new stack in this way, from the parent `stacks` directory:

```bash
$ ./ci/build.sh . experimental/mystack
```

This will run a Docker build under the covers, install mystack into a local appsody repository (called `experimental-local-index`) as well as run some basic tests on it to make sure it is well formed. Once the build is complete, you can check that it is now available in the local repo:

```bash
$ appsody list experimental-index-local
REPO                            ID      VERSION         TEMPLATES       DESCRIPTION
experimental-index-local        mystack 0.1.0           *simple         sample stack to help...
```

So at this point we have built and installed our copy of the sample stack - and we can actually try it out. Let's create a new directory and initialize it with this new appsody stack:

```bash
$ mkdir ~/test
$ cd ~/test
$ appsody init experimental-index-local/mystack
```

The above will set up this `test` folder based on the default template in mystack, which you can now inspect:

```bash
$ ls -l
drwxr-xr-x   6 henrynash  staff   192 14 Sep 19:52 .
drwxr-xr-x  41 henrynash  staff  1312 14 Sep 19:52 ..
-rw-r--r--   1 henrynash  staff    27 14 Sep 19:52 .appsody-config.yaml
drwxr-xr-x   3 henrynash  staff    96 14 Sep 19:52 .vscode
-rw-r--r--   1 henrynash  staff    26 14 Sep 19:52 hello.sh
drwxr-xr-x   3 henrynash  staff    96 14 Sep 19:52 tests
```

The file .appsody-config.yaml points at the stack, while hello.sh is the starter application (about as simple as you can get, it just echos a line of text saying "Hello from Appsody!"). We can run this starter application (in a container environment) by:

```bash
$ appsody run
Running development environment...
Running command: docker[pull appsody/mystack:0.1]
[Warning] Docker image pull failed: exit status 1
Using local cache for image appsody/mystack:0.1
Running docker command: docker[run --rm -p 8080:8080 --name test20-dev -v /Users/henrynash/codewind-workspace/test20/:/project/userapp -v test20-deps:/project/deps -v /Users/henrynash/.appsody/appsody-controller:/appsody/appsody-controller -t --entrypoint /appsody/appsody-controller appsody/mystack:0.1 --mode=run]
[Container] Running: /bin/bash /project/userapp/hello.sh
[Container] Hello from Appsody!
```

The above indicates that our sample stack successfully took the simple hello.sh, within a container running in your Docker environment.

-----
**Note** Since we hadn't pushed our new stack to dockerhub, appsody couldn't find it there and used the local cache instead. If you want to prevent appsody trying to do a remote download if it is already in the local cache (and hence avoid this Warning line), you can set the following environment variable:

```bash
export APPSODY_PULL_POLICY=IFNOTPRESENT
```
-----

OK, finally, we have our environment set up, a sample stack all working - and we can start modifying it to create the new stack that we actually want. When building a new stack you need to consider a number of things:

* What set of technologies do you need installed in your stack, and how will you ensure all the dependencies are also installed?
* What kind of sample starter application(s) would you want to provide?
* How will you ensure that the user of your stack can install any additional dependencies they might need as they build out their application?

For this tutorial, we are going to create a stack that can be used to easily build a python http server - and we will consider the above questions as we do this. For the first of these questions, we will configure our stack to use the popular web framework `flask`. As this is going to be a python stack, we'll use the python packaging tool `pipenv` to install it and any dependencies. For a starter application, we'll just have simple "Hello World" application that will respond to a url of `/hello`. Let's look at the steps we need to carry out, then we will dive in and execute them one by one.

1. **Modify the Docker file for the stack image**  
This the first of two Docker files we will modify - this one (`Dockerfile-stack`), in the `image` directory within the stack source structure, will be used by Docker to build an image of the stack itself (for use by the appsody CLI). We'll need to chose base image, include the pipenv commands to install flask, set the appsody docker environment variables to the appropriate settings, and set any relevant ports etc. needed.

2. **Decide on the architecture of how the components in the stack will interact with any user application**  
The nub of this is often whether there is some server process that is in control or whether the user application is in control (essentially, if you where writing a regular Docker container, what would be the entry point?). In this case, the `flask` application will be in control, and in order to minimize the code the user has to write, we'll architect it so that the user just has to provide whatever url entry points they need for their application - and we will make the entry point to be server code we provide as part of the stack.

3. **Write our sample Hello World application in the template directory**  
As just discussed, `flask` will be managing the web environment, and ensures that as a developer all you need to do is provide code for any endpoints you are exposing. We'll cerate a single python file that contains our `/hello` endpoint. The idea is that the user of your stack can take this file and expand it to contain all the endpoints in their actual application.

4. **Modify the Docker file for the final application image**  
This second Docker file (`Dockerfile`), in the `image/project` directory within the stack source structure, will be used by Docker to build the final application image (that contains everything from the stack and the user written application). This build is carried out by the appsody CLI `build` and/or `deploy` commands. This Docker file has the responsibility of ensuring the combined dependencies are installed in this final image.

### Modify the Docker file for the stack image

There is already an example `Dockerfile-stack` in your sample stack - and we will use this as a basis for the Docker file we want. It has a nu,ber of the appsody environment variables already configured, that much the strcuture of the sample stack, to save you having to start from scratch. Let's go ahead and make the changes we need for our new stack.

The first is to select a base image. For us, a good base image to use is the standard Docker python image, so let's replace the current RedHat image selection defined by the `FROM` label at the top of the file with:

```bash
FROM python:3.7
```

The next step is modify the appropriate appsody environment variables. There are quite a number of these (as already shown in the sample stack), but to be able to just run our new stack with an application, there are only 3 we need to change to get us started:

First we need to update the `APPSODY_RUN` environment variable, since this is what appsody will pass control to when you issue the 'appsody run' command, when operating in Rapid Local Development Mode. This is typically whatever command you would use to execute your main technology service if you were running it manually - for flask it is as follows:

```bash
ENV APPSODY_RUN="python -m flask run --host=0.0.0.0 --port=8080"
```

In Rapid Local Development Mode your application can be restarted on detection of source file changes in the user application directory (this isn't waiting for a git commit, it is directly watching for files being changed), and in case you need to do anything special on a re-start, this is kept in its own environment variable (`APPSODY_RUN_ON_CHANGE`). For Flask, however, nothing special is needed, so we can set this to be the same:

```bash
ENV APPSODY_RUN_ON_CHANGE="python -m flask run --host=0.0.0.0 --port=8080"
```

Finally, we should tell appsody which files to watch that will be considered a relevant change and our application will be re-run. This is specified using the `APPSODY_WATCH_REGEX` variable. For a python stack, we might set this to any file ending in `.py`.

```bash
ENV APPSODY_WATCH_REGEX="^.*.py$"
```

We'll come back and update additional environment variables later in this tutorial (and for reference you can find all of them described [here] (https://appsody.dev/docs/stacks/environment-variables).

So now that we have the environment variables set, we need to ensure that the flask module is installed, along with any dependencies. We will use `pipenv` to help us build a list of dependencies, and then use pip to install those dependencies into the dependency directory (which is already defined in the sample stack in the line `ENV APPSODY_DEPS=/project/deps`). We then set the python path to pick up these dependencies. Don't worry if you are not that familiar with dependencies in python - the key take away here is that we need to provide some code in the Docker file that will install whatever technology components we need, plus their dependencies.

Finally, we tell flask the entry point (which is server code we will write in the next step). To do all the above, we add the following to `Dockerfile-stack`, typically towards the end of the file after the `WORKDIR` is set.

```bash
RUN pip install pipenv
RUN pipenv install flask==0.11.1
RUN pipenv lock -r > requirements.txt
RUN python -m pip install -r requirements.txt -t /project/deps
ENV PYTHONPATH=/project/deps
ENV FLASK_APP=/project/server/__init__.py
```

### Provide the server side of the architecture that will support the user application

Now that we have our Docker file fixed up, we need to provide that server entry point we defined above. We will, just for clarity, use a sub-directory to the `project` directory to hold our server code, and within that create a file that contains our server:

```bash
$ mkdir server
$ cd server
$ cat <<EOF > __init__.py
from flask import Flask

app = Flask(__name__)

from userapp import *
EOF
```

The above will start a flask application (`app`) and the import and python files the user writes into it so that they are part of the server. That's all we need to do for the server side!

### Write our sample Hello World application in the template directory

Finally, we create a sample app - which as we discussed just responds to a single url end point. This is added in the `templates/simple` directory:

```bash
$ cat <<EOF > __init__.py
from server import app

@app.route('/hello')
def HelloWorld():
    return 'Hello from Appsody!'
EOF
```

## Building and running our new stack

We now have the minimal support required for our new python/flask stack - so let's try it out! We use the same stack building command we used at the start to check out our sample stack and build environment:

```bash
$ ./ci/build.sh . experimental/mystack
```

Provided this builds successfully, we are ready to run it. We should create a new directory and re-initialize using appsody, e.g.:

```bash
$ mkdir ~/test1
$ cd ~/test1
$ appsody init experimental-index-local/mystack
```

We should now see that our new hello world python app has been installed for us

```bash
$ ls -l
drwxr-xr-x   9 henrynash  staff   288 16 Sep 00:22 .
drwxr-xr-x  43 henrynash  staff  1376 16 Sep 00:00 ..
-rw-r--r--   1 henrynash  staff    27 16 Sep 00:00 .appsody-config.yaml
drwxr-xr-x   3 henrynash  staff    96 16 Sep 00:00 .vscode
-rw-r--r--   1 henrynash  staff    96 16 Sep 00:00 __init__.py
-rw-r--r--   1 henrynash  staff    26 16 Sep 00:00 hello.sh
drwxr-xr-x   3 henrynash  staff    96 16 Sep 00:00 tests
```

-----

**Note** The original `hello.sh` is still there because we haven't yet deleted it from our stack - and since we have not yet hooked in the `TEST` and `DEBUG` options for our flask server, that's OK, since those facilities still reference it. Once we have added these, we can remove `hello.sh`.

-----

We can now run our app and stack using appsody:

```bash
$ appsody run
Running development environment...
Running command: docker[pull appsody/mystack:0.1]
[Warning] Docker image pull failed: exit status 1
Using local cache for image appsody/mystack:0.1
Running docker command: docker[run --rm -p 8080:8080 --name test22-dev -v /Users/henrynash/codewind-workspace/test22/:/project/userapp -v test22-deps:/project/deps -v /Users/henrynash/.appsody/appsody-controller:/appsody/appsody-controller -t --entrypoint /appsody/appsody-controller appsody/mystack:0.1 --mode=run]
[Container] Running: python -m flask run --host=0.0.0.0 --port=8080
[Container]  * Serving Flask app "server"
[Container]  * Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)
```

Note that now flask is the running application - and if we where to hit our published endpoint, we should get a response:

```bash
$ curl http://0.0.0.0:8080/hello
Hello from Appsody!
```

We can also check that appsody will restart the server if we make a change to our hello world application. For instance, if we edit `__init__.py` in the templates/simple directly to change the message response for the /hello url to, say, "Hello again from Appsody!" and save the file, you should see the flask server automatically restart, and then if you again hit the end point you should see the updated message:

```bash
$ curl http://0.0.0.0:8080/hello
Hello again from Appsody!
```

We are now running our new stack in Rapid Local Development Mode, and a developer could now build out their application, with their changes being immediately made live in their container for testing. Now that we have the basics running, we will go on to augment the stack with the other changes needed for a full solution, namely:

* Support TEST and DEBUG phases of Rapid Local Development Mode
* Support for inclusion of dependencies from the user application
* Support for appsody build and deploy
* Future improvements proposed as a follow on to this tutorial

## Support TEST and DEBUG phases of Rapid Local Development Mode

TBA

## Support for inclusion of dependencies from the user application

TBA

## Support for appsody build and deploy

TBA

## Future improvements proposed as a follow on to this tutorial

TBA
