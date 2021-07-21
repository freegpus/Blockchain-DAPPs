# Decentralized Instagram

NOTE: I'm using a 20.04.1-Ubuntu

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
  
  cd decentragram
  
  #install
  npm install
  ```

  

[Front End Dev](#Frontend-Web-Client-development)



### Backend Smart Contract Development

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

#truffle console to interact with the development blockchain 
truffle console
#fetch decentragram from blockchain and assign it to variable, asynchronus with await
> decentragram = await Decentragram.deployed()
#see JS version of smart contract
> decentragram
> decentragram.address
> name = await decentragram.name()
> name
```

**<u>Tests:</u>**

Get the mocha, chai assertion library. Mocha comes with truffle, get chai by

```bash
npm install chai

#run tests
truffle test
```

![tests](./screenshots/tests.png)

Testing is absolutely vital because smart contracts are IMMUTABLE

**<u>Creating images:</u>**

```js
//in Decentragram.sol add
  //store images
  mapping(uint => Image) public images;

  struct Image {
    uint id;
    string hash;
    string description;
    uint tipAmount;
    address payable author;
  }
  //create images
  function uploadImage() public {
    images[1] = Image(1, 'abc123', 'hello world', 0, address(0x0));
  }
```

Create a test for it:

```js
//in test.js, add
    describe('images', async() => {

      let result

      it('creates images', async() => {

        result = await decentragram.uploadImage()
        //have to call out each individual image by mapping, does not return entire array
        let image = await decentragram.images(1)
        console.log(image)

      })

    })
```

The mapping of key '1' will reference that first new struct of images. Image hash will eventually be from IPFS. 

![tests1](./screenshots/tests1.png)

Test passed so now we know we can make images. Let's move on to making them dynamically.



**<u>Create event for whenever image is uploaded:</u>**

At this point, our Decentragram.sol should look like:

```js
pragma solidity ^0.5.0;

contract Decentragram {
  // Code goes here...
  string public name = "Decentragram";

  //store images
  uint public imageCount = 0;
  mapping(uint => Image) public images;

  struct Image {
    uint id;
    string hash;
    string description;
    uint tipAmount;
    address payable author;
  }


  event ImageCreated(
    uint id,
    string hash,
    string description,
    uint tipAmount,
    address payable author
  );

  //create images
  function uploadImage(string memory _imgHash, string memory _description) public {
    //generate IDs dynamically for the mapping 
    imageCount++;

    //add image to contract, msg.sender will be the author/person who called the function
    images[imageCount] = Image(imageCount, _imgHash, _description, 0, msg.sender);
    
    // Trigger an event
    emit ImageCreated(imageCount, _imgHash, _description, 0, msg.sender);
  }

  //tip images
}
```

Now we can create tests for it. test.js should now look like:

```js
const Decentragram = artifacts.require('./Decentragram.sol')

require('chai')
  .use(require('chai-as-promised'))
  .should()

contract('Decentragram', ([deployer, author, tipper]) => {
  let decentragram

  before(async () => {
    decentragram = await Decentragram.deployed()
  })

  describe('deployment', async () => {
    it('deploys successfully', async () => {
      const address = await decentragram.address
      assert.notEqual(address, 0x0)
      assert.notEqual(address, '')
      assert.notEqual(address, null)
      assert.notEqual(address, undefined)
    })

    it('has a name', async () => {
      const name = await decentragram.name()
      assert.equal(name, 'Decentragram')
    })
})

  describe('images', async() => {

    let result, imageCount
    const hash = 'abc123'

    before(async () => {

    //from author is the metadata
    result = await decentragram.uploadImage(hash, 'Image description', { from: author })
    imageCount = await decentragram.imageCount()
    
    })

    it('creates images', async() => {

      //success
      assert.equal(imageCount, 1)
      console.log(result)
      
    })

  })

})
```

![tests2](./screenshots/tests2.png)



**<u>Tipping Images:</u>**

decentragram.sol smart contract should now look like:

```js
pragma solidity ^0.5.0;

contract Decentragram {
  // Code goes here...
  string public name = "Decentragram";

  //store images
  uint public imageCount = 0;
  mapping(uint => Image) public images;

  struct Image {
    uint id;
    string hash;
    string description;
    uint tipAmount;
    address payable author;
  }


  event ImageCreated(
    uint id,
    string hash,
    string description,
    uint tipAmount,
    address payable author
  );

   event ImageTipped(
    uint id,
    string hash,
    string description,
    uint tipAmount,
    address payable author
  );

  //create images
  function uploadImage(string memory _imgHash, string memory _description) public {
    
    // Make sure the image hash exists
    require(bytes(_imgHash).length > 0);

    // Make sure image description exists
    require(bytes(_description).length > 0);

    // Make sure uploader address exists
    require(msg.sender!= address(0x0));

    //generate IDs dynamically for the mapping 
    imageCount++;

    //add image to contract, msg.sender will be the author/person who called the function
    images[imageCount] = Image(imageCount, _imgHash, _description, 0, msg.sender);
    
    // Trigger an event
    emit ImageCreated(imageCount, _imgHash, _description, 0, msg.sender);
  }

  //tip images
  function tipImageOwner(uint _id) public payable {
      // Make sure the id is valid
      require(_id > 0 && _id <= imageCount);
      // Fetch the image from memory inside the function, not on blockchain
      Image memory _image = images[_id];
      // Fetch the author
      address payable _author = _image.author;
      // Pay the author by sending them Ether
      address(_author).transfer(msg.value);
      // Increment the tip amount
      _image.tipAmount = _image.tipAmount + msg.value;
      // Update the image
      images[_id] = _image;
      // Trigger an event
      emit ImageTipped(_id, _image.hash, _image.description, _image.tipAmount, _author);
    }

}
```

Test cases should now looks like:

```js
const Decentragram = artifacts.require('./Decentragram.sol')

require('chai')
  .use(require('chai-as-promised'))
  .should()

contract('Decentragram', ([deployer, author, tipper]) => {
  let decentragram

  before(async () => {
    decentragram = await Decentragram.deployed()
  })

  describe('deployment', async () => {
    it('deploys successfully', async () => {
      const address = await decentragram.address
      assert.notEqual(address, 0x0)
      assert.notEqual(address, '')
      assert.notEqual(address, null)
      assert.notEqual(address, undefined)
    })

    it('has a name', async () => {
      const name = await decentragram.name()
      assert.equal(name, 'Decentragram')
    })
})

  describe('images', async() => {

    let result, imageCount
    const hash = 'abc123'

    before(async () => {

    //from author is the metadata
    result = await decentragram.uploadImage(hash, 'Image description', { from: author })
    imageCount = await decentragram.imageCount()
    
    })

    it('creates images', async() => {

      //success
      assert.equal(imageCount, 1)
      const event = result.logs[0].args
      assert.equal(event.id.toNumber(), imageCount.toNumber(), 'id is correct')
      assert.equal(event.hash, hash, 'Hash is correct')
      assert.equal(event.description, 'Image description', 'description is correct')
      assert.equal(event.tipAmount, '0', 'tip amount is correct')
      assert.equal(event.author, author, 'author is correct')

      //see the actual event
      //console.log(result.logs[0].args)

      // FAILURE: Image must have hash
      await decentragram.uploadImage('', 'Image description', { from: author }).should.be.rejected;
      
      // FAILURE: Image must have description
      await decentragram.uploadImage('Image hash', '', { from: author }).should.be.rejected;
    })

    //check from Struct, fetch all images to make newsfeed
    it('lists images', async () => {
      const image = await decentragram.images(imageCount)
      assert.equal(image.id.toNumber(), imageCount.toNumber(), 'id is correct')
      assert.equal(image.hash, hash, 'Hash is correct')
      assert.equal(image.description, 'Image description', 'description is correct')
      assert.equal(image.tipAmount, '0', 'tip amount is correct')
      assert.equal(image.author, author, 'author is correct')
    })

    it('allows users to tip images', async () => {
      // Track the author balance before purchase
      let oldAuthorBalance

      //get balance of author
      oldAuthorBalance = await web3.eth.getBalance(author)
      oldAuthorBalance = new web3.utils.BN(oldAuthorBalance)

      //tipper account
      result = await decentragram.tipImageOwner(imageCount, { from: tipper, value: web3.utils.toWei('1', 'Ether') })

      // SUCCESS
      const event = result.logs[0].args
      assert.equal(event.id.toNumber(), imageCount.toNumber(), 'id is correct')
      assert.equal(event.hash, hash, 'Hash is correct')
      assert.equal(event.description, 'Image description', 'description is correct')
      assert.equal(event.tipAmount, '1000000000000000000', 'tip amount is correct')
      assert.equal(event.author, author, 'author is correct')

      // Check that author received funds
      let newAuthorBalance
      newAuthorBalance = await web3.eth.getBalance(author)
      newAuthorBalance = new web3.utils.BN(newAuthorBalance)

      let tipImageOwner
      tipImageOwner = web3.utils.toWei('1', 'Ether')
      tipImageOwner = new web3.utils.BN(tipImageOwner)

      const expectedBalance = oldAuthorBalance.add(tipImageOwner)

      assert.equal(newAuthorBalance.toString(), expectedBalance.toString())

      // FAILURE: Tries to tip a image that does not exist
      await decentragram.tipImageOwner(99, { from: tipper, value: web3.utils.toWei('1', 'Ether')}).should.be.rejected;
    })

  })

})
```

![tests3](./screenshots/tests3.png)



Now redeploy whole smart contract

```bash
truffle migrate --reset
```



### Frontend Web Client development



**<u>Starting Client:</u>**

```bash
npm run start

#open up web browser and navigate to localhost:3000
```

**<u>Building out Client with React:</u>**

App.js will be main file to modify. React is a modern web dev framework that's popular in the blockchain dev community. Component based, has state objects (mini database for front end) where we'll put our blockchain data. 

1. Connect the app to the blockchain using the web3.js library
2. Connect ganache to metamask and import 2 ganache accounts (for author and tipper)\
3. Store account in state objects
4. Get decentragram smart contract and create JS version of the smart contract from ABI data





