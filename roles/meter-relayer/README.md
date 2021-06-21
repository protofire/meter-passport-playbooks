Meter passport relayer
=========

Ansible role to configure and run Meter Passport Relayer. Includes docker and docker-compose installation. Clones the official meterio repository, builds binary and the conatiner image. Creates unit-file to manage `chainbridge` container with docker-compose.

Meter passport relayer requires an account for signing the bridge transfers. The relayer software stores keypairs in encrypted key file to provide a basic level of security. If you use this ansible role you have two options to obtain a key file:

- Generate New Key (this is the default option)
- Import Existing Private Key (to use this option define `private_key` variable in your playbook for this role. This variable should contain a private key string)  

Requirements
------------

The role works on RedHat, CentOS, Amazon Linux, Debian and Ubuntu distributions. The role requires root access. Tested on:

- Ubuntu 18.04

Role Variables
--------------

| Variable | Description | Default Value |
| :------- | :---------- | :------------ |
| go_version | go version to download and install | "1.16.4" |
| go_shasum  | sha256 sum to check downloaded go  | "7154e88f5a8047aad4b80ebace58a059e36e7e2e4eb3b383127a28c711b4ff59" |
| docker_compose_version | docker-compose version to install | "1.29.2" |
| docker_compose_shasum | sha256 sum to check downloaded docker-compose | "f3f10cf3dbb8107e9ba2ea5f23c1d2159ff7321d16f0a23051d68d8e2547b323" |
| keystore_path | Path where key-file will be created and stored | "/root/chainbridge/keys/" |
| keystore_password | Password for encrypted key-file | |
| private_key       | Define it if you want to import existing private Key. Should contain private key string | |
| eth_rpc_endpoint | Ethereum node RPC endpoint |  |
| eth_start_block | The block to start processing events from in Ethereum chain | |
| eth_bridge_address | Address of the bridge contract in Ethereum chain | "0xbD515E41DF155112Cc883f8981CB763a286261be" |
| eth_erc20_handler | Address of erc20 handler in Ethereum chain | "0xde4fC7C3C5E7bE3F16506FcC790a8D93f8Ca0b40" |
| eth_generic_handler | Address of generic handler in Ethereum chain | "0x517828d2549cEC09386f89a67E92825E26740240" |
| eth_gas_limit | Gas limit for transactions | "1000000" |
| eth_max_gas_price | Gas price for transactions | "1000000000000" |
| eth_gas_multiplier | Multiplies the gas price by the supplied value | "1" |
| eth_block_confirmations | Number of blocks to wait before processing a block in Ethereum chain | "25" |
| eth_http | Whether the chain connection is `ws` or `http` | "true" |
| mtr_rpc_endpoint | Meter node RPC endpoint |  |
| mtr_start_block | The block to start processing events from in Meter chain |  |
| mtr_bridge_address | Address of the bridge contract in Meter chain | "0x7C6Fb3B4a23BD9b0c2874bEe4EF672C64e83838B" |
| mtr_erc20_handler | Address of erc20 handler in Meter chain | "0x60f1ABAa3ED8A573c91C65A5b82AeC4BF35b77b8" |
| mtr_generic_handler | Address of generic handler in Meter chain | "0x89CA53Bf11d24D32A7aC3aDb7750868360c90590" |
| mtr_gas_limit | Gas limit for transactions | "8000000" |
| mtr_max_gas_price | Gas price for transactions | "500000000000" |
| mtr_gas_multiplier | Multiplies the gas price by the supplied value | "1" |
| mtr_block_confirmations | Number of blocks to wait before processing a block in Meter chain | "0" |
| mtr_http | Whether the chain connection is `ws` or `http` | "true" |
| bsc_rpc_endpoint | Binance smart chain node RPC endpoint | |
| bsc_start_block | The block to start processing events from in Binance chain | |
| bsc_bridge_address | Address of the bridge contract in Binance chain | "0x223fafbc2cA53A75CcfF5B2369128d3d1a828F36" |
| bsc_erc20_handler | Address of erc20 handler in Binance chain | "0x5945241BBB68B4454bB67Bd2B069e74C09AC3D51" |
| bsc_generic_handler | Address of generic handler in Binance chain | "0x83Fc24eB56121FA2A05e0b5170E7310738425839" |
| bsc_gas_limit | Gas limit for transactions | "1000000" |
| bsc_max_gas_price | Gas price for transactions | "10000000000" |
| bsc_gas_multiplier | Multiplies the gas price by the supplied value | "1.4" |
| bsc_block_confirmations | Number of blocks to wait before processing a block in Binance chain | "3" |
| bsc_http | Whether the chain connection is `ws` or `http` | "true" |

Dependencies
------------

-

Example Playbook
----------------

```yaml
---
- hosts: relayer-host-name
  become: true
  roles: 
    - role: meter-relayer
      keystore_password: "Pa$$w0rd"
      eth_start_block: "12612223"
      eth_rpc_endpoint: "http://<ETHEREUM_NODE_IP>:8545"
      mtr_start_block: "12331193"
      mtr_rpc_endpoint: "http://<METER_NODE_IP>:8545"
      bsc_start_block: "8199580"
      bsc_rpc_endpoint: "ws://<BINANCE_NODE_IP>:8546"
      bsc_http: "false"
```

# Usage

## Step 1: Install prerequisites

To use this role you will need [Ansible >= 2.5.1](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

## Step 2: SSH connection

Make sure you can connect to the host you're going to run Meter Passport Relayer on via SSH with ssh-key. We reccomend using a separate key for ansible deployment. To generate a new key pair run:

```bash
ssh-keygen -t rsa -b 4096 -C "ansible-deployer" -f ~/.ssh/ansible-meter-relayer
```

After the key is generated push it to your host where Meter Passport Relayer will be installed:

```bash
ssh-copy-id -i ~/.ssh/ansible-meter-relayer <YOUR_RELAYER_USER>@<YOUR_RELAYER_IP>
```

## Step 3: Ansible inventory file

Clone this repository and create a file named `hosts.txt` in repository's root directory. The file should be like this:
```ini
[meter]
meter-relayer ansible_host=<YOUR_RELAYER_IP> ansible_user=<YOUR_RELAYER_USER> ansible_ssh_private_key_file=~/.ssh/ansible-meter-relayer
```

## Step 4: Ansible playbook
Create a file named `meter-relayer-playbook.yml` in repository's root directory. Fill it like this:
```yaml
- hosts: meter
  become: true
  roles: 
    - role: meter-relayer
      keystore_password: "<SOME_VERY_STRONG_PASSWORD>"
      eth_start_block: "<SEE_VARIABLES_DESCRIPTION>"
      eth_rpc_endpoint: "<SEE_VARIABLES_DESCRIPTION>"
      eth_http: "<SEE_VARIABLES_DESCRIPTION>"
      mtr_start_block: "<SEE_VARIABLES_DESCRIPTION>"
      mtr_rpc_endpoint: "<SEE_VARIABLES_DESCRIPTION>"
      mtr_http: "<SEE_VARIABLES_DESCRIPTION>"
      bsc_start_block: "<SEE_VARIABLES_DESCRIPTION>"
      bsc_rpc_endpoint: "<SEE_VARIABLES_DESCRIPTION>"
      bsc_http: "<SEE_VARIABLES_DESCRIPTION>"
```
Make sure you've placed all the necessary variables in the `meter-relayer-playbook.yml`. You can find variables description in this README file. Also do not forget to define `private_key` variable if you prefere using existing account instead of generating a new key-pair.

After all these steps run:
```bash
ansible-playbook meter-relayer-playbook.yml
```
Ansible will install and configure all the stuff

License
-------

GPLv3

Author Information
------------------

 - [Dzmitry Kliapkou](https://github.com/dzmitrykliapkou) 