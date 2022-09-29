# ml-pisp-bootcamp
https://dfslab.net/mojaloop-bootcamp-for-pisps/


## WireFrame Prototypes

https://drive.google.com/file/d/1Wixa8JdLInXRDZm4vVVk2d1UOQ6K8FC-/view?usp=sharing

## Exploring GSP API using TTK
```
docker-compose up
```
3. Open TTK Sender UI on http://localhost:16060
4. Goto `Test Runner` and open `Collection Managaer`
5. Click on `Import File` and load the TTK test case `gsp.json` from the folder `test-cases`
6. Select the imported file in the collection manager
7. You can explore the test cases there
8. You can explore the GSP API in `API Manager`

## TTK Demo UI

The code is at the following link
https://github.com/mojaloop/ml-testing-toolkit-ui/tree/feature/ml-pisp-bootcamp-demo

## GSP-Connector (Git Sub-module)

This is a git-submodule to this repo.

To initialize sub-modules run the following command:

```bash
git submodule init
```

To update the sub-modules run the following command:

```bash
git submodule update
```

## Docker-Compose

### Pre-requisites

Ensure that you have initialized the [GSP-Connector Sub-module](#gsp-connector-git-sub-module), and made sure its updated. See [GSP-Connector Sub-module Section](#gsp-connector-git-sub-module).

### Build and Run

```bash
docker compose build
```

```bash
docker compose up
```
