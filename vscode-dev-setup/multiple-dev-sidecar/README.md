# Multiple Sidecar Development Containers

## Overview

This development environment consists of multiple development containers. Each dev container is specialized for developing and debugging a single app. The Dapr components - the `daprd` daemon and the Dapr `placement` daemon - run in separate containers. Although it's possible to run the Dapr components within the same containers as the Node.js and Python development containers, this sample demonstrates a simpler debugging configuration for Node.js and Python by running the `daprd` daemons in separate containers.

An important difference between this sample and the other VS Code container samples is that in this sample VS Code attaches to an already running container instead of launching the container from VS Code.

```ASCII
Host machine (Windows 10, version 2004, with Docker Desktop)
    |
    -- WSL 2: local git repo
    |
    -- Custom Docker network
        |
        -- Dapr placement container
        |
        -- Redis container (state storage and pub/sub)
        |
        -- Dapr zipkin container
        |
        -- Node.js app development container, with VS Code attached, sidecar to daprd container
        |
        -- daprd container for the Node.js app
        |
        -- Python app development container, with VS Code attached, sidecar to daprd container
        |
        -- daprd container for the Python app
```

### Docker Compose

The custom Docker network and the containers for Dapr, Redis, and Zipkin are defined with Docker Compose, which simplifies the provisioning of required containers on the host machine and also ensures that DNS works among all machines on the network so that machines can reference each other by their DNS names. (You could build the same environment using Docker commands instead of Docker Compose.)

## How To Run the Sample

### Prerequisites

Your host machine must be running Docker and Docker Compose.

### Setup

1. Pull this repository to your host machine.
1. Open a terminal window, and navigate to the `multiple-dev-sidecar` folder, where you should see a `docker-compose.yml` file.
1. Run the command `docker-compose up`, which will result in three containers starting:

    - `redis-dev`: standalone Redis instance for state storage and pub/sub messaging
    - `zipkin-dev`: logs, which are available from the host machine at [http://localhost:9411](http://localhost:9411)
    - `dapr-placement-dev`: Dapr actor placement container, which is not requried for this demo because actors are not used, but here to show how it would be done
    - `node-dev`: VS Code connects to this container for Node.js development
    - `dapr-nodeapp-dev`: container that runs the daprd process for the node-dev container
    - `python-dev`: VS code connects to this container for Python app development
    - `dapr-python-dev`: container that runs the daprd process for the python-dev container

   > Note: when you run the `docker-compose up` command, your terminal window will pause until you cancel the command (`Ctrl+C` in BASH), but you can also run the command in the background with the `--detach` parameter, as `docker-compose up --detach`. When you are ready to stop the containers, use `docker-compose stop`. However, it is sometimes useful to view the running console output of `docker-compose`.

## Step-By-Step

### Start the containers

1. Open a terminal window in WSL, Linux, or PowerShell and navigate to the folder containing this `README.md` file.

2. Run this command to start the containers

    ```BASH
    docker-compose up
    ```

You should see the console output of the containers starting.

### Run the node app development container

1. Open a terminal window in WSL, Linux, or PowerShell and navigate to the folder containing this `README.md` file.

1. Open VS Code to the `multiple-dev-container/node` folder.

    > IMPORTANT: The folder opened in VS Code must have the `.devcontainer` folder at the root level. Files in that folder define the container VS Code will build and attach to.

    > NOTE: From WSL on Windows, a simple way to open VS Code to the correct folder is to navigate to the folder in a terminal, and type the command `code .` (make sure to include the trailing "`.`" character).

1. From the VS Code command palette (Ctrl+Shift+P), run this command:

    ```ASCII
    Remote-Containers: Open Folder in Container...
    ```

    VS Code will prompt you to choose a folder. If you started in the `node` foder, just click OK. If you started in a different folder, navigate to the folder using the VS Code command dialog, then click OK.

1. When VS Code has attached to the running container, you can press `F5` to build and start debugging.

    When you launch the debugger you will see a terminal window open with the output of the `daprd` task. Click on the `DEBUG CONSOLE` to see the application output. It should look something like this:

    ```ASCII
    /usr/local/bin/node app.js
    Debugger listening on ws://127.0.0.1:41305/c1e69cbe-55fb-46f5-a695-57221e293793
    For help, see: https://nodejs.org/en/docs/inspector
    Debugger attached.
    Node App listening on port 3000!
    ```

    When the python app is started from a separate instance of VS Code (see below for step-by-step instructions), you will see debugging output in the window that looks like this:

    ```ASCII
    /usr/local/bin/node app.js
    Debugger listening on ws://127.0.0.1:41305/c1e69cbe-55fb-46f5-a695-57221e293793
    For help, see: https://nodejs.org/en/docs/inspector
    Debugger attached.
    Node App listening on port 3000!
    Got a new order! Order ID: 1
    Successfully persisted state.
    Got a new order! Order ID: 2
    Successfully persisted state.
    Got a new order! Order ID: 3
    Successfully persisted state.
    ```

See the files in the `.vscode` folder for details on how debugging is configured.  

Zipkin is running as a container on your host machine, so you can view Zipkin logs in a host machine browser by navigating to [http://localhost:9411](http://localhost:9411).  

Dapr configuration is stored in `utils/.dapr` so that you can have configuration that is different between a regular deployment and a development container.

### Run the python app development container

The python app is run from a separate instance of VS Code.

1. Open a new instance of VS Code to the `multiple-dev-container/python` folder. (Do not close the instance of VS Code that is attached to the node app development container.)
1. From the VS Code command palette (Ctrl+Shift+P), run this command:

    ```ASCII
    Remote-Containers: Reopen in Container
    ```

1. When VS Code has attached to the running container, you can press `F5` to build and start debugging.

    When you launch the debugger you will see a terminal window open with the output of the `daprd` task. Click on the `DEBUG CONSOLE` to see the application output. It should look something like this:

    ```ASCII
    Sending order 1
    Sending order 2
    Sending order 3
    ```

1. Switch to the instance of VS Code that is running the node app. (The app should be running with the debugger attached.) You should see output like this in the `DEBUG CONSOLE`.

    ```ASCII
    /usr/local/bin/node app.js
    Debugger listening on ws://127.0.0.1:41305/c1e69cbe-55fb-46f5-a695-57221e293793
    For help, see: https://nodejs.org/en/docs/inspector
    Debugger attached.
    Node App listening on port 3000!
    Got a new order! Order ID: 1
    Successfully persisted state.
    Got a new order! Order ID: 2
    Successfully persisted state.
    Got a new order! Order ID: 3
    Successfully persisted state.
    ```
