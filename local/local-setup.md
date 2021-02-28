# local set-up

The following are the components you will need to set-up if you want to run the entire purpleteam solution `local`ly.

If you can't be bothered with all this work, then use the purpleteam-labs `cloud` environment. Head to the [quick-start page](https://github.com/purpleteam-labs/purpleteam-doc/blob/main/quick-start.md).

If you are using the `local` environment, run through all of the set-up steps in this file and the linked resources. Once you have completed the `local` set-up, continue to the [workflow](https://github.com/purpleteam-labs/purpleteam-doc/blob/main/local/local-workflow.md) page to get all of the purpleteam components running `local`ly.

# purpleteam `local` Architecture

The following diagram shows how the purplteam components hang together:

![purpleteam local architecture](../assets/img/purpleteam_local_2021-01-min.png)

# Docker Network

Before most of the supporting commands can be run, such as running docker-compose-ui (which hosts the stage two containers), and sam-cli (which hosts the purpleteam lambda functions `local`ly), the [`compose_pt-net`](https://github.com/purpleteam-labs/purpleteam-orchestrator/blob/b4502fe50cfe151edb70ef1be376a70c58a78729/compose/orchestrator-testers-compose.yml#L4) Docker network needs to be created.
You can create this bridge network manually, alternatively once you have the purpleteam-orchestrator repository cloned, run the following two commands from the purpleteam-orchestrator root directory:

```
npm run dc-build
```
```
npm run dc-up
```

`dc-build` will build the stage one Docker services (images).  
`dc-up` will create the required Docker network and containers, it will then bring all containers up that are listed (stage one) in the [orchestrator-testers-compose.yml](https://github.com/purpleteam-labs/purpleteam-orchestrator/blob/b4502fe50cfe151edb70ef1be376a70c58a78729/compose/orchestrator-testers-compose.yml) file.

# Your System Under Test (SUT)

Obviously you are going to need a web application that you want to put under test. A couple of options for you to start to experiment with if you don't yet have a SUT in a Docker container ready to be tested are:

1. If you want to target a local SUT, it needs to be running in a Docker container. Clone a copy of [NodeGoat](https://github.com/OWASP/NodeGoat) and make the following changes:  
   * Add the following docker-compose-override.yml file to the root directory  
     ```yaml
     version: "3.7"
     
     networks:
       compose_pt-net:
         external: true
     
     services:
       web:
         networks:
           compose_pt-net:
         depends_on:
           - mongo
         container_name: pt-sut-cont
         environment:
           - NODE_ENV=production
       mongo:
         networks:
           compose_pt-net:
     ```
        This option means that NodeGoat will be running in the `pt-net` network created by the orchestrator's docker-compose
   * Change the passwords in the artifacts/db-reset.js file.  
   
   When you are ready to bring NodeGoat up, just run the following command from the NodeGoat root directory:  
     ```shell
     docker-compose up
     ```
2. An alternative is to spin up [purpleteam-iac-sut](https://github.com/purpleteam-labs/purpleteam-iac-sut). Purpleteam-labs use this for both `local` and `cloud` testing

# Lambda functions

Details on installing and configuring the aws cli and aws-sam-cli in order to be able to host the purpleteam lambda functions `local`ly can be found [here](https://github.com/purpleteam-labs/purpleteam-lambda)

# Stage Two containers

Details on configuring and debugging (gaining insights into what is happening inside the containers) if needed can be found [here](https://github.com/purpleteam-labs/purpleteam-s2-containers)

# Orchestrator

To run the stage one containers (orchestrator, testers and redis) we use npm scripts from the purpleteam-orchestrator project to run the [docker-compose files](https://github.com/purpleteam-labs/purpleteam-orchestrator/tree/main/compose).
The main docker-compose file is [orchestrator-testers-compose.yml](https://github.com/purpleteam-labs/purpleteam-orchestrator/blob/b4502fe50cfe151edb70ef1be376a70c58a78729/compose/orchestrator-testers-compose.yml).

## Environment Variables

The orchestrator-testers-compose.yml file has a bind mount that expects the `HOST_DIR` environment variable to exist and be set to a host directory of your choosing.
Make sure you have a source directory set-up and the `HOST_DIR` environment variable has its  value assigned to it.
We usually add this to a .env file in root directory of the purpleteam-orchestrator.

`HOST_DIR=</mnt/your-spare-drive/Logs/purpleteam/outcomes>`

This host directory gets written to by the testers and orchestrator and read from the orchestrator. This directory needs group `rwx` permissions at least.

## Set-up for Running Emissaries

### Firewall Rules

If you use a firewall, you may have to make sure that the purpleteam components can communicate with each other.

1. [Testers](https://github.com/purpleteam-labs/purpleteam-doc/blob/main/definitions.md) need to communicate with the locally running Lambda service in order to start and stop [stage two containers](https://github.com/purpleteam-labs/purpleteam-s2-containers).  

  Communications (TCP) will need to flow from the [app-scanner](https://github.com/purpleteam-labs/purpleteam-app-scanner) container ([`pt-app-scanner-cont`](https://github.com/purpleteam-labs/purpleteam-orchestrator/blob/4324d85e13ffa637b6da75079e08a3c6595f619d/compose/orchestrator-testers-compose.yml#L40)) bound to the [`pt-net`](https://github.com/purpleteam-labs/purpleteam-orchestrator/blob/4324d85e13ffa637b6da75079e08a3c6595f619d/compose/orchestrator-testers-compose.yml#L4) (or listed as `compose_pt-net` with `docker network ls`) Docker network IP address of [`172.25.0.120`](https://github.com/purpleteam-labs/purpleteam-orchestrator/blob/4324d85e13ffa637b6da75079e08a3c6595f619d/compose/orchestrator-testers-compose.yml#L22) - to the IP address and port that local Lambda is listening on (172.25.0.1:3001) which can be seen in the `sam local start-lambda` commands (as seen in the [local-workflow documentation](https://github.com/purpleteam-labs/purpleteam-doc/blob/e761c374d43f94496983b5c7dc4d6d4c69b61f58/local/local-workflow.md)) used to host the lambda functions

### Host IP Forwarding

Make sure host [IP forwarding](https://www.dedoimedo.com/computers/docker-networking.html#mozTocId387645) is [turned on](https://linuxconfig.org/how-to-turn-on-off-ip-forwarding-in-linux).

<br>

Details on installing the orchestrator dependencies and configuring can be found  [here](https://github.com/purpleteam-labs/purpleteam-orchestrator).

# Testers

Currently purpleteam has the app-scanner implemented. The server-scanner and tls-checker are stubbed out and waiting to be implemented.

Additional Testers can be added by community contributors.

## Application Scanner

Details on installing the app-scanner dependencies and configuring can be found [here](https://github.com/purpleteam-labs/purpleteam-app-scanner)

## Server Scanner

Not yet implemented.

Details on installing the server-scanner dependencies and configuring once implemented will be found [here](https://github.com/purpleteam-labs/purpleteam-server-scanner)

## TLS Checker

Not yet implemented.

Details on installing the tls-checker dependencies and configuring once implemented will be found  [here](https://github.com/purpleteam-labs/purpleteam-tls-checker)

# purpleteam (CLI)

Details on installing the purpleteam (CLI) dependencies, configuring and running can be found [here](https://github.com/purpleteam-labs/purpleteam)

<br>

Once completed the `local` set-up, head to the [workflow](https://github.com/purpleteam-labs/purpleteam-doc/blob/main/local/local-workflow.md) page to get all of the purpleteam components running `local`ly.
