
## 1. Update Drosera CLI (Droseraup)

```bash
curl -L https://app.drosera.io/install | bash
```
```bash
source /root/.bashrc
```
```bash
droseraup
```

## 2. Update Seed Node in `drosera.roml`
You need to replace the previous Seed Node RPC set in `drosera_rpc` field in `drosera.toml` with new Seed Node.
```bash
cd my-drosera-trap
```
```bash
nano drosera.toml
```
* Replace `https://seed-node.testnet.drosera.io` with new Seed Node: `https://relay.testnet.drosera.io`

![image](https://github.com/user-attachments/assets/589567b1-7b04-4514-af98-f721b29dfde9)


## 3. Verify Trap's Address
In your `drosera.toml` file, Verify that you have a line showing your trap's address as `address = "0x..."`

![image](https://github.com/user-attachments/assets/0db14076-1bbd-40fc-b18d-c785cc94776c)

If you don't see your trap's address as `address = "0x..."` line, then connet your missing trap's wallet to the [Dashboard](https://app.drosera.io/), find it under Trap's Config and Create a line like picture above in `drosera.toml` file.

![image](https://github.com/user-attachments/assets/5f70f788-4a99-4a6b-9f50-4ca5ce99073b)


## 4. Re-Apply Drosera Configurations
```bash
DROSERA_PRIVATE_KEY=xxx drosera apply
```
* Replace `xxx` with your trap's Private Key.
* Enter `ofc`, when prompted.

## 5. Open UDP ports
```console
# 1st operator
ufw allow 31313/udp
ufw allow 31314/udp

# 2nd operator
ufw allow 31315/udp
ufw allow 31316/udp
```


## 6. Re-Run Operator Node(s)
* Head to Operator's directory:
```bash
cd ~
```
```bash
cd Drosera-Network
```

* Stop Nodes:
```bash
docker compose down -v
```

* Restart Nodes:
```bash
docker compose up -d
```

* Node Logs:
```bash
docker compose logs -f
```
* You will get errors initially

![image](https://github.com/user-attachments/assets/c4af432a-cb30-412a-abe4-0e5d0fd5f6ac)

* After a few mintues, you'll get healthy logs:

![image](https://github.com/user-attachments/assets/418229a7-5462-46bd-b81f-a18996a3c822)

Then, Your operators will produce Green Blocks, bro. Congratz.
![image](https://github.com/user-attachments/assets/669b4048-3952-4079-95e1-58dd279e194c)


# Immortalize Discord username on-chain and Earn `Cadet` role!
Assuming your Trap is deployed and your operator is running, let's set up a new Trap to submit your Discord username on-chain and unlock an exclusive **Cadet** role

### 1. Create New Trap
1- Move to your trap directory:
```bash
cd my-drosera-trap
```
2- Create a new `Trap.sol` file:
```bash
nano src/Trap.sol
```
3- Paste the following contract code in it:
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ITrap} from "drosera-contracts/interfaces/ITrap.sol";

interface IMockResponse {
    function isActive() external view returns (bool);
}

contract Trap is ITrap {
    address public constant RESPONSE_CONTRACT = 0x4608Afa7f277C8E0BE232232265850d1cDeB600E;
    string constant discordName = "DISCORD_USERNAME"; // add your discord name here

    function collect() external view returns (bytes memory) {
        bool active = IMockResponse(RESPONSE_CONTRACT).isActive();
        return abi.encode(active, discordName);
    }

    function shouldRespond(bytes[] calldata data) external pure returns (bool, bytes memory) {
        // take the latest block data from collect
        (bool active, string memory name) = abi.decode(data[0], (bool, string));
        // will not run if the contract is not active or the discord name is not set
        if (!active || bytes(name).length == 0) {
            return (false, bytes(""));
        }

        return (true, abi.encode(name));
    }
}
```
* Replace `DISCORD_USERNAME` with your discord username.
* To save: `Ctrl+X`, `Y` & `Enter`

### 2. Edit `drosera.toml` config
```
nano drosera.toml
```
* Modify the values of the specified variables as follows:
* `path` = `"out/Trap.sol/Trap.json"`
* `response_contract` = `"0x4608Afa7f277C8E0BE232232265850d1cDeB600E"`
* `response_function` = `"respondWithDiscordName(string)"`

![image](https://github.com/user-attachments/assets/67b7cd71-0a01-49e7-aa69-540bd3d1f37d)

*  Your final `drosera.toml` file should match the example shown above.

### 3. Deploy Trap
1- Compile your Trap's Contract:
```
forge build
```
* If you got errors like: `command not found`, Enter `source /root/.bashrc` or reinstall dependecies from [here](https://github.com/0xmoei/Drosera-Network#1-configure-enviorments)

2- Test the trap before deploying:
```bash
drosera dryrun
```

3- Apply and Deploy the Trap:
```bash
DROSERA_PRIVATE_KEY=xxx drosera apply
```
* Replace `xxx` with your EVM wallet privatekey (Ensure it's funded with Holesky ETH)
* Enter the command, when prompted, write `ofc` and press `Enter`.

![image](https://github.com/user-attachments/assets/ddccd255-4288-4a8d-99a2-77ba1269089b)

### 4. Verify Trap can respond
After the trap is deployed, we can check if the user has responded by calling the `isResponder` function on the response contract.
```bash
cast call 0x4608Afa7f277C8E0BE232232265850d1cDeB600E "isResponder(address)(bool)" OWNER_ADDRESS --rpc-url https://ethereum-holesky-rpc.publicnode.com
```
* Replace `OWNER_ADDRESS` with your Trap's owner address. (Your main address that has deployed the Trap's contract)
* If you receive `true` as a response, it means you have successfully completed all the steps.

![image](https://github.com/user-attachments/assets/b6f89508-1ce4-46d6-8dcb-685ae7063d07)

* It may take a few minutes to successfully receive `true` as a response

### 5. Re-run Operator nodes
```
cd
cd Drosera-Network
```
```
docker compose up -d
```


# Troubleshooting Errors

## ⬜️⬜️ White Blocks for an Operator

![image](https://github.com/user-attachments/assets/7bf3bd34-d706-4c8a-8573-46d124258528)

* Use this `docker-compose.yaml` file instead.

```yaml
version: '3'
services:
  drosera1:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-node1
    network_mode: host
    volumes:
      - drosera_data1:/data
    command: node --db-file-path /data/drosera.db --network-p2p-port 31313 --server-port 31314 --eth-rpc-url RPC_URL_1 --eth-backup-rpc-url https://holesky.drpc.org --drosera-address 0xea08f7d533C2b9A62F40D5326214f39a8E3A32F8 --eth-private-key ${ETH_PRIVATE_KEY} --listen-address 0.0.0.0 --network-external-p2p-address ${VPS_IP} --disable-dnr-confirmation true
    restart: always

  drosera2:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-node2
    network_mode: host
    volumes:
      - drosera_data2:/data
    command: node --db-file-path /data/drosera.db --network-p2p-port 31315 --server-port 31316 --eth-rpc-url RPC_URL_2 --eth-backup-rpc-url https://holesky.drpc.org --drosera-address 0xea08f7d533C2b9A62F40D5326214f39a8E3A32F8 --eth-private-key ${ETH_PRIVATE_KEY2} --listen-address 0.0.0.0 --network-external-p2p-address ${VPS_IP} --disable-dnr-confirmation true
    restart: always

volumes:
  drosera_data1:
  drosera_data2:
```
* Replace `RPC_URL_1` & `RPC_URL_2` with your own Ethereum Holesky RPC, You can use [Alchemy](https://dashboard.alchemy.com/) or [QuickNode](https://dashboard.quicknode.com/) to create Ethereum Holesky RPC.
* You can use same RPCs for both `RPC_URL_1` & `RPC_URL_2`.
* To save: `CTRL`+`X`, `Y` & `ENTER`

**Re-run your Operators**:
```bash
cd ~/Drosera-Network
```
```bash
docker compose down -v
```
```bash
docker compose up -d
```

**Now Give it a few minutes. Boom!**

![image](https://github.com/user-attachments/assets/87a6eca8-5d77-44d6-84e8-f370200e02f3)



## 🚨 **Error**: Apply config failed
* You may get `Apply config failed` due to several issues:
![image](https://github.com/user-attachments/assets/326a931f-f107-4865-8d1b-add5a3735ce1)

* You need to verify if you have deployed any trap before.
* In your `drosera.toml` file, Verify that you have a line showing your trap's address as `address = "0x..."`

![image](https://github.com/user-attachments/assets/eb84d0c3-fb8d-43c5-aa00-c6e4422cebbd)

> If you don't see your trap's address as address = "0x..." line, Then follow the steps:

* Connect to the [Dashboard](https://app.drosera.io/) using same account you used to deploy your trap:

![image](https://github.com/user-attachments/assets/50b0b026-8a1d-492c-bb20-41ea1511e215)


* Select the **“Traps Owned”** button to display only traps that your account owns:

![image](https://github.com/user-attachments/assets/bc4550c6-b4a7-43ee-833f-fe179d9d5cf5)

* Click on the trap to go to the trap’s page:

![image](https://github.com/user-attachments/assets/e21666c7-fdc2-42f8-9ae9-7e5a5ca16267)

* On the trap’s page, click on the **“Trap Config”** address hyperlink. This will open a page to holesky etherscan.

![image](https://github.com/user-attachments/assets/09d1112c-8d7c-4d49-a4f2-01e660f01139)

* Click the button to copy the contract’s address:

![image](https://github.com/user-attachments/assets/9a8ecf32-a4f2-4da8-9b21-e3ddb2d977c8)

* Paste that address as the value for the corresponding trap’s address field in your drosera.toml file:

![image](https://github.com/user-attachments/assets/5934ce37-d144-4d0e-9084-8a789a6ebc01)
