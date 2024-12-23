### Deployment Process

The deployment is performed via GitHub Actions from the `testnet` and `mainnet` branches for their respective environments.

The deployment utilizes a **HELM chart**.

Templates for the `testnet` and `mainnet` environments are identical and located at the following paths:
- `./helm/charts/besu-genesis` (used for initializing the genesis)
- `./helm/charts/besu-node` (used for the `validator`, `txnode`, `reader`, and `bootnode` services)

#### Configuration Management
Configuration of the HELM charts is managed using variable files, which are located at:
- `./helm/values/genesis-besu.yml` for deploy genesis 
- `./helm/values/bootnode.yml` for deploy boot nodes
- `./helm/values/validator.yml` for deploy validators nodes
- `./helm/values/txnode.yml` for deploy members nodes

#### Service Endpoints
Service endpoints are configured via **Ingress**, with its configuration located at:
- `./ingress/ingress-rules-besu.yml`

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