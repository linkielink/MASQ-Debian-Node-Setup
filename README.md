# Setting up a Home Server and use it as a MASQ Node

## 1. Setting up the HomeServer

If you never setup a debian HomeServer before, I would recommend using this step by step tutorial: https://www.youtube.com/watch?v=aJuX9c4DCRo

## 2. De-Occupping Port 80

The MASQ Node will need to use the Port 80. As I have a **munin** status monitor running for my home server, I decided to move it off of Port 80 to Port 8080. With that the only difference going forward for me is, that I need to add :8080 to the IP address of my HomeServer, whenever I want to access the munin dashboard (e.g: 192.168.x.x:8080/munin).

## 3. Creating the Port Forwarding Rules

I daisychained two routers in my network. Because of that I have to create two port forwardings for the MASQ Node.
I chose Port 8091 (this is the unofficial CouchBase administration port, that I never use).
I set up a first Port forwarding for port 8091 from my “public” router (Telekom, that connects to the internet) to my private router (ASUS RT-AX88U that is permanently connected with a PIA VPN). After that I set up the same Port forwarding for Port 8091 inside the ASUS Router to my HomeServer.

## 4. Getting tMASQ and Mumbai testnet MATIC

I added the Mumbai Testnet to my Metamask. If you’re unsure how to add Mumbai to Metamask I recommend reading this wiki by Polygon: https://wiki.polygon.technology/docs/develop/metamask/config-polygon-on-metamask/

To get tMASQ, just open a ticket in the support channel of the MASQ discord (https://discord.gg/masq) and you will be helped

I also imported the tMASQ to my wallet, by clicking on “import tokens” and adding the following credentials.

```
Token Contract Address: 0x9B27034acaBd44223fB23d628Ba4849867cE1DB2
Token Symbol: tMASQ
Token Decimal: 18
```

I kept those tokens for later to send them over to the MASQ Node Wallets

## 5. Getting a Descriptor

To make the MASQ node working you will need add a descriptor to your node. You will need to have access to a specific channel in the MASQ discord, the Moderators will help you with that as well.
You will be presented to a channel, with a list of public nodes. Copy one of those and use it. Later you’ll be able to get your own descriptor by using the masq cli.

_TIPP: use two descriptors initially (separated with a comma – no space)_

## 6.Getting your HomeServer’s external public IP

If you want to know what your external IP address is, just connect to your HomeServer and use this command:

```
dig +short txt ch whoami.cloudflare @1.0.0.1
```

## 7. Getting a RPC endpoint

You will need a rpc endpoint to run your node. https://alchemy.com is a good place to get it. Sign up for free and create a new app on Polygon Mumbai.
Copy the HTTPS API Endpoint including your key (e.g.: https://polygon-mumbai.g.alchemy.com/v2/[API-KEY]) and store it somewhere safe for later.

## 8. Preparing the Server

Ready? Let’s go. This was also a first for me, that’s why I walk you through every command that I used to make sure we’re on the same path to success here.

_I used Stephan’s video tutorial (https://www.youtube.com/watch?v=JrKSfwDA334)l for the following steps, you should be fine by just following and copying whatever commands you see listed below._

I connected to my HomeServer by opening a ssh connection

```
ssh [username]@192.168.x.x
```

After logging in with my password, I switched to the root user

```
su
```

Update your apt packages

```
apt update
```

```
apt upgrade
```

Install the sudo, screen and unzip package

```
apt install sudo screen unzip
```

Add a new user named masqnode

```
useradd -m masqnode
```

And add it to the sudo group

```
usermod -aG sudo masqnode
```

Give the user a password

```
passwd masqnode
```

Now exit the root user

```
exit
```

And exit the server entirely

```
exit
```

Reconnect to the server with the new user

```
ssh masqnode@192.168.x.x
```

To secure your server and permit root login open the sshd_config file

```
sudo nano /etc/ssh/sshd_config
```

Remove the # infront of PermitRootLogin and set the value to no

```
PermitRootLogin no
```

**Press Ctrl + o, press Enter and press Ctrl + x to exit the editor**

Restart the ssh service

```
sudo service ssh restart
```

## 9. Setting up the MASQ Node

Are you still logged in as masqnode? Yes? Good. If not, do so now.

Navigate to https://github.com/MASQ-Project/Node/releases
Copy the link of the Node-[version]-linux.zip file by right clicking and selecting “copy link address” (e.g: https://github.com/MASQ-Project/Node/releases/download/v0.6.2/Node-0.6.2-linux.zip)
**NOTE: Currently only v0.6.2 can be installed on debian - ask in the Discord for help if you need the binaries for v0.7.0 or higher**

Get back to the server and create a folder called node

```
mkdir node
```

And navigate to it

```
cd node
```

Download the binary to your HomeServer

```
wget https://github.com/MASQ-Project/Node/releases/download/v0.6.2/Node-0.6.2-linux.zip
```

Unzip the package

```
unzip Node-0.6.2-linux.zip
```

Delete the zip file

```
rm Node-0.6.2-linux.zip
```

Make the files executable

```
sudo chmod +x *
```

Get your user and group id (and copy them to a file to use them later)

```
id
```

Create a file called config

```
nano config.toml
```

Copy the rpc endpoint you generated in step 8 and insert it as blockchain-service-url

```
blockchain-service-url = "https://polygon-mumbai.g.alchemy.com/v2/[API-KEY])"
```

Insert the home servers IP address from step 7

```
ip = "[SERVER PUBLIC IP]"
```

And insert the chain, you want to use

```
chain = "polygon-mumbai"
```

Insert the port, you setup a forward to in step 3

```
clandestine-port = "8091"
```

Add some open free dns server from here https://www.lifewire.com/free-and-public-dns-servers-2626062 separated by comma, without space

```
dns-server = "1.1.1.1,8.8.8.8,9.9.9.9"
```

Set the log-level to info

```
log-level = "info"
```

The Neighborhood Mode will be set to standard

```
neighborhood-mode = "standard"
```

Set the Gas price the node will pay in gwei for every transaction to a reasonable value taken from a transaction in polyscan or leave it at the default of 1

```
gas-price = "40"
```

Set the Real User consisting of user id, group id and home directory.

```
real-user = "1001:1001:/home/masqnode"
```

Set the neighbors by using the descriptors you got in step 5

```
neighbors = "masq://polygon-mumbai:4b3lJZIvs9f4AeHV2rGPKBL03kiH-b-QSkxCkQYxwEk@45.77.61.140:6421,masq://polygon-mumbai:19f2j5hCmxenhx95P7hNxHAlYjeNBBw3n4ZyurZr2VI@155.138.231.133:39603"
```

**Save the file by pressing Ctrl + o, Enter and then Ctrl + x**

Your config.toml should look something like this now:

```
blockchain-service-url = "[rpc endpoint]"
ip = "[SERVER PUBLIC IP]"
chain = "polygon-mumbai"
clandestine-port = "8091"
dns-servers = "1.1.1.1,8.8.8.8,9.9.9.9"
log-level = "info"
neighborhood-mode = "standard"
gas-price = "40"
real-user = "1001:1001:/home/masqnode"
neighbors = "masq://polygon-mumbai:4b3lJZIvs9f4AeHV2rGPKBL03kiH-b-QSkxCkQYxwEk@45.77.61.140:6421,masq://polygon-mumbai:19f2j5hCmxenhx95P7hNxHAlYjeNBBw3n4ZyurZr2VI@155.138.231.133:39603"
```

## 10. Starting the node for the first time

Start a node in zero-hop mode

```
sudo ./MASQNode --neighborhood-mode zero-hop --data-directory /home/masqnode/node
```

Open another shell terminal and keep the previous one open.
Navigate to the node folder

```
cd node
```

Start the masq CLI

```
./masq
```

Create a password, save it somewhere safe and set it as the database password

```
set-password [PASSWORD]
```

Generate wallets and save the seed phrase informations you get somewhere safe

```
generate-wallets –db-password [PASSWORD]
```

**Exit the masq CLI with Ctrl + c**

**Switch to the previous Terminal and press Ctrl + c again**

Open the config.toml in an editor

```
nano config.toml
```

Add the parameter db-password

```
db-password = "[PASSWORD]"
```

## 11. Start Your node

Start the node

```
sudo ./MASQNode --data-directory /home/masqnode/node --config-file /home/masqnode/node/config.toml
```

**Press Ctrl + c to stop the node running**

## 12. Setting up screen to keep the node running

To keep the node running in the background of your server, just use the previously installed screen package.

Create a new screen

```
screen -S masq
```

And start the node again

```
sudo ./MASQNode --data-directory /home/masqnode/node --config-file /home/masqnode/node/config.toml
```

**To exit the screen press Ctrl + a + d**

To resume the screen with the running node run

```
screen -r masq
```

**To kill the screen press Ctrl + k**

# Updating your MASQ Node

If a new version of the MASQ Node binaries is released I replace my node installation by rerunning the step 9.

## Debian Support

However, if you only need to replace the masq and MASQNode binary because the linux installer of newer versions doesn't work with Debian.
Here's are two quick commands to do so

```
scp /[LOCAL_FOLDER]/masq masqnode@[IP_ADDRESS_OF_YOUR_NODE]:"node/masq"
```

and

```
scp /[LOCAL_FOLDER]/MASQNode masqnode@[IP_ADDRESS_OF_YOUR_NODE]:"node/MASQNode"
```

Enter the password for your masqnode user and it will upload and override the file from your local system on your node.

After that login via ssh and run

```
cd node
```

and

```
sudo chmod +x *
```

To make the binaries executable again.

## Delete the existing Database

Delete the Database by running

```
rm node-data.db
```

## Clearing the data-directory

Now you have to clear the `data-directory`, to find the path to the directory, open a new terminal and connect to your node via ssh.
Navigate to the node folder

```
cd node
```

and run the MASQNode Deamon in initialization mode

```
sudo ./MASQNode --initialization
```

Leave this terminal open and go back to your initial terminal window where you are located in the `node` folder and run the `masq setup` CLI.

```
./masq setup
```

Check the value of your `data-directory` and navigate to the folder

```
cd [PATH_OF_data-directory]
```

If you're told that you `can't cd to [PATH_OF_data-directory]` then give the folder the needed permission

```
sudo chmod o+x [PATH_OF_data-directory]
```

Now remove the contents of the folder.

## Recover your wallets

Because we deleted the Database file, we need to recover the nodes wallets. Start the masq CLI again

```
./masq
```

Set the same database password, you set in step 10

```
set-password [PASSWORD]
```

And restore your wallets with the data you stored somewhere safe in step 10

```
recover-wallets --db-password [PASSWORD] --consuming-key [PRIVATE_KEY_OF_CONSUMING_WALLET] --earning-address [WALLET_ADDRESS_OF_EARNING_WALLET]
```

Now you Node is updated and you should be able to start it regularly again.
