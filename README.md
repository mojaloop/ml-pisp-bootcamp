# ml-pisp-bootcamp

- https://mojaloop.io/events/mojaloop-bootcamp-for-pisps/
- https://dfslab.net/mojaloop-bootcamp-for-pisps/

## Team

- Paul Baker - @PaulGregoryBaker / paul.baker@modusbox.com
- Miguel de Barros - @mdebarros / miguel.debarros@modusbox.com
- Vijaya Guthi - @vijayg10 / vijaya.guthi@modusbox.com
- Sam Kummary - @elnyry-sam-k	/ sam@modusbox.com
- Kevin Leyow - @kleyow / kevin.leyow@modusbox.com
- Michael Richards - @MichaelJBRichards / Michael.Richards@modusbox.com

_* Ordered by Surname_

## Resources

1. Demo Presentation: https://docs.google.com/presentation/d/1Lfq3W724fqJx6iOA-CFRMDYWKdmvb0aVhRosaM4Dr5Q/edit?usp=sharing
2. Workshop Design Sprint Workspace: https://docs.google.com/presentation/d/1FUrfjR7n_y9xaOTTBVKYCXZr4sYnox2qGzjQ_NybnKM/edit?usp=sharing
3. Sequence Diagram: https://github.com/mojaloop/ml-pisp-bootcamp/blob/main/sequence-diagrams/p2p.md
4. Wireframe Prototypes: https://drive.google.com/file/d/1Wixa8JdLInXRDZm4vVVk2d1UOQ6K8FC-/view?usp=sharing

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

The [GSP-Connector](https://github.com/mojaloop/gsp-connector) has been added a sub-module.

You can access the [GSP-Connector](https://github.com/mojaloop/gsp-connector) repo here: https://github.com/mojaloop/gsp-connector

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
