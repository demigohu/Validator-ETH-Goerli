<p align="center" width="100%">
    <img width="33%" src="https://miro.medium.com/max/828/0*sexIRKTeuygGo3ku"> 
</p>

# Guide on how to join Parastate SafeStake Testnet

## Overview

We will use the latest version for Geth and the latest version for Lighthouse.

## Installing Prerequisites

Make sure we have fully updated packages first.

```
sudo apt -y update
sudo apt -y upgrade
```

Install prerequisites commonly available.

```
sudo apt -y install software-properties-common wget curl ccze
```

## Installing Geth

Add the Ethereum PPA and install the Geth package.

```
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt -y install geth
```

## Installing Lighthouse

Download the latest release version for Lighthouse and extract it. If the latest version is more recent than what is used here, use that version and adjust for the new URL and archive name. Make sure to use the linux x86_64 version.

```
cd ~
wget https://github.com/sigp/lighthouse/releases/download/v3.2.1/lighthouse-v3.2.1-x86_64-unknown-linux-gnu.tar.gz
tar xvf lighthouse-v3.2.1-x86_64-unknown-linux-gnu.tar.gz
rm lighthouse-v3.2.1-x86_64-unknown-linux-gnu.tar.gz
```

Install this Lighthouse version globally.

```
sudo cp ~/lighthouse /usr/local/bin
rm ~/lighthouse
```

## Creating the JWT token file

Create a JWT token file in a neutral location and make it readable to everyone. We will use the `/var/lib/ethereum/jwttoken` location to store the JWT token file.

```
sudo mkdir -p /var/lib/ethereum
openssl rand -hex 32 | tr -d "\n" | sudo tee /var/lib/ethereum/jwttoken
sudo chmod +r /var/lib/ethereum/jwttoken
```

## Configuring your Geth node

Create a dedicated user for running Geth, create a directory for holding the data and assign the proper permissions.

```
sudo useradd --no-create-home --shell /bin/false goeth
sudo mkdir -p /var/lib/goethereum
sudo chown -R goeth:goeth /var/lib/goethereum
```

Create a systemd service config file to configure the Geth node service.

```
sudo nano /etc/systemd/system/geth.service
```

Paste the following service configuration into the file. Exit and save once done (`Ctrl` + `X`, `Y`, `Enter`).

```ini
[Unit]
Description=Go Ethereum Client - Geth (Goerli)
After=network.target
Wants=network.target

[Service]
User=goeth
Group=goeth
Type=simple
Restart=always
RestartSec=5
TimeoutStopSec=180
ExecStart=geth \
    --goerli \
    --http \
    --datadir /var/lib/goethereum \
    --metrics \
    --metrics.expensive \
    --pprof \
    --authrpc.jwtsecret=/var/lib/ethereum/jwttoken

[Install]
WantedBy=default.target
```

Reload systemd to reflect the changes and start the service. Check status to make sure it’s running correctly.

```
sudo systemctl daemon-reload
sudo systemctl start geth.service
sudo systemctl status geth.service
```

It should say active (running) in green text. If not then go back and repeat the steps to fix the problem. Press Q to quit (will not affect the geth service).

Enable the geth service to automatically start on reboot.

```
sudo systemctl enable geth.service
```

You can watch the live messages from your Geth node logs using this command. Make sure nothing suspicious shows up in your logs.

```
sudo journalctl -f -u geth.service -o cat | ccze -A
```
![image](https://user-images.githubusercontent.com/46390405/200120237-5ff0651f-c8a1-478c-949c-ad1c3224d6a1.png)

Press `Ctrl` + `C` to stop showing those messages.

## Configuring your Lighthouse beacon node

Create a dedicated user for running the Lighthouse beacon node, create a directory for holding the data, copy testnet files and assign the proper permissions.

```
sudo useradd --no-create-home --shell /bin/false lighthousebeacon
sudo mkdir -p /var/lib/lighthouse
sudo chown -R lighthousebeacon:lighthousebeacon /var/lib/lighthouse
```

Create a systemd service config file to configure the Lighthouse beacon node service.

```
sudo nano /etc/systemd/system/lighthousebeacon.service
```

Paste the following service configuration into the file. Exit and save once done (`Ctrl` + `X`, `Y`, `Enter`).

```ini
[Unit]
Description=Lighthouse Ethereum Client Beacon Node (Prater)
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=lighthousebeacon
Group=lighthousebeacon
Restart=always
RestartSec=5
ExecStart=/usr/local/bin/lighthouse bn \
    --network prater \
    --datadir /var/lib/lighthouse \
    --http \
    --execution-endpoint http://127.0.0.1:8551 \
    --metrics \
    --validator-monitor-auto \
    --checkpoint-sync-url https://goerli.checkpoint-sync.ethdevops.io \
    --execution-jwt /var/lib/ethereum/jwttoken

[Install]
WantedBy=multi-user.target
```

Reload systemd to reflect the changes and start the service. Check status to make sure it’s running correctly.

```
sudo systemctl daemon-reload
sudo systemctl start lighthousebeacon.service
sudo systemctl status lighthousebeacon.service
```

It should say active (running) in green text. If not then go back and repeat the steps to fix the problem. Press Q to quit (will not affect the Lighthouse beacon node service).

Enable the Lighthouse beacon node service to automatically start on reboot.

```
sudo systemctl enable lighthousebeacon.service
```

You can watch the live messages from your Lighthouse beacon node logs using this command. Make sure nothing suspicious shows up in your logs.

```
sudo journalctl -f -u lighthousebeacon.service -o cat | ccze -A
```
![image](https://user-images.githubusercontent.com/46390405/200120282-01417a4a-d151-48e0-9e5c-fc90c9853ab4.png)


Press `Ctrl` + `C` to stop showing those messages.

### Requesting testnet funds
Open link : https://faucetlink.to/goerli

Paste Your Address Metamask

You Need 32 ETH Goerli

### Creating your validator keys and performing the deposit

Create Validator Key With Wagyu Keygen

Download Here [Wagyu Key Gen](https://github.com/stake-house/wagyu-key-gen/releases)

If you choose the *Wagyu Key Gen* application, make sure to select the *Prater* network and follow the instructions provided.

![image](https://user-images.githubusercontent.com/46390405/200476058-2a254f2f-53bf-4185-b6dd-6d5fb0b77d19.png)


Make sure to store your keystore password and your mnemonic somewhere safe. You should end up with a deposit file (starts with `deposit_data-` and ends with `.json`) and one or more keystore files (starts with `keystore-` and ends with `.json`), 1 per validator. Copy them around if needed. Make sure your deposit file and your keystore files are in a known and accessible location on your machine.

![image](https://user-images.githubusercontent.com/46390405/200504087-25288f49-c015-434d-8cf2-0be85c3f56ec.png)


Next we will do the deposit using the Prater launchpad. Make sure you have access to a browser with MetaMask, your account with the funds from the faucet and the deposit file we just created.

Go to [the Prater launchpad](https://prater.launchpad.ethereum.org/en/). Follow the instructions, make sure *Prater* is the selected network in MetaMask and use the deposit file to perform your deposit.

You can check that your deposit transaction went through on [the transaction explorer](https://goerli.etherscan.io/address/0xff50ed3d0ec03aC01D4C79aAd74928BFF48a7b2b).

### Put your 2 Json Deposit and Keystore files in the Vps Terminal
![image](https://user-images.githubusercontent.com/46390405/200514743-1bce2bca-af05-4b19-9d6e-408992a64527.png)


### Configuring your Lighthouse validator client

Create a dedicated user for running the Lighthouse validator client, create a directory for holding the data and assign the proper permissions.

```
sudo useradd --no-create-home --shell /bin/false lighthousevalidator
sudo mkdir -p /var/lib/lighthouse/validators
sudo chown -R lighthousevalidator:lighthousevalidator /var/lib/lighthouse/validators
sudo chmod 700 /var/lib/lighthouse/validators
```

Import your keystore that includes your validator key for the Lighthouse validator client. Running the first command will prompt you for that keystore password. Make sure to enter it correctly and avoid leaving it blank. Make sure to replace `/path/to/keystores` with the actual path to your keystores created [in the previous step](#creating-your-validator-keys-and-performing-the-deposit).

```
sudo /usr/local/bin/lighthouse account validator import \
    --directory /root/keystores \
    --datadir /var/lib/lighthouse \
    --network prater
sudo chown -R lighthousevalidator:lighthousevalidator /var/lib/lighthouse/validators
```

Create a systemd service config file to configure the Lighthouse validator client service.

```
sudo nano /etc/systemd/system/lighthousevalidator.service
```

Paste the following service configuration into the file. Exit and save once done (`Ctrl` + `X`, `Y`, `Enter`). Make sure to replace the `0x0000000000000000000000000000000000000000` address with your own Ethereum address that you control where you want to receive the transaction tips.

```ini
[Unit]
Description=Lighthouse Ethereum Client Validator Client (Prater)
Wants=network-online.target
After=network-online.target

[Service]
User=lighthousevalidator
Group=lighthousevalidator
Type=simple
Restart=always
RestartSec=5
ExecStart=/usr/local/bin/lighthouse vc \
    --network prater \
    --datadir /var/lib/lighthouse \
    --graffiti EthStaker \
    --metrics \
    --suggested-fee-recipient 0x0000000000000000000000000000000000000000

[Install]
WantedBy=multi-user.target
```

Reload systemd to reflect the changes and start the service. Check status to make sure it’s running correctly.

```
sudo systemctl daemon-reload
sudo systemctl start lighthousevalidator.service
sudo systemctl status lighthousevalidator.service
```

It should say active (running) in green text. If not then go back and repeat the steps to fix the problem. Press Q to quit (will not affect the Lighthouse validator client service).

Enable the Lighthouse validator client service to automatically start on reboot.

```
sudo systemctl enable lighthousevalidator.service
```

You can watch the live messages from your Lighthouse validator client logs using this command.

```
sudo journalctl -f -u lighthousevalidator.service -o cat | ccze -A
```

Press `Ctrl` + `C` to stop showing those messages.

## What's next?

You performs a lot of different tasks to help with the [*#TestingTheMerge*](https://twitter.com/search?q=%23TestingTheMerge) initiave. Check out [the program structure](https://hackmd.io/WKpg6SNzQbi1jVKNgrSgWg). There are different tasks for all technical abilities.

## Support

If you have any question or if you need additional support, make sure to get in touch with people involved with this initiative:

* EthStaker Discord: [discord.io/ethstaker](https://discord.io/ethstaker) in the #ropsten channel
* Eth R&D Discord: [discord.gg/qGpsxSA](https://discord.gg/qGpsxSA) in the #testing channel under the *Merge* category.

## Credits

Based on [Somer Esat's guide](https://github.com/SomerEsat/ethereum-staking-guide).
Based on [Ethereum community's guide](https://notes.ethereum.org/qrDBhhydTsyKFmGaBl2COQ).
