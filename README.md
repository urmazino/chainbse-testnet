1. Install Dependecies

# Update & Install Packages
sudo apt update & sudo apt upgrade -y
sudo apt install ca-certificates zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev curl git wget make jq build-essential pkg-config lsb-release libssl-dev libreadline-dev libffi-dev gcc screen unzip lz4 -y

# Install Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
docker version

# Install Docker-Compose
VER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d '"' -f 4)

curl -L "https://github.com/docker/compose/releases/download/"$VER"/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
docker-compose --version

# Docker Permission to user
sudo groupadd docker
sudo usermod -aG docker $USER

# Install Go
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.22.4.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> $HOME/.bash_profile
source .bash_profile
go version

2. Install EigenLayer CLI

curl -sSfL https://raw.githubusercontent.com/layr-labs/eigenlayer-cli/master/scripts/install.sh | sh -s

export PATH=$PATH:~/bin

eigenlayer --version

3. Clone Chainbase AVS repo

git clone https://github.com/chainbase-labs/chainbase-avs-setup

cd chainbase-avs-setup/holesky

4. Create Eigenlayer wallet
Create a new Key

1- Create operator address

eigenlayer operator keys create --key-type ecdsa opr

2- Enter a password and press Enter

    Best symbol for password is * , you can choose a password like this: ***123ABCabc123***

3- Save your wallet private key

image

4- Ctrl+C+Enter+Enter & save your wallet info

image
Optional: Import an old key

    Replace PRIVATEKEY

eigenlayer operator keys import --key-type ecdsa opr PRIVATEKEY

    Enter a password and press Enter

    Best symbol for password is * , you can choose a password like this: ***123ABCabc123***

Fund your Eigen wallet

You’ll need at least 1 Holesky ETH to cover the gas cost of the operator registration. Make sure to send at least 1 ETH to your Eigenlayer operator’s address
5. Config & register operator

eigenlayer operator config create

    operator address: Your Eigenlayer ETH address
    earnings address: press Enter
    ETH rpc url: https://ethereum-holesky-rpc.publicnode.com
    network: holesky
    signer type: local_keystore
    ecdsa key path:: /root/.eigenlayer/operator_keys/opr.ecdsa.key.json

6. Edit metadata.json

1- Open metadata.json edit menu

nano metadata.json

2- Complete your details in metadata.json

    logo: Must be raw link to your operator favorite logo
    Logo only supports .png file less than 1MB in size
    Create a repositry in github and upload .png there
    To get your logo raw link: 1) Navigate to the file in your repository, 2) Click on the file name to view its contents, 3) In the address bar of your browser, 4) replace blob with raw in the URL, 5) Press Enter to load the raw content view, 6) Copy the updated URL from the address bar and paste in front of logo field in metadata.json
    My example metadata.json file in github: https://github.com/0xmoei/chainbase-testnet/blob/main/metadata.json
    Ctrl+X+Y+Enter to save metadata.json

3- Upload metadata.json to your github repository and get its raw link like previous step

    You can create a metadata.json file in github with the contents of your file or you must download it from VPS using Termius or Mobaxterm ssh clients

4- Open operator.yaml edit menu

nano operator.yaml

5- Add your metadata.json raw url in github in front of metadata-url

Screenshot_158
7. Run Eigenlayer holesky Node

eigenlayer operator register operator.yaml

# Check status
eigenlayer operator status operator.yaml

image
8. Config Chainbase AVS

1- Create .env file

# Delete old files
rm -rf .env

# Open Edit menu
nano .env

2- Paste below codes in it

    NODE_ECDSA_KEY_PASSWORD: Replace ***123ABCabc123*** with your Eigenlayer password
    Optional: You can Change #TODO lines if needed but it should be okay by default

# Chainbase AVS Image
MAIN_SERVICE_IMAGE=repository.chainbase.com/network/chainbase-node:testnet-v0.1.7
FLINK_TASKMANAGER_IMAGE=flink:latest
FLINK_JOBMANAGER_IMAGE=flink:latest
PROMETHEUS_IMAGE=prom/prometheus:latest

MAIN_SERVICE_NAME=chainbase-node
FLINK_TASKMANAGER_NAME=flink-taskmanager
FLINK_JOBMANAGER_NAME=flink-jobmanager
PROMETHEUS_NAME=prometheus

# FLINK CONFIG
FLINK_CONNECT_ADDRESS=flink-jobmanager
FLINK_JOBMANAGER_PORT=8081
NODE_PROMETHEUS_PORT=9091
PROMETHEUS_CONFIG_PATH=./prometheus.yml

# Chainbase AVS mounted locations
NODE_APP_PORT=8080
NODE_ECDSA_KEY_FILE=/app/operator_keys/ecdsa_key.json
NODE_LOG_DIR=/app/logs

# Node logs configs
NODE_LOG_LEVEL=debug
NODE_LOG_FORMAT=text

# Metrics specific configs
NODE_ENABLE_METRICS=true
NODE_METRICS_PORT=9092

# holesky smart contracts
AVS_CONTRACT_ADDRESS=0x5E78eFF26480A75E06cCdABe88Eb522D4D8e1C9d
AVS_DIR_CONTRACT_ADDRESS=0x055733000064333CaDDbC92763c58BF0192fFeBf

###############################################################################
####### TODO: Operators please update below values for your node ##############
###############################################################################
# TODO: Operators need to point this to a working chain rpc
NODE_CHAIN_RPC=https://rpc.ankr.com/eth_holesky
NODE_CHAIN_ID=17000

# TODO: Operators need to update this to their own paths
USER_HOME=$HOME
EIGENLAYER_HOME=${USER_HOME}/.eigenlayer
CHAINBASE_AVS_HOME=${EIGENLAYER_HOME}/chainbase/holesky

NODE_LOG_PATH_HOST=${CHAINBASE_AVS_HOME}/logs

# TODO: Operators need to update this to their own keys
NODE_ECDSA_KEY_FILE_HOST=${EIGENLAYER_HOME}/operator_keys/opr.ecdsa.key.json

# TODO: Operators need to add password to decrypt the above keys
# If you have some special characters in password, make sure to use single quotes
NODE_ECDSA_KEY_PASSWORD=***123ABCabc123***

3- Create docker-compose.yml file

# Remove old file
rm -rf docker-compose.yml

# Open edit menu
nano docker-compose.yml

4- Paste below codes in it and save with CTRL+X+Y+ENTER

    You can change ports if any of them are in use: 8081, 9091, 8080, 9092
    If you want to change 8081 to 35081 then just change the port on the left side like: 35081:8081

services:
  prometheus:
    image: ${PROMETHEUS_IMAGE}
    container_name: ${PROMETHEUS_NAME}
    env_file:
      - .env
    volumes:
      - "${PROMETHEUS_CONFIG_PATH}:/etc/prometheus/prometheus.yml"
    command: 
      - "--enable-feature=expand-external-labels"
      - "--config.file=/etc/prometheus/prometheus.yml"
    ports:
      - "9091:9090"
    networks:
      - chainbase
    restart: unless-stopped

  flink-jobmanager:
    image: ${FLINK_JOBMANAGER_IMAGE}
    container_name: ${FLINK_JOBMANAGER_NAME}
    env_file:
      - .env
    ports:
      - "8081:8081"
    command: jobmanager
    networks:
      - chainbase
    restart: unless-stopped

  flink-taskmanager:
    image: ${FLINK_JOBMANAGER_IMAGE}
    container_name: ${FLINK_TASKMANAGER_NAME}
    env_file:
      - .env
    depends_on:
      - flink-jobmanager
    command: taskmanager
    networks:
      - chainbase
    restart: unless-stopped

  chainbase-node:
    image: ${MAIN_SERVICE_IMAGE}
    container_name: ${MAIN_SERVICE_NAME}
    command: ["run"]
    env_file:
      - .env
    ports:
      - "8080:8080"
      - "9092:9092"
    volumes:
      - "${NODE_ECDSA_KEY_FILE_HOST:-./opr.ecdsa.key.json}:${NODE_ECDSA_KEY_FILE}"
      - "${NODE_LOG_PATH_HOST}:${NODE_LOG_DIR}:rw"
    depends_on:
      - prometheus
      - flink-jobmanager
      - flink-taskmanager
    networks:
      - chainbase
    restart: unless-stopped

networks:
  chainbase:
    driver: bridge

5- Create folders for docker

source .env && mkdir -pv ${EIGENLAYER_HOME} ${CHAINBASE_AVS_HOME} ${NODE_LOG_PATH_HOST}

6- Give permissions to bash script

chmod +x ./chainbase-avs.sh

7- update prometheus.yml

    Replace ${YOUR_OPERATOR_NAME} with your operator address

nano prometheus.yml

Screenshot_160
9. Run Chainbase AVS

1- Register AVS

./chainbase-avs.sh register

image

2- Run AVS

./chainbase-avs.sh run

image
10. Check AVS health

Check chainbase-node logs

docker compose logs chainbase-node -f

image

Get your AVS link

export PATH=$PATH:~/bin

eigenlayer operator status operator.yaml

image

Check Operator Health

# If your port is 8080
curl -i localhost:8080/eigen/node/health

image

Check docker containers

    You must have 4 new docker containers

Docker PS

image
11. Fill in the form

https://forms.gle/w9h8Su87kEnDwRMA7
12. Post your Operator address in discord

https://discord.gg/chainbase

Screenshot_159
