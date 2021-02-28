# local workflow

Make sure you have been through the `local` [set-up](/local/local-setup) page and followed all the steps before working through this workflow page.

# Emulating the AWS Lambda service

Leaving `docker stats` running in a terminal is often useful to see which containers are running and how much work they're doing.

Adding the `--debug` flag to the end of `sam local` commands can be helpful.

We have tried [these launch configurations](https://aws.amazon.com/blogs/developer/introducing-launch-configurations-support-for-sam-debugging-in-the-aws-toolkit-for-vs-code/). The fact that you are expected to add your environment variables in yet another place, specify your runtime again, and add your event again, along with the fact that some environment variables would override and some wouldn't was enough to make us stick with our existing VS Code launch configurations.

## Testing the [`provisionAppEmissaries`](https://github.com/purpleteam-labs/purpleteam-lambda/blob/main/local/app-emissary-provisioner/index.js)

1. Terminal 1: Run docker-compose-ui from the `purpleteam-s2-containers/` root directory. Run the following command:  
   ```shell
   docker run --name docker-compose-ui -v $(pwd):$(pwd) -w $(dirname $(pwd)) -p 5000:5000 --rm --network compose_pt-net -v /var/run/docker.sock:/var/run/docker.sock francescou/docker-compose-ui:1.13.0
   ```
2. Terminal 2: Host Lambda function. from the `purpleteam-lambda/` root directory using [`sam local start-lambda`](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-local-start-lambda.html). Run the following command:  
   ```shell
   sam local start-lambda --host 172.25.0.1 --env-vars local/env.json --docker-network compose_pt-net
   ```
3. Terminal 3: Invoke `provisionAppEmissaries` from aws cli. From the `purpleteam-lambda/` root directory, start 10 containers:  
   ```shell
   aws lambda invoke --function-name "provisionAppEmissaries" --endpoint-url "http://172.25.0.1:3001" --no-verify-ssl --payload '{"provisionViaLambdaDto":{"items": [{"testSessionId":"lowPrivUser", "browser":"chrome", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"adminUser", "browser":"firefox", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"lowPrivUser", "browser":"chrome", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"adminUser", "browser":"firefox", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"lowPrivUser", "browser":"chrome", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"adminUser", "browser":"firefox", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"lowPrivUser", "browser":"firefox", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"adminUser", "browser":"firefox", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"lowPrivUser", "browser":"chrome", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"adminUser", "browser":"chrome", "appEmissaryContainerName":"", "seleniumContainerName":""}]}}' local/app-emissary-provisioner/out.txt
   ```
   
   Just be aware on your first run that the stage two images are going to be fetched, so it won't be instant
   
4. If you haven't already got `docker stats` running, verify that the containers were started with the following command:  
   ```shell
   docker container ls
   ```
5. Bring containers down with one of the following options:
   * The following command from the `purpleteam-s2-containers/app-emissary/` directory:  
     ```shell
     docker-compose down
     ```
   * Using the `deprovisionS2Containers` Lambda as seen below
   * Possibly the easiest way: using the docker-compose-ui UI as discussed in the last step of the Full system run below

## Testing the [`provisionSeleniumStandalones`](https://github.com/purpleteam-labs/purpleteam-lambda/blob/main/local/selenium-standalone-provisioner/index.js)

1. Terminal 1: Run docker-compose-ui from the `purpleteam-s2-containers/` root directory. Run the following command:  
   ```shell
   docker run --name docker-compose-ui -v $(pwd):$(pwd) -w $(dirname $(pwd)) -p 5000:5000 --rm --network compose_pt-net -v /var/run/docker.sock:/var/run/docker.sock francescou/docker-compose-ui:1.13.0
   ```
2. Terminal 2: Host Lambda function. from the `purpleteam-lambda/` root directory using [`sam local start-lambda`](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-local-start-lambda.html). Run the following command:  
   ```shell
   sam local start-lambda --host 172.25.0.1 --env-vars local/env.json --docker-network compose_pt-net
   ```
3. Terminal 3: Invoke `provisionSeleniumStandalones` from aws cli. From the `purpleteam-lambda/` root directory, start 10 containers:  
   ```shell
   aws lambda invoke --function-name "provisionSeleniumStandalones" --endpoint-url "http://172.25.0.1:3001" --no-verify-ssl --payload '{"provisionViaLambdaDto":{"items": [{"testSessionId":"lowPrivUser", "browser":"chrome", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"adminUser", "browser":"firefox", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"lowPrivUser", "browser":"chrome", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"adminUser", "browser":"firefox", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"lowPrivUser", "browser":"chrome", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"adminUser", "browser":"firefox", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"lowPrivUser", "browser":"firefox", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"adminUser", "browser":"firefox", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"lowPrivUser", "browser":"chrome", "appEmissaryContainerName":"", "seleniumContainerName":""},{"testSessionId":"adminUser", "browser":"chrome", "appEmissaryContainerName":"", "seleniumContainerName":""}]}}' local/selenium-standalone-provisioner/out.txt
   ```
   
   Just be aware on your first run that the stage two images are going to be fetched, so it won't be instant
   
4. If you haven't already got `docker stats` running, verify that the containers were started with the following command:  
   ```shell
   docker container ls
   ```
5. Bring containers down with one of the following options:
   * The following command from the `purpleteam-s2-containers/selenium-standalone/` directory:  
     ```shell
     docker-compose down
     ```
   * Using the `deprovisionS2Containers` Lambda as seen below
   * Possibly the easiest way: using the docker-compose-ui UI as discussed in the last step of the Full system run below

## Testing the [`deprovisionS2Containers`](https://github.com/purpleteam-labs/purpleteam-lambda/blob/main/local/s2-deprovisioner/index.js)

1. Terminal 1: Run docker-compose-ui from the `purpleteam-s2-containers/` root directory. Run the following command:  
   ```shell
   docker run --name docker-compose-ui -v $(pwd):$(pwd) -w $(dirname $(pwd)) -p 5000:5000 --rm --network compose_pt-net -v /var/run/docker.sock:/var/run/docker.sock francescou/docker-compose-ui:1.13.0
   ```
2. Terminal 2: Host Lambda function. from the `purpleteam-lambda/` root directory using [`sam local start-lambda`](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-local-start-lambda.html). Run the following command:  
   ```shell
   sam local start-lambda --host 172.25.0.1 --env-vars local/env.json --docker-network compose_pt-net
   ```
3. Terminal 3: Invoke `deprovisionS2Containers` from aws cli. From the `purpleteam-lambda/` root directory, run the following command:  
   ```shell
   aws lambda invoke --function-name "deprovisionS2Containers" --endpoint-url "http://172.25.0.1:3001" --no-verify-ssl --payload '{"deprovisionViaLambdaDto":{"items": ["app-emissary", "selenium-standalone"]}}' local/s2-deprovisioner/out.txt
   ```
4. If you haven't already got `docker stats` running, verify that the containers were brought down with the following command:  
   ```shell
   docker container ls
   ```

## [Testing Lambda function directly](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-using-invoke.html)

No local Lambda service needs to be running first, in our case:

1. Run docker-compose-ui as already discussed
2. Then simply run one of the following [`sam local invoke`](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-local-invoke.html) commands:  
   * For `provisionAppEmissaries`, from the `purpleteam-lambda/` root directory, run the following command:  
     ```shell
     echo '<same-JSON-payload-as-above>' | sam local invoke --event - --env-vars local/env.json --docker-network compose_pt-net provisionAppEmissaries
     ```
   * For `provisionSeleniumStandalones`, from the `purpleteam-lambda/` root directory, run the following command:  
     ```shell
     echo '<same-JSON-payload-as-above>' | sam local invoke --event - --env-vars local/env.json --docker-network compose_pt-net provisionSeleniumStandalones
     ```
3. Then check that conatiners are running as already discussed
4. Bring them down again as already discussed

## [Debugging Lambda function directly](https://github.com/binarymist/aws-sam-local#debugging-applications)

1. Run docker-compose-ui as already discussed
2. Then simply run one of the following commands:  
   * For `provisionAppEmissaries`, from the `purpleteam-lambda/` root directory, run the following command:  
     ```shell
     echo '<same-JSON-payload-as-above>' | sam local invoke --debug-port 5858 --env-vars local/env.json --event - --docker-network compose_pt-net provisionAppEmissaries
     ```
   * For `provisionSeleniumStandalones`, from the `purpleteam-lambda/` root directory, run the following command:  
     ```shell
     echo '<same-JSON-payload-as-above>' | sam local invoke --debug-port 5858 --env-vars local/env.json --event - --docker-network compose_pt-net provisionSeleniumStandalones
     ```
3. [Debug](#debugging) in VS Code
4. Then check that conatiners are running as already discussed
5. Bring them down again as already discussed

# Debugging

## Back End

### Lambda `app-emissary-provisioner`

In VS Code:

We used a similar launch.json as mentioned [here](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-using-debugging-nodejs.html)

Open the Lambda handler and put a break point where you want it to stop.  
The VS Code instance needs to be restarted after any changes to the launch.json.

1. Terminal 1: Run docker-compose-ui so that the Lambda functions are able to request that Stage Two containers be brought up and down
2. Terminal 2: Run the following [`sam local start-lambda`](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-local-start-lambda.html) command from the `purpleteam-lambda/` root directory:  
   ```shell
   sam local start-lambda --host 172.25.0.1 --env-vars local/env.json --docker-network compose_pt-net --debug-port 5858
   ```
3. Terminal 3: Now run the `aws lambda invoke` command as discussed previously from the `purpleteam-lambda/` root directory:  
   ```shell
    aws lambda invoke --function-name "provisionAppEmissaries" --endpoint-url "http://172.25.0.1:3001" --no-verify-ssl --payload '<same-JSON-payload-as-above>' local/app-emissary-provisioner/out.txt
   ```  
   
   Just be aware that if the stage two containers have not yet been fetched that it will take some time to do so
   
4. In VS Code:  
  Click on the `app-emissary-provisioner` folder, switch to debug (Run) view, click the "app-emissary-provisioner (purpleteam-lambda)" project in the drop-down, click debug arrow

### `app-scanner` and sub-processes

Using VS Code, we have created a launch.json to use for this.

Make sure you have the app-scanner project loaded in VS Code.

1. Terminal 1: Run docker-compose-ui so that the Lambda functions are able to request that Stage Two containers be brought up and down
2. Terminal 2: Run the `sam local start-lambda` command as detailed in the [Full system run](#full-system-run)
3. Make sure you have your SUT running in a clean state unless you specifically want otherwise, and your Build User config (Job) has it's contact details for purpleteam CLI to send to the orchestrator
4. Terminal 3: Build the orchestrator and app-scanner (app-scanner for debug), this step only needs to be done for the first app-scanner debug session and after that if you make any code changes. From the `purpleteam-orchestrator/` root directory:  
   ```shell
   npm run dc-build-debug-app
   ```
5. Terminal 3: Bring up the orchestrator and app-scanner with the following command from the same root directory:  
   ```shell
   npm run dc-up-debug-app
   ```
6. VS Code: switch to debug (Run) view, click on the debug drop-down and select "Docker: Attach to Node (purpleteam-app-scanner)" and click the Run button. You should notice that the terminal running the orchestrator and app-scanner will inform you "Debugger attached", and VS Code should have broken on the first line of code in the app-scanner. At this point you are ready to step through the app-scanner and pause on any break-points you may add
7. In order to reach the bulk of the app-scanner code, you will need to start the purpleteam CLI. If you want to step through the Test Session sub-processes, it's often useful to put a break-point in app.parallel.js on the line that logs which PID has been assigned to which test session (at the time of writing this, this is in the `runTestSession` routine). This is useful information for taking through the rest of your session and correlating the app-scanner logs
8. Now you can debug into each Cucumber sub-process, currently we have two defined in the launch.json file (you can add more if you need). Click on the debug drop-down again and select "Docker: Attach to child_process [n] (purpleteam-app-scanner)" and click the Run button. VS Code should break on the first line of code in one of the Cucumber processes. To check which process you are in, just execute the `process.id` command in the VS Code Debug Console, this will give you a PID, which you can correlate with the PID you captured from the app-scanner log in the previous step, along with the test session Id. From here if you want to debug the other test session, just click on it in the Call Stack pane, and check the `process.id` again. This way you can always know which test session you are debugging.

## Front End

Discussed [here](https://glebbahmutov.com/blog/debugging-mocha-using-inspector/)

### [CLI](https://github.com/purpleteam-labs/purpleteam)

Assuming you have set the relevant environment variables [detailed](https://github.com/purpleteam-labs/purpleteam) for the CLI, from the `purpleteam` root directory you can run the following command:  
```shell
npm run debug
```  
Or if you need to override the `NODE_ENV` environment variable to `local` then run:  
```shell
NODE_ENV=local npm run debug
```  
Or if you actually want to exercise say the `test` command, run the following command:  
```shell
NODE_ENV=local npm run debug test
```   
In Chromium, open `chrome://inspect` and click "inspect", which will drop you into the first loaded script.

### Tests

From the `purpleteam` root directory you can run the following command:  
```shell
npm run test:debug
```  
Or if you need to override the `NODE_ENV` environment variable to `local` then run:  
```shell
NODE_ENV=local npm run test:debug
```  
In Chromium, open `chrome://inspect` and click "inspect", which will drop you into the first loaded script.

Also review the run options detailed in the [CLI](https://github.com/purpleteam-labs/purpleteam#run).

# Full system run

Leaving `docker stats` running in a terminal is often useful to see which containers are running and how much work they're doing.

`docker container ls` is also quite useful for watching port allocations.

1. [docker-compose-ui](https://github.com/francescou/docker-compose-ui) needs to be running. Running it in it's own terminal is a good idea to see what is happening.  
   
   You need to use the user-defined network already created in order for Lambda function in (what was previously [docker-lambda](https://github.com/lambci/docker-lambda)) the [AWS managed Docker Image](https://aws.amazon.com/blogs/compute/the-aws-serverless-application-model-cli-is-now-generally-available/) to be able to send requests to `docker-compose-ui`. From the `purpleteam-s2-containers/` root directory, run the following command:  
   ```shell
   docker run --name docker-compose-ui -v $(pwd):$(pwd) -w $(dirname $(pwd)) -p 5000:5000 --rm --network compose_pt-net -v /var/run/docker.sock:/var/run/docker.sock francescou/docker-compose-ui:1.13.0
   ```  
   
   Additional resources:
   * docker compose ui [API](https://francescou.github.io/docker-compose-ui/api.html)
   * Once running, `http://localhost:5000/api/v1/projects` will list your projects
   * The following command will start two containers defined by the `chrome` service in the `purpleteam-s2-containers/selenium-standalone/docker-compose.yml` file:  
     ```shell
     curl -X PUT http://localhost:5000/api/v1/services --data '{"service":"chrome","project":"selenium-standalone","num":"2"}' -H 'Content-type: application/json'
     ```
     * Verify with either `docker stats` or `docker container ls`
     * Then bring the `purpleteam-s2-containers/selenium-standalone` `docker-compose.yml` down
<!---->
2. Host Lambda functions:  
   
   From the `purpleteam-lambda/` root directory run the following [`sam local start-lambda`](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-local-start-lambda.html) command. Running it in it's own terminal is a good idea to see what is happening:  
   ```shell
   sam local start-lambda --host 172.25.0.1 --env-vars local/env.json --docker-network compose_pt-net
   ```  
   The `--host [gateway IP address of compose_pt-net]` is required to bind sam local to the user-defined bridge network `compose_pt-net` in order for it to be reachable from the app-scanner container.  
   The following are the links that were useful for working this out: [`host.docker.internal`, `extra_hosts` and other comments from here down](https://github.com/docker/for-linux/issues/264#issuecomment-410048049), [docker-host container](https://github.com/qoomon/docker-host/blob/master/entrypoint.sh), [`extra_hosts` reference](https://docs.docker.com/compose/compose-file/#extra_hosts), along with creating the firewall rule as mentioned above, and testing connectivity as mentioned in the [Docker](https://github.com/purpleteam-labs/purpleteam-doc/blob/main/local/local-Docker.md#docker-cli) page with shelling into running container

3. <div id="start_your_sut"></div> Start your SUT (NodeGoat in this example). Running it in it's own terminal is a good idea to see what is happening:  
   
   There are at least two options as mentioned in the [set-up](https://github.com/purpleteam-labs/purpleteam-doc/blob/main/local/local-setup.md#your-system-under-test-sut)
<!---->
4. Run the docker-compose to bring the stage one containers up. Running the following commands for this step in their own terminal is a good idea to see what is happening:   
   
   * Standard:
     1. To build/rebuild images after code changes, from the `purpleteam-orchestrator/` root directory, run the following command:  
        ```shell
        npm run dc-build
        ```
     2. From the `purpleteam-orchestrator/` root directory, run the following command:  
        ```shell
        npm run dc-up
        ```
   * Debug (orchestrator or app-scanner. For this example we demo orchestrator, for app-scanner just swap the `orchestrator` with `app`):
     1. To build/rebuild images after code changes, from the `purpleteam-orchestrator/` root directory, run the following command:
        ```shell
        npm run dc-build-debug-orchestrator
        ```
     2. From the `purpleteam-orchestrator/` root directory, run the following command:  
        ```shell
        npm run dc-up-debug-orchestrator
        ```  
        * Now you can attach to the `purpleteam-orchestrator`, or `purpleteam-app-scanner` process within the container. Further details in the [Debugging](#debugging) section
<!---->
5. Start cli:  
   
   If you are running in the `local` environment and this is the first time you are doing this on a given machine, beware that the stage two images will take some time to fetch. The terminal that you have run docker-compose-ui in will be visibly retrieving these images. There are some things that can go wrong:  
   
   * The app-scanner may timeout while testing connectivity of the yet to be running stage two containers. If this happens a timeout `error` message will be logged in the terminal you ran `npm run dc-up` in
   * We have also seen a `Missing region in config` `Error` logged. If you have [configured the aws cli](https://github.com/purpleteam-labs/purpleteam-lambda) correctly then this is a red herring   
   
   On subsequent runs you should not see the above mentioned issues. If this  concerns you, the timeout (found in the app-scanner) can be increased.  
   
   Depending on whether you are testing against a local copy of your containerised app or a copy hosted on the Internet will determine how you [configure the cli](https://github.com/purpleteam-labs/purpleteam#configure) (purpleteam). This example demonstrates the two options mentioned in [step 3](#start_your_sut).  
   By specifying the `local` environment, you are instructing the purpleteam CLI to use it's `config/config.local.json`, and communicate with the purpleteam back-end that you have already set-up and have running locally (step 4). If using the `cloud`, the back-end is all taken care of for you. You will also need to specify the location of the SUT in the Build User config (Job) that you provide to the CLI. Examples of these can be found in the [`testResources/jobs`](https://github.com/purpleteam-labs/purpleteam/tree/main/testResources/jobs) directory:  
   1. Locally cloned copy of NodeGoat:  
   The SUT details in your Job will be as follows:  
    
      ```json
      "sutIp": "pt-sut-cont",
      "sutPort": 4000,
      "sutProtocol": "http",
      ```  
   2. NodeGoat running on the Internet via [purpleteam-iac-sut](https://github.com/purpleteam-labs/purpleteam-iac-sut):  
   The SUT details in your Job will be as follows, with `<your-domain-name.com>` replaced with a domain you have set-up:  
   
      ```json
      "sutIp": "nodegoat.sut.<your-domain-name.com>",
      "sutPort": 443,
      "sutProtocol": "https",
      ```
    
    Assuming you have set the relevant environment variables [detailed](https://github.com/purpleteam-labs/purpleteam) for the CLI, from the purpleteam CLI root directory, run the following command:  
    
     ```shell
     npm start test
     ```
6. Once the test run has finished, you can check to make sure the Stage Two containers have been brought down. If you are not using `docker stats`:  
   * In one terminal run the following command:  
     ```shell
     docker container ls
     ```
     If they haven't been brought down:  
     * In another terminal from the `purpleteam-s2-containers/app-emissary/` directory run the following command:  
       ```shell
       docker-compose down
       ```
     * In another terminal from the `purpleteam-s2-containers/selenium-standalone/` directory run the following command:  
       ```shell
       docker-compose down
       ```

   
   If you want to keep the Stage Two containers running after a test run to inspect for any reason, simply change the app-scanner's `emissary.shutdownEmissariesAfterTest` config value to `false`, rebuild the container and run.  
   
   
   If the Stage Two containers have not been brought down, you can also shut down the Stage Two containers with the docker-compose-ui UI. When it's running, you can browse to http://localhost:5000, select the specific Stage Two container project and click the Down button, this is convenient as it brings all of the specific Stage Two containers down with one click.

## Useful resources

* Dockerising all of your components
  * [How I do local Docker development for my AWS Fargate application
](https://medium.com/containers-on-aws/how-i-do-local-docker-development-for-my-aws-fargate-application-8957e3fdb50)
  * [How to Debug a Node.js app in a Docker Container](https://blog.risingstack.com/how-to-debug-a-node-js-app-in-a-docker-container/)
