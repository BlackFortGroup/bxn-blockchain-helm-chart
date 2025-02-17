## Table of Contents
1. [General Information](#general-information)
2. [Deployment Process](#deployment-process)
   - [Running the Bootstrap Script](#running-the-bootstrap-script)
   - [Configuration Management](#configuration-management)
   - [Service Endpoints](#service-endpoints)
   - [Genesis File](#genesis-file)
   - [Validator Keys](#validator-keys)
3. [Using a Custom Genesis File](#using-a-custom-genesis-file)
4. [Using Custom Validator Keys](#using-custom-validator-keys)
5. [Important Notes](#important)
6. [Manual Deployment (Testnet Example)](#manual-deploy-testnet-example)

### Prerequisites
Before proceeding with the deployment, ensure the following:
- **Helm** is installed and configured. Install it using the official guide: [Helm Installation](https://helm.sh/docs/intro/install/).
- **kubectl** is installed and configured to access the target Kubernetes cluster. Install it using the official guide: [kubectl Installation](https://kubernetes.io/docs/tasks/tools/).
- **AWS CLI** is installed and configured. Install it using the official guide: [AWS CLI Installation](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).
- **GPG** is installed for decrypting validator keys (if using custom keys). Install it using the official guide: [GPG Installation](https://gnupg.org/download/).

### General information

Chain ID `testnet` **4888**

Chain ID `mainnet` **488**

#### Main API URLs

https://rpc.blackfort.network/mainnet/rpc - node reader `mainnet`

https://rpc.blackfort.network/mainnet/validator-1 - validator `mainnet`

https://rpc.blackfort.network/testnet/rpc - node reader `testnet`

https://rpc.blackfort.network/testnet/validator-1 - validator `testnet`

### Deployment Process

The deployment is performed via GitHub Actions from the `testnet` and `mainnet` branches for their respective environments.

The deployment utilizes a **HELM chart**.

Templates for the `testnet` and `mainnet` environments are identical and located at the following paths:
- `./helm/charts/besu-genesis` (used for initializing the genesis)
- `./helm/charts/besu-node` (used for the `validator`, `txnode`, `reader`, and `bootnode` services)

#### Running the Bootstrap Script
Before deploying the network, you must run the bootstrap script to set up the necessary AWS resources and configure the EKS cluster.
The bootstrap script is located at ./aws/scripts/bootstrap.sh. It requires the following arguments:

* Region: The AWS region where the EKS cluster is located.
* Account ID: The AWS account ID where the EKS cluster is deployed.
* EKS Cluster Name: The name of the EKS cluster.
* Namespace: The Kubernetes namespace where the resources will be deployed.

   ```bash
   ./aws/scripts/bootstrap.sh "eu-central-1" "123456789012" "k8s-blockchain" "besu-testnet"
   ```
This script will:

* Set up the necessary IAM roles and policies.
* Prepare the Kubernetes namespace for deployment.

#### Configuration Management
Configuration of the HELM charts is managed using variable files, which are located at:
- `./helm/values/genesis-besu.yml` for deploy genesis 
- `./helm/values/bootnode.yml` for deploy boot nodes
- `./helm/values/validator.yml` for deploy validators nodes
- `./helm/values/txnode.yml` for deploy members nodes
Variable files for `mainnet`
- `./helm/mainnet-values/genesis-besu.yml` for deploy genesis
- `./helm/mainnet-values/bootnode.yml` for deploy boot nodes
- `./helm/mainnet-values/validator.yml` for deploy validators nodes
- `./helm/mainnet-values/txnode.yml` for deploy members nodes

#### Service Endpoints
Service endpoints are configured via **Ingress**, with its configuration located at:
- `./ingress/ingress-rules-besu-testnet.yml` for testnet
- `./ingress/ingress-rules-besu-mainnet.yml` for mainnet

#### Genesis File
Encrypted genesis files for `testnet` and `mainnet`
- `./helm/genesis-testnet.json`
- `./helm/genesis-mainnet.json`

#### Validator Keys
Encrypted validator keys files for `testnet` and `mainnet`
- `./helm/validator-key-testnet.yaml.gpg`
- `./helm/validator-key-mainnet.yaml.gpg`

### Using a Custom Genesis File
To deploy the network with a custom `genesis.json` file, follow these steps:
1. Prepare the Custom Genesis File:
   * Ensure your custom `genesis.json` file is ready and located at the appropriate path ./helm/charts/besu-genesis/.
2. Update the `genesis-besu.yml` Configuration:
   * Modify the `genesis-besu.yml` file to include the `genesisLocalFile` parameter, pointing to your custom `genesis.json` file.
   ```yaml
   genesisLocalFile: genesis.json
   ```

### Using Custom Validator Keys
To deploy the network with custom validator keys, follow these steps:
1. Prepare the Encrypted Validator Keys
   * Encrypted validator keys for testnet and mainnet are located at:
   
     `./helm/validator-key-testnet.yaml.gpg` (for testnet)

     `./helm/validator-key-mainnet.yaml.gpg` (for mainnet)

     These files need to be decrypted before use.
2. Decrypt the Validator Keys
   * Decrypt the validator keys using the appropriate decryption tool (e.g., gpg). For example:
   ```bash
   gpg --decrypt ./helm/validator-key-testnet.yaml.gpg > ./helm/validator-key-testnet.yaml
   gpg --decrypt ./helm/validator-key-mainnet.yaml.gpg > ./helm/validator-key-mainnet.yaml
   ```
Example Validator Key Configuration
   ```yaml
   key_validators:
   - name: validator0
     args:
        - "ADDRESS"      # Validator address
        - "NODEKEY"     # Validator node key
        - "NODEKEYPUB"  # Validator node public key
   - name: validator1
     args:
        - "ADDRESS"
        - "NODEKEY"
        - "NODEKEYPUB"
   # Add more validators as needed
   ```
3. Deploy the Genesis with Custom Validator Keys
   * Use the `helm upgrade --install` command to deploy the genesis block while including the custom validator keys. For example, for `testnet`:
   ```bash
   helm upgrade --install genesis ./helm/charts/besu-genesis \
   --namespace besu-testnet \
   --create-namespace \
   --values ./helm/values/genesis-besu.yml \
   --values ./helm/validator-key-testnet.yaml
   ```

#### IMPORTANT
If needed to increase the number of any services (e.g., the number of `validator` instances), the services must be added in the corresponding workflow files:
- `./.github/workflows/deploy-mainnet.yml` for the `mainnet` environment
- `./.github/workflows/deploy-testnet.yml` for the `testnet` environment

#### MANUAL DEPLOY (TESTNET EXAMPLE)

First we need generate genesis:

`helm upgrade -i genesis ./helm/charts/besu-genesis -n besu-testnet --create-namespace -f ./helm/values/genesis-besu.yml`

After we can deploy all services:

boot nodes:

`helm upgrade -i bootnode-1 ./helm/charts/besu-node -n besu-testnet --create-namespace -f ./helm/values/bootnode.yml`

`helm upgrade -i bootnode-2 ./helm/charts/besu-node -n besu-testnet --create-namespace -f ./helm/values/bootnode.yml`

Validators:

`helm upgrade -i validator-1 ./helm/charts/besu-node -n besu-testnet --create-namespace -f ./helm/values/validator.yml`

`helm upgrade -i validator-2 ./helm/charts/besu-node -n besu-testnet --create-namespace -f ./helm/values/validator.yml`

`helm upgrade -i validator-3 ./helm/charts/besu-node -n besu-testnet --create-namespace -f ./helm/values/validator.yml`

`helm upgrade -i validator-4 ./helm/charts/besu-node -n besu-testnet --create-namespace -f ./helm/values/validator.yml`

Members:

`helm upgrade -i member-1 ./helm/charts/besu-node -n besu-testnet --create-namespace -f ./helm/values/txnode.yml`

RPC:

`helm upgrade -i rpc-1 ./helm/charts/besu-node -n besu-testnet --create-namespace -f ./helm/values/reader.yml`

Ingress:

`kubectl apply -f ./ingress/ingress-rules-besu.yml -n besu-testnet`

For more detailed capabilities and examples, refer to the official repository:  

[ConsenSys Quorum Kubernetes](https://github.com/ConsenSys/quorum-kubernetes)