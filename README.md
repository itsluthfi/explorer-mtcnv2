# Quick start (using Docker)

## Prerequisites

* Docker
* Docker Compose
  * **Note(for v2.0.0 and above):**
    The following docker images are automatically pulled from **GHCR instead of Docker Hub** when starting docker-compose.

    * [Hyperledger Explorer ghcr repository](https://github.com/hyperledger-labs/blockchain-explorer/pkgs/container/explorer)
    * [Hyperledger Explorer PostgreSQL ghcr repository](https://github.com/hyperledger-labs/blockchain-explorer/pkgs/container/explorer-db)

  * **Note(for v1.1.8 and below):**
    The following docker images are automatically pulled from Docker Hub when starting docker-compose.

    * [Hyperledger Explorer docker repository](https://hub.docker.com/r/hyperledger/explorer/)
    * [Hyperledger Explorer PostgreSQL docker repository](https://hub.docker.com/r/hyperledger/explorer-db)

## Start Hyperledger Fabric network

This guide assumes that you've already started the test network by following [Hyperledger Fabric official tutorial](https://hyperledger-fabric.readthedocs.io/en/latest/test_network.html).

## Configure

* Create a new directory (e.g. `explorer`)

    ```bash
    mkdir explorer
    cd explorer
    ```

* Copy the following files from the repository

  - [.env](https://github.com/hyperledger/blockchain-explorer/blob/main/.env)
  - [docker-compose.yaml](https://github.com/hyperledger/blockchain-explorer/blob/main/docker-compose.yaml)
  - [examples/net1/connection-profile/test-network.json](https://github.com/hyperledger/blockchain-explorer/blob/main/examples/net1/connection-profile/test-network.json)
  - [examples/net1/config.json](https://github.com/hyperledger/blockchain-explorer/blob/main/examples/net1/config.json)


  ```bash
  wget https://raw.githubusercontent.com/hyperledger/blockchain-explorer/main/examples/net1/config.json
  wget https://raw.githubusercontent.com/hyperledger/blockchain-explorer/main/examples/net1/connection-profile/test-network.json -P connection-profile
  wget https://raw.githubusercontent.com/hyperledger/blockchain-explorer/main/docker-compose.yaml
  ```

* Copy entire crypto artifact directory (organizations/) from your fabric network (e.g /fabric-samples/test-network)

    ```bash
    cp -r ../fabric-samples/test-network/organizations/ .
    ```

* Now, you should have the following files and directory structure.

    ```
    docker-compose.yaml
    config.json
    connection-profile/test-network.json
    organizations/ordererOrganizations/
    organizations/peerOrganizations/
    ```

* Edit environmental variables in `docker-compose.yaml` to align with your environment

    ```yaml
        networks:
        mynetwork.com:
            external:
                name: fabric_test

        ...

        services:
          explorer.mynetwork.com:

            ...

            volumes:
              - ./config.json:/opt/explorer/app/platform/fabric/config.json
              - ./connection-profile:/opt/explorer/app/platform/fabric/connection-profile
              - ./organizations:/tmp/crypto
              - walletstore:/opt/explorer/wallet
    ```

    An alternative option is to export environment variables in your shell.

    ```bash
    export EXPLORER_CONFIG_FILE_PATH=./config.json
    export EXPLORER_PROFILE_DIR_PATH=./connection-profile
    export FABRIC_CRYPTO_PATH=./organizations
    ```

* When you connect Explorer to your fabric network through the bridge network, you need to set `DISCOVERY_AS_LOCALHOST` to `false` for disabling hostname mapping into localhost.

    ```yaml
    services:

      ...

      explorer.mynetwork.com:

        ...

        environment:
          - DISCOVERY_AS_LOCALHOST=false
    ```

* Replace the user's certificate with an admin certificate and a secret (private) key in the connection profile (test-network.json). You need to specify the absolute path on the Explorer container.

    Before:
    ```json
    "adminPrivateKey": {
        "path": "/tmp/crypto/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/keystore/priv_sk"
    }
    ```

    After:
    ```json
    "adminPrivateKey": {
        "path": "/tmp/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/priv_sk"
    }
    ```
    **Make sure you replace all paths.**

## Start container services

* Run the following to start up explore and explorer-db services after starting your fabric network:

    ```shell
    $ docker-compose up -d
    ```

## Clean up

* To stop services without removing persistent data, run the following:

    ```shell
    $ docker-compose down
    ```

* In the docker-compose.yaml, two named volumes are allocated for persistent data (for Postgres data and user wallet). If you would like to clear these named volumes up, run the following:

    ```shell
    $ docker-compose down -v
    ```
