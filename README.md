# Setting up and Running a Tidechain Validator

- Date: October 10, 2022
- Author: Jeff Galbraith
- Contributing Authors: David Lemarier, Jonathon Provost

**TIP:** If you are interested in setting up a validator node on the Tidechain ecosystem, read this document in its entirety before proceeding so you have a full understanding before execution.

## What is TIDEFI?

**TIDEFI. DeFi for you.**

Tidefi is a novel decentralized exchange (DEX), for all kinds of digital assets, that is built on the substrate-based Tidechain, a permissionless nominated Proof of Stake (nPoS) Blockchain. Tidefi brings together the speed and performance of centralized exchanges with primitives of decentralized finance (DeFi). The Tidefi Token (Ticker: TDFY) governs both the underlying **Tidechain** network as well as creating new avenues for its holders to participate both in the validation of the network as well as the growth and revenue of the TIDEFI exchange.

Tidefi and its community are centered around open-source technology and decentralization and are committed to creating opportunities for all individuals and institutions that are part of the Tidefi ecosystem. Join our community, start trading and build the financial future that you dream of with us.

## What is a Tidechain Validator?

A blockchain validator is someone responsible for verifying transactions on a blockchain. Once transactions are verified, they are added to the distributed ledger. In proof of stake (PoS) systems like Tidechain, validators are given rewards as long as they stake the network’s token (TDFY) and correctly participate in the network. This mechanism helps secure the network by imposing the need to lock up value in the network to participate in the consensus decisions. Open-source software is available for those that wish to become validators.

## Why Become a Tidechain Validator?

Network validators verify user transactions. If all validators reach a consensus that a transaction is valid, it is included in the blockchain. Invalid transactions are rejected. Therefore, validators have to correctly process user transactions at maximum speed to get rewarded and avoid penalties. Validators stake their tokens as collateral to support the network’s security and, in exchange for their service, earn rewards. Validator nodes can also charge a commission, which gives them a percentage of the pooled stake rewards, based on the number of transactions they process.

## Requirements

If you have decided that running a validator node is the right thing for you, then there are a few recommended hardware requirements, as well as requirements for becoming an elected Validator that can process transactions.

It is highly recommended that you have significant system administration experience with Linux before attempting to run a validator node.

You must be able to handle technical issues and anomalies with your node which you must be able to tackle yourself. Running a validator can involve much more than just executing the Tidechain binary.

### Hardware

It is recommended to have the following bare-metal hardware with a fixed IP:

| CPUs | Memory | SSD  | Network |
| ---- | ------ | ---- | ---     |
| 16   | 64 GB  | 1 TB | 1 GB/s  |

### Bonding

To have your Validator elected to the chain, the minimum amount is 100k TDFY.

## Setting Up a Tidechain Validator

First of all, select your hardware provider and use Ubuntu 20.04 (do not use 22.04 as this is not currently supported by the chain). There are steps you can take for securing your server. These will be discussed below. If your provider supports MFA (multi-factor authentication), do it now before proceeding. Always keep in mind that security should be front and foremost in all your endeavors to be a Tidechain validator.

After your purchase, you will receive the necessary SSH credentials to access your server. All commands below assume your server is running Linux and you're also accessing your server from Linux (Windows is not supported in this document).

```
ssh root@your-server
```

Make sure your server is fully updated.

```
sudo apt update && sudo apt upgrade -y
```

Also, install the net-stat tools (optional).

```
sudo apt install net-tools
```

Create a new user (as we will be protecting the root user) with sudo privileges.

```
adduser newusernamehere
```

Add your new user to the sudo group.

```
usermod -aG sudo newusernamehere
```

### Hardening the SSH Config

Edit the SSH config:

```
sudo nano /etc/ssh/ssh_config
```

Change the `Port` to a value under 1024 and change the `AddressFamily` to `inet`.

```
Port 911
AddressFamily inet
```

Logging changes:

```
# Logging
SyslogFacility AUTH
LogLevel INFO
```

Other options for hardening:

```
LoginGraceTime 2m
PermitRootLogin prohibit-password
StrictModes yes
MaxAuthTries 6
MaxSessions 10

PubkeyAuthentication yes

AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2
PermitEmptyPasswords no
X11Forwarding no
```

Save the config and then restart sshd:

```
sudo systemctl restart ssh
```

You can verify if it's listening on the right port/interface:

```
netstat -tpanu |grep LISTEN
```

### SSH | Generate Your Keypair

On your client system – the one you’re using to connect to the server – you need to create a pair of key codes.

First, we will create the ssh directory for storing keys and your client config, if it doesn't already exist:

```
mkdir –p $HOME/.ssh
```

Set the permissions:

```
chmod 0700 $HOME/.ssh
```

Generate your keys:

```
ssh-keygen –t rsa -b 4096
```

Save the key with a different name (like sn_validator) that will be easy to identify in case you already have some and remember to save your passphrase.

```
Enter file in which to save the key (/home/yourname/.ssh/id_rsa): /home/yourname/.ssh/sn_validator
```

### Copy Public Key to Your Server

On the client system, use the `ssh-copy-id` command to copy the identity information to the server, and don't forget to change below to the correct port that you changed above:

```
ssh-copy-id -p PORT -i $HOME/.ssh/sn_validator newusernamehere@server_ip
```

Example:

```
$ ssh-copy-id -p 911 -i $HOME/.ssh/sn_validator newusernamehere@server_ip

/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/xxxx/.ssh/sn_validator.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
newusernamehere@server_ip's password: 

Number of key(s) added: 1
```

Now try logging into the machine, with (change the port value if you used something else):

```
ssh -p 911 newusernamehere@server_ip
```

and check to make sure that only the key(s) you wanted were added. But, keep your other console going in case a mistake was made that needs to be fixed.

### Verify Your SSH Key

```
ssh -i $HOME/.ssh/sn_validator -p 911 newusernamehere@server_ip          

Enter passphrase for key '/home/xxxx/.ssh/sn_validator':
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-125-generic x86_64)
```

### Turn Off Password Authentication

On the server, edit sshd_config:

```
sudo nano /etc/ssh/sshd_config
```

Search the file and find the `PasswordAuthentication` option.
Edit the file and change the value to **no**:

```
PasswordAuthentication no
```

Save the file and exit, then restart the SSH service:

```
sudo systemctl restart ssh
```

Verify that SSH is still working before ending the session. From your remote workstation:

```
ssh -i $HOME/.ssh/yourkeyname -p PORT newusernamehere@server_ip
```

### Make Your Life Easier

In case you are managing multiple remote locations with different identities, you can edit your client computer's ssh config to make it easier.

Edit your client config file:

```
nano $HOME/.ssh/config
```

And you can add your information following this example:

```
Host snvalidator
     HostName server_ip
     User newusernamehere
     Port 911
     IdentityFile ~/.ssh/sn_validator
     IdentitiesOnly yes
```

Save the file and try to log in to your remote server:

```
ssh snvalidator
```

You can now close all console windows knowing your server is hardened.

## Validator Node Setup with Tidechain

Now it's time to install and setup the Validator node. We will use the DEB package. The DEB package will create the `tidechain` user and `tidechain` group on the remote system.

Log onto your server from your client:

```
ssh newusernamehere@server_ip
```

Download the latest Tidechain package (look to see what the latest release is before doing this step and change the appropriate values accordingly):

```
wget https://github.com/tidelabs/tidechain/releases/download/v0.6.0/tidechain.deb && sudo dpkg -i tidechain.deb
```

Verify user and group:

```
cat /etc/group | grep tidechain && cat /etc/passwd | grep tidechain
tidechain:x:119:
tidechain:x:112:119::/home/tidechain:/usr/sbin/nologin
```

Start your first sync (if you are setting up a node for Testnet, then change `--chain=tidechain` to `--chain=lagoon`):

```
sudo runuser -u tidechain -- tidechain --chain=tidechain --pruning=archive
```

This should start the first sync and it should sync as a tidechain user.
You can confirm with the log from the tidechain binary.

You should see something like this:

```
2022-06-02 13:21:17 Tidechain
2022-06-02 13:21:17 ✌️  version 0.6.0-295fd1b0a9e
2022-06-02 13:21:17 ❤️  by Semantic Network Team <publishers@tidelabs.org>, 2021-2022
2022-06-02 13:21:17 📋 Chain specification: tidechain
2022-06-02 13:21:17 🏷  Node name: stiff-beast-8220
2022-06-02 13:21:17 👤 Role: FULL
** 2022-06-02 13:21:17 💾 Database: RocksDb at /home/tidechain/.local/share/tidechain/chains/tidechain/db/full **
```

## Creating Your Validator Accounts

While the chain is syncing, let's get your accounts set up so the bonding can occur to the validator.

Download the latest **Tidechain Explorer** from here (Windows, Mac OS, and Linux are supported): https://github.com/tidelabs/explorer/releases

**Note:** By default, the Tidechain Explorer is set to `lagoon`, the Testnet chain. You will need to switch the network by selecting "custom endpoint" and change `rpc.lagoon.tidefi.io` to `rpc.tidefi.io` (this may change in a future release).

You will need to create two accounts:

| Account | Description |
| -------- | -------- |
| Stash     | This is the primary account that holds the funds and has a portion bonded for participation |
| Controller     | This is used to control the operation of the validator, switching between validating and idle; (It only needs enough funds to send transactions when actions are taken) |

I have found 3 accounts work best for me and I will touch on this in a bit.

For best practices, we recommend naming your `stash` account as `val-0` and your `controller` account as `val-0-c`. This way, if you run more than one validator, all you need to do is increment the number.

**TIP:** Make sure you remember your passwords and save and store the mnemonic files safely (preferably somewhere external).

If you want to use an `on-chain identity` then this is where you create your third account.

Once you have created your accounts, you will need to do the bonding. However, you will need to send your accounts some TDFY, to do the bonding, and for paying any transaction fees. You will need to make sure your stash account has the minimum 100k TDFY for the bonding. You will also need some TDFY on your controller account. A minimum of 500 TDFY works, but more is always better.

If you created a third account to handle the on-chain identity, then set the identity now on that account by selecting `Set on-chain identity` from the menu of that account (hopefully you sent some TDFY to that account as well to handle the transaction).

![](https://i.imgur.com/hI66HpW.png)

Fill out the information as per your needs.

![](https://i.imgur.com/R740sBB.png)

Click the `Set Identity` to complete the operation.

Now, from the menu on that same account, select the `Set on-chain sub-identities`.

Select the `stash` account and give it a name (`val-0`) as this will override the original name given. Click the `Add sub` button, then select your `controller` account and give it a name (`val-0-c`).

![](https://i.imgur.com/hVSB0TB.png)

Click the `Set Subs` button.

**TIP:** Selecting a good identity on the chain may help you get good nominations for your validator.

### Bonding Your Validator Account

From the top-level menu in the Tidechain Explorer, select "Network" -> "Staking" and then select the "Accounts" tab. On the right, click the `Stash` button:

![](https://i.imgur.com/4JPvjfc.png)

On the following screen, select your `stash` account and the `controller` account and add your `100000` TDFY for bonding to the `value bonded` field:

![](https://i.imgur.com/sixOjkv.png)

You can also look at the `payment destination` field, in case you want to change that. It is recommended to leave it as-is for the time being. It can always be changed later.

Now click the `Bond` button.

### Associating Your Accounts to Your Tidechain Instance

Once the tokens have been bonded and the chain synced, we can get the chain running and associate it with your `stash` account.

First, make sure the tidechain process is not running. If you followed the steps above (with `runuser`), you can CTRL-C to stop it. If you are in a different terminal, you may need to:

```
sudo pkill tidechain 
```

Now, a session token needs to be created. To do this, we have to run the chain in `unsafe` mode (generating session tokens are only available while in `unsafe` mode).

**TIP:** If you are setting up a validator for Testnet, then change `--chain=tidechain` to `--chain=lagoon`

```
sudo runuser -u tidechain -- tidechain --chain=tidechain --validator --rpc-methods=unsafe
```
Once your node is running in `unsafe` mode, open another terminal to your server and run this command:

```
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_rotateKeys", "params":[]}' http://localhost:9933
```

The output will have a hex-encoded "result" field. The result is the concatenation of the four public keys. Save this result for the next step.

Restart the node without the `--rpc-methods=unsafe` flag.

It's time to associate the `results` data between validator and node. In the Tidechain Explorer, navigate to "Network" => "Accounts" and click the "Session Key" link for your `stash` account. Paste the `results` value into the "Keys from rotateKeys" field and then click the "Set Session Keys" button.

![](https://github.com/tidelabs/tidechain/raw/dev/docs/assets/explorer-session-key.gif)

After that (and this is important and many forget this step), we have to provide our intention to validate by clicking the `Validate` button.

![](https://github.com/tidelabs/tidechain/raw/dev/docs/assets/explorer-validate.gif)

Congratulations! Your node is set up to validate. Hopefully, it gets elected in the next era. The validator set is refreshed every era. In the next era (every 24 hours), if there is a slot available and your node is selected to join the validator set, your node will become an active validator. Until then, it will remain in the waiting queue. If your validator is not selected to become part of the validator set, it will remain in the waiting queue until it is. There is no need to restart if you are not selected for the validator set in a particular era. However, it may be necessary to increase the number of TDFY bonded or seek out nominators for your validator to join the validator set.

In the meantime, we are not quite finished with the set up, so let's do that now.

## Permanently Running Your Validator Node

We are done with temporarily running the node and should get it running in a more permanently.

We need to do the following:

```
sudo nano /etc/default/tidechain
```

And, then set the tidechain args as follows:

```
TIDECHAIN_CLI_ARGS="--chain tidechain --validator --name=YOUR_NAME-validator-0"
```

The `--name` part of the flags is used in the telemetry (different from on-chain). As seen here: https://telemetry.tidefi.io/#/0xe02d5f0a06c10faa29d19ac1fe3aae40c41f819de074a8047a7814a5cbed7f6b

**TIP:** If you want to use a monitoring service, we can add additional flags to the above file. You can add to the end:

```
--prometheus-external
```

Now, you can use a service to verify your validator node is running and healthy to avoid slashes.

You can access the information like so:

```
http://server_ip:9615/metrics
```

You can now use services such as https://uptimerobot.com/ or https://hetrixtools.com/ to monitor and inform you should your validator go down.

Time to set up your validator node to permanently run.

```
sudo systemctl enable tidechain
sudo systemctl start tidechain
```

The first command allows systemd to start your validator node after a reboot. The second one starts it now.


## Updating Your Validator Node

When the time comes for a chain update, follow these instructions. If you are validating on-chain, you want to do things quickly to avoid slashes.

First of all, verify the version for the release here: https://github.com/tidelabs/tidechain/releases.

On Tidechain Explorer observe when your node validates a block. Then, perform the following (replace with the correct version number):

```
rm -rf tidechain.deb
wget https://github.com/tidelabs/releases/download/vx.x.x/tidechain.deb
sudo dpkg -i tidechain.deb
sudo systemctl daemon-reload
sudo service tidechain restart
```

or, a more automated way where you don't have to worry about the version:

```
curl -s https://api.github.com/repos/tidelabs/tidechain/releases/latest \
| grep "browser_download_url.*deb" \
| cut -d : -f 2,3 \
| tr -d \" \
| wget -qi - \
| sha256sum -c tidechain.deb.sha256 && sudo dpkg -i tidechain.deb && sudo systemctl daemon-reload && sudo systemctl restart tidechain 
```

To verify it is running:

```
sudo tail -f /var/log/syslog
```

## Updating Your Server

Occasionally, you will need to update your server.

```
sudo apt update && sudo apt upgrade -i
```

To see if a reboot is needed:

```
cat /var/run/reboot-required
```

If the file does not exist, no reboot is required. If a reboot is required, watch when your validator node processes a block on the chain and then immediately reboot. If all goes well, your system should be back in less than 10 minutes and won't be slashed.

## Backing Up Important Files

When you created your accounts using the Tidechain Portal, you would have been presented with a PDF to download for each one that contains the mnemonics for that account. This is one way to recover an account. These files should be saved in a secure location.

You will also want to back up on your client computer `~/.config/com.semnet.explorer`.

On the validator node system, you will want to back up the following folder:

(Testnet/Lagoon): /home/tidechain/.local/share/tidechain/chains/lagoo1/keystore

(Mainnet/Tidechain): /home/tidechain/.local/share/tidechain/chains/tidec1/keystore



## More Resources

- [Tidechain Explorer](https://github.com/tidelabs/explorer/releases)
- [Medium | Running a Validator on the Tidefi Lagoon Testnet](https://medium.com/tidefi/running-a-validator-on-the-tidefi-lagoon-testnet-f2928731f32c)
- [GitHub Sources](https://github.com/tidelabs/tidechain)
- [Tidechain API](https://tidelabs.github.io/tidechain/tidechain_service/index.html)
- [Telemetry](https://telemetry.tidefi.io/#/0xc87195c66912e4280aa2aa8498e5bd3fae699f364d3a156fd716a79f27f97c7f)
- [Web Explorer](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc.tidefi.io#/explorer)
