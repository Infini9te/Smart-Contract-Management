# Smart-Contract-Management

## Meta ATM
This is a straightforward React component designed for a Meta ATM application. It enables users to link their MetaMask wallet, observe their account balance, make deposits and withdrawals of ETH, access the owner's name, and utilize a basic calculator with restricted operations.

## Functionalities
Connect to MetaMask wallet
Display user's account address
View user's account balance
Deposit ETH into the ATM
Withdraw ETH from the ATM
Check the owner's name
Divide two values
Subtract two values
Remainder of two values

## Commands

Inside the project directory, in the terminal type: npm i
Open two additional terminals in your VS code
In the second terminal type: npx hardhat node
In the third terminal, type: npx hardhat run --network localhost scripts/deploy.js
Back in the first terminal, type npm run dev to launch the front-end

## STEPS
### Assessment.sol:
   // SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.16;

//import "hardhat/console.sol";

contract Assessment {
    address payable public owner;
    uint256 public balance;
    // uint public b=5;
    event Deposit(uint256 amount);
    event Withdraw(uint256 amount);
    


    constructor(uint initBalance) payable {
        owner = payable(msg.sender);
        balance = initBalance;
    }

    function getBalance() public view returns(uint256){
        return balance;
    }

    function deposit(uint256 _amount) public payable {
        uint _previousBalance = balance;

        // make sure this is the owner
        require(msg.sender == owner, "You are not the owner of this account");

        // perform transaction
        balance += _amount;

        // assert transaction completed successfully
        assert(balance == _previousBalance + _amount);

        // emit the event
        emit Deposit(_amount);
    }

    // custom error
    error InsufficientBalance(uint256 balance, uint256 withdrawAmount);

    function withdraw(uint256 _withdrawAmount) public {
        require(msg.sender == owner, "You are not the owner of this account");
        uint _previousBalance = balance;
        if (balance < _withdrawAmount) {
            revert InsufficientBalance({
                balance: balance,
                withdrawAmount: _withdrawAmount
            });
        }

        // withdraw the given amount
        balance -= _withdrawAmount;

        // assert the balance is correct
        assert(balance == (_previousBalance - _withdrawAmount));

        // emit the event
        emit Withdraw(_withdrawAmount);
    }

    function checkOwner()public pure returns(string memory){
        string memory name="Shubham Negi";
        return name;
    }
    
    function division(uint a, uint b) public pure returns(uint){
        return a/b;
        
        // emit division((a+b));
    }

    function substraction(uint a, uint b) public pure returns(uint){
        require(a>=b,"value of a must me greater than b");
        return a-b;
    }

    function remainder(uint a, uint b) public pure returns(uint){
        return a%b;
    }
    
}
### Index.js:
import {useState, useEffect} from "react";
import {ethers} from "ethers";
import atm_abi from "../artifacts/contracts/Assessment.sol/Assessment.json";

export default function HomePage() {
  const [ethWallet, setEthWallet] = useState(undefined);
  const [account, setAccount] = useState(undefined);
  const [atm, setATM] = useState(undefined);
  const [balance, setBalance] = useState(undefined);
  const [ownerName, setOwnerName] = useState(undefined);
  const [div, setDiv] = useState(undefined);
  const [sub, setSub] = useState(undefined);
  const [rem, setRem] = useState(undefined);
  const [inputA, setInputA] = useState("");
  const [inputB, setInputB] = useState("");

  
  const contractAddress = "0x5FbDB2315678afecb367f032d93F642f64180aa3";

  const atmABI = atm_abi.abi;

  const getWallet = async() => {
    if (window.ethereum) {
      setEthWallet(window.ethereum);
      window.ethereum.on("accountsChanged", (accounts) => {
        handleAccount(accounts);
      });
    }

    if (ethWallet) {
      const accounts = await ethWallet.request({method: "eth_accounts"});
      handleAccount(accounts);
    }
  }

  const handleAccount = (account) => {
    if (account) {
      console.log ("Account connected: ", account);
      setAccount(account);
    }
    else {
      console.log("No account found");
    }
  }

  const connectAccount = async() => {
    if (!ethWallet) {
      alert('MetaMask wallet is required to connect');
      return;
    }
  
    const accounts = await ethWallet.request({ method: 'eth_requestAccounts' });
    handleAccount(accounts);
    
    // once wallet is set we can get a reference to our deployed contract
    getATMContract();
  };

  const getATMContract = () => {
    const provider = new ethers.providers.Web3Provider(ethWallet);
    const signer = provider.getSigner();
    const atmContract = new ethers.Contract(contractAddress, atmABI, signer);

    setATM(atmContract);
  }

  const getBalance = async() => {
    if (atm) {
      setBalance((await atm.getBalance()).toNumber());
    }
  }

  const deposit = async() => {
    if (atm) {
      let tx = await atm.deposit(1);
      await tx.wait()
      getBalance();
    }
  }

  const withdraw = async() => {
    if (atm) {
      let tx = await atm.withdraw(1);
      await tx.wait()
      getBalance();
    }
  }
  const checkOwner = async () => {
    if (atm) {
      let owner = await atm.checkOwner();
      setOwnerName("Shubham Negi");
    }
  }
  const division = async () => {
      if (atm) {
        const a = parseInt(inputA);
        const b = parseInt(inputB);
        const answer = await atm.division(a,b);
        setDiv(answer);
      }
  }  
  const subtraction = async () => {
    if (atm) {
      const a = parseInt(inputA);
      const b = parseInt(inputB);
      const answer = await atm.substraction(a,b);
      setSub(answer);
    }
  }
  const remainder = async () => {
    if (atm) {
      const a = parseInt(inputA);
      const b = parseInt(inputB);
      const answer = await atm.remainder(a,b);
      setRem(answer);
    }
  }
  const handleInputAChange = (event) => {
    setInputA(event.target.value);
  };

  const handleInputBChange = (event) => {
    setInputB(event.target.value);
  };

  
  const initUser = () => {
    // Check to see if user has Metamask
    if (!ethWallet) {
      return <p>Please install Metamask in order to use this ATM.</p>
    }

    // Check to see if user is connected. If not, connect to their account
    if (!account) {
      return <button onClick={connectAccount}>Please connect your Metamask wallet</button>
    }

    if (balance == undefined) {
      getBalance();
    }

    return (
      <>
        <div>
          <p style={{ fontFamily: "Sans-serif" }}>Your Account: {account}</p>
          <p style={{ fontFamily: "Sans-serif" }}>Your Balance: {balance}</p>
          <p style={{ fontFamily: "Sans-serif" }}>Owner Name: {ownerName}</p>
  
          <button style={{ backgroundColor: "red" }} onClick={deposit}>
            Deposit 1 ETH
          </button>
          <button style={{ backgroundColor: "yellow" }} onClick={withdraw}>
            Withdraw 1 ETH
          </button>
        </div>
  
        <div>
          <h2>Balancer</h2>
          <p style={{ fontFamily: "Sans-serif" }}>Div: {div ? div.toString() : ""}</p>
          <p style={{ fontFamily: "Sans-serif" }}>Sub: {sub ? sub.toString() : ""}</p>
          <p style={{ fontFamily: "Sans-serif" }}>Rem: {rem ? rem.toString() : ""}</p>

          <input
            type="number"
            placeholder="Enter value A"
            value={inputA}
            onChange={handleInputAChange}
          />
          <input
            type="number"
            placeholder="Enter value B"
            value={inputB}
            onChange={handleInputBChange}
          />
  
          <button style={{ backgroundColor: "yellow",
          color: "black",
          border: "2px solid #f2c94c",
          borderRadius: "8px",
          padding: "10px 20px",
          marginRight: "10px",
          cursor: "pointer",
          fontWeight: "bold", }} onClick={division}>
            Div
          </button>
          <button style={{backgroundColor: "grey",
          color: "white",
          border: "2px solid #4f4f4f",
          borderRadius: "8px",
          padding: "10px 20px",
          marginRight: "10px",
          cursor: "pointer",
          fontWeight: "bold", }} onClick={subtraction}>
            Sub
          </button>
          <button style={{backgroundColor: "red",
          color: "white",
          border: "2px solid #b22222",
          borderRadius: "8px",
          padding: "10px 20px",
          cursor: "pointer",
          fontWeight: "bold", }} onClick={remainder}>
            Rem
          </button>
        
      
        </div>
      </>
    );
    
  }

  useEffect(() => {
    getWallet();
    checkOwner();
  }, []);

  return (
    <main className="container">
      <header><h1>Meta ATM</h1></header>
      {initUser()}
      <style jsx>{`
        .container {
          text-align: center
        }
        
      `}
      </style>
    </main>
  )
}

### Make sure you have set chainId in hardhatconfig.ts.

## AUTHOR
### SHUBHAM
