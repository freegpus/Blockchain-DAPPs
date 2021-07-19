# Decentralized Instagram

<u>**Dependencies**</u>:

- NodeJs & NPM

  ```bash
  sudo apt install nodejs
  sudo apt install npm
  ```

- Truffle framework (building ethereum smart contracts, etc.)

  ```bash
  npm install --g truffle@5.4.1
  ```

- Ganache (private test blockchain w/o using real crypto)

  ```bash
  Navigate to https://www.trufflesuite.com/ganache and download the appimage
  
  chmod +x ganache-2.5.4-linux-x86_64.AppImage
  ```

- Metamask

  ```
  Install from Google Chrome store
  ```

- Clone dapp univ's starter code

  ```bash
  git clone -b starter-code https://github.com/dappuniversity/decentragram decentragram
  ```

  

**<u>Starting Client:</u>**

```bash
npm run start

#open up web browser and navigate to localhost:3000
```



**<u>Creating the Smart Contracts:</u>**

```javascript
//Navigate to decentragram.sol and add in 
string public name = "Decentragram";
```



**<u>Create migration and deploy contract on ganache</u>**

```javascript
//navigate to deploy_contracts.js and add in
deployer.deploy(Migrations);
```

```bash
#migrate
truffle migrate --reset
```

