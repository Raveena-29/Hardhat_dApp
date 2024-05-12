# HARDHAT


## Technology Stack & Tools

- Solidity (Writing Smart Contracts & Tests)
- Javascript (React & Testing)
- [Hardhat](https://hardhat.org/) (Development Framework)
- [Ethers.js](https://docs.ethers.io/v5/) (Blockchain Interaction)
- [React.js](https://reactjs.org/) (Frontend Framework)
- [MetaMask](https://metamask.io/)

## Requirements For Initial Setup
- Install [NodeJS](https://nodejs.org/en/). Recommended to use the LTS version.
- Install [MetaMask](https://metamask.io/) on your browser.

## Setting Up
### 1. Clone/Download the Repository

### 2. Install Dependencies:
`$ npm install`

### 3. Run tests
`$ npx hardhat test`

### 4. Start Hardhat node
`$ npx hardhat node`

### 5. Run deployment script
In a separate terminal execute:
`$ npx hardhat run ./scripts/deploy.js --network localhost`

### 6. Start frontend
`$ npm run start`
## Setting up Environment
### Installing Node.js

### Copy and paste these commands in a terminal:
```
sudo apt-get install
sudo apt-get update
```
![dapp_1](https://github.com/Raveena-29/Hardhat_dApp/assets/148243757/43608e1d-3c10-440d-b5d7-959d90eeb0c5)

```
sudo apt install curl git
```
![dapp_2](https://github.com/Raveena-29/Hardhat_dApp/assets/148243757/d499b316-59eb-4e71-ad80-6c0ef155819b)

```
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
```
![dapp_3](https://github.com/Raveena-29/Hardhat_dApp/assets/148243757/bca356a0-9e38-4338-ba00-0a01c097754c)
```
sudo apt-get install -y nodejs
```
![dapp_4](https://github.com/Raveena-29/Hardhat_dApp/assets/148243757/5014f6e9-40f5-46be-ad6b-c97ad8879d34)

## Creating a Hardhat project
Installing Hardhat using the Node.js package manager (npm), which is both a package manager and an online repository for JavaScript code. <br>

### Open a new terminal and run these commands to create a new folder:
```
mkdir hardhat-tutorial
cd hardhat-tutorial
npm init
```
![dapp_5](https://github.com/Raveena-29/Hardhat_dApp/assets/148243757/46adef71-5e2c-4008-b610-c5eeda31174d)

### Now installing Hardhat:
```
npm install --save-dev hardhat
```
![dapp_6](https://github.com/Raveena-29/Hardhat_dApp/assets/148243757/faadf04d-3f1f-49d4-8895-cb677b8e7f9c)

### In the same directory where you installed Hardhat run:
```
npx hardhat init
```
**Then, Select `Create an empty hardhat.config.js` with your keyboard and hit enter**
![dapp_7](https://github.com/Raveena-29/Hardhat_dApp/assets/148243757/49b74afb-735b-446f-b448-72420e450dd3)

### Plugins
We are now going to use our recommended plugin, `@nomicfoundation/hardhat-toolbox`, which has everything you need for developing smart contracts.
**To install it, run this in your project directory:**
```
npm install --save-dev @nomicfoundation/hardhat-toolbox
```
**Add the highlighted line to your hardhat.config.js so that it looks like this:**
![dapp_8](https://github.com/Raveena-29/Hardhat_dApp/assets/148243757/c97a00ca-374c-48d0-9e87-111c8996ac3d)

## Writing and compiling smart contracts
### Writing smart contracts
**Start by creating a new directory called contracts and create a file inside the directory with .sol extension** <br>
**Paste the code below into the file**
![dapp_19](https://github.com/Raveena-29/Hardhat_dApp/assets/148243757/ed3fdeb4-ab5a-487e-a0cb-429e7be7d6d7)

### Compiling contracts
**To compile the contract run the below code in your terminal.**
```
npx hardhat compile
```
![dapp_10](https://github.com/Raveena-29/Hardhat_dApp/assets/148243757/408bac85-2e15-4e1f-a5ed-f982dfae5f8d)

## Testing contracts
### Writing tests
**Create a new directory called test inside our project root directory and create a new file in there with .js extension.**
```
const {
  time,
  loadFixture,
} = require("@nomicfoundation/hardhat-toolbox/network-helpers");
const { anyValue } = require("@nomicfoundation/hardhat-chai-matchers/withArgs");
const { expect } = require("chai");

describe("Lock", function () {
  // We define a fixture to reuse the same setup in every test.
  // We use loadFixture to run this setup once, snapshot that state,
  // and reset Hardhat Network to that snapshot in every test.
  async function deployOneYearLockFixture() {
    const ONE_YEAR_IN_SECS = 365 * 24 * 60 * 60;
    const ONE_GWEI = 1_000_000_000;

    const lockedAmount = ONE_GWEI;
    const unlockTime = (await time.latest()) + ONE_YEAR_IN_SECS;

    // Contracts are deployed using the first signer/account by default
    const [owner, otherAccount] = await ethers.getSigners();

    const Lock = await ethers.getContractFactory("Lock");
    const lock = await Lock.deploy(unlockTime, { value: lockedAmount });

    return { lock, unlockTime, lockedAmount, owner, otherAccount };
  }

  describe("Deployment", function () {
    it("Should set the right unlockTime", async function () {
      const { lock, unlockTime } = await loadFixture(deployOneYearLockFixture);

      expect(await lock.unlockTime()).to.equal(unlockTime);
    });

    it("Should set the right owner", async function () {
      const { lock, owner } = await loadFixture(deployOneYearLockFixture);

      expect(await lock.owner()).to.equal(owner.address);
    });

    it("Should receive and store the funds to lock", async function () {
      const { lock, lockedAmount } = await loadFixture(
        deployOneYearLockFixture
      );

      expect(await ethers.provider.getBalance(lock.target)).to.equal(
        lockedAmount
      );
    });

    it("Should fail if the unlockTime is not in the future", async function () {
      // We don't use the fixture here because we want a different deployment
      const latestTime = await time.latest();
      const Lock = await ethers.getContractFactory("Lock");
      await expect(Lock.deploy(latestTime, { value: 1 })).to.be.revertedWith(
        "Unlock time should be in the future"
      );
    });
  });

  describe("Withdrawals", function () {
    describe("Validations", function () {
      it("Should revert with the right error if called too soon", async function () {
        const { lock } = await loadFixture(deployOneYearLockFixture);

        await expect(lock.withdraw()).to.be.revertedWith(
          "You can't withdraw yet"
        );
      });

      it("Should revert with the right error if called from another account", async function () {
        const { lock, unlockTime, otherAccount } = await loadFixture(
          deployOneYearLockFixture
        );

        // We can increase the time in Hardhat Network
        await time.increaseTo(unlockTime);

        // We use lock.connect() to send a transaction from another account
        await expect(lock.connect(otherAccount).withdraw()).to.be.revertedWith(
          "You aren't the owner"
        );
      });

      it("Shouldn't fail if the unlockTime has arrived and the owner calls it", async function () {
        const { lock, unlockTime } = await loadFixture(
          deployOneYearLockFixture
        );

        // Transactions are sent using the first signer by default
        await time.increaseTo(unlockTime);

        await expect(lock.withdraw()).not.to.be.reverted;
      });
    });

    describe("Events", function () {
      it("Should emit an event on withdrawals", async function () {
        const { lock, unlockTime, lockedAmount } = await loadFixture(
          deployOneYearLockFixture
        );

        await time.increaseTo(unlockTime);

        await expect(lock.withdraw())
          .to.emit(lock, "Withdrawal")
          .withArgs(lockedAmount, anyValue); // We accept any value as `when` arg
      });
    });

    describe("Transfers", function () {
      it("Should transfer the funds to the owner", async function () {
        const { lock, unlockTime, lockedAmount, owner } = await loadFixture(
          deployOneYearLockFixture
        );

        await time.increaseTo(unlockTime);

        await expect(lock.withdraw()).to.changeEtherBalances(
          [owner, lock],
          [lockedAmount, -lockedAmount]
        );
      });
    });
  });
});
```

**In your terminal run**
```
npx hardhat test
```
You should see the following output:
![dapp_12](https://github.com/Raveena-29/Hardhat_dApp/assets/148243757/649691e3-249c-431e-8d79-40193ce00aa8)

## Deploying to a live network
- Let's create a new directory ignition inside the project root's directory.
- Then, create a directory named modules inside of the ignition directory.
- Then paste the following into a Lock.js file in that directory:
![dapp_13](https://github.com/Raveena-29/Hardhat_dApp/assets/148243757/cfc92273-5e15-43d3-8a72-8a8427f821eb)
- In terminal Run:
```
$ npx hardhat ignition deploy ./ignition/modules/Lock.js
```
![dapp_14](https://github.com/Raveena-29/Hardhat_dApp/assets/148243757/a74fef92-60c1-49d2-9aa5-c876f21fa14a)
## Hardhat Boilerplate Project
```
cd hardhat-boilerplate
npm install
```
![dapp_16](https://github.com/Raveena-29/Hardhat_dApp/assets/148243757/9117a0f2-d072-4378-8324-b527f4b26ecf)
```
npx hardhat node
```
![dapp_17](https://github.com/Raveena-29/Hardhat_dApp/assets/148243757/b9703e7c-015d-4c05-bb54-8c1ee2cfb949)
![dapp_18](https://github.com/Raveena-29/Hardhat_dApp/assets/148243757/f3c9a08c-62ff-46eb-a285-5627a5dbd386)

