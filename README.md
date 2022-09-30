# ml-pisp-bootcamp
https://dfslab.net/mojaloop-bootcamp-for-pisps/

## WireFrame Prototypes

https://drive.google.com/file/d/1Wixa8JdLInXRDZm4vVVk2d1UOQ6K8FC-/view?usp=sharing

## Quick Start

To see the demo in action, please follow the below steps
1. Clone the repository
```
git clone https://github.com/mojaloop/ml-pisp-bootcamp.git
cd ml-pisp-bootcamp
```
2. Initialize and update the sub modules
```
git submodule init
git submodule update
```
3. Build the docker images
```
docker compose build
```
4. Run the services
```
docker compose up
```
5. Open TTK Mobile Simulator UI at http://localhost:6060/mobilesimulator
6. Try to make a transfer using the mobile screen
7. Observe the dynamic sequence diagram and activity log after a successful transfer


## Exploring GSP API using TTK

1. Follow the Quick Start guide to start the services
2. Open TTK UI on http://localhost:6060
3. You can explore the GSP API in `API Manager -> API Documentation`


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
