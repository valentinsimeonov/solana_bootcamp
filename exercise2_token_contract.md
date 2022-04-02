**Solana Blockchain Developer Bootcamp Day 1 Exercises*****

Link for the Exercise:
https://docs.google.com/document/d/e/2PACX-1vSOgwdz9-vpBDwh3Epr3fdjzGyMWB1GHNT4H7YysNRyBFRJ0_qpcafgGcZUgNJLoyTH9IBVBaaInHsc/pub

**Exercise 2: Token Contract**

In this exercise we will create, deploy and interact with a solana program that takes your name as an input, and simply outputs a message to the program output. This exercise will teach you about the basics of Solana programs and accounts, as well as serialization/deserialization.



Initiating the project
First step is to create a new project using cargo. Enter the following command into the terminal in the folder containing all your projecst.  It will create the necessary folders and files for the token program


Optional: If you are still in the ‘gm-program’ folder, you will need to go back to a folder.


cd ..




cargo new token-program --lib





The next step is to initiate a new project in that folder with npm using the following command in the terminal. You can accept all default values

cd token-program
npm init -y



Next, we need to add all the required dependencies for our program. Replace the contents of your package.json file with the text below

{
 "name": "01-token-program",
 "version": "1.0.0",
 "description": "",
 "main": "index.js",
 "scripts": {
   "start": "ts-node ./client/main.ts",
   "start-with-test-validator": "start-server-and-test 'solana-test-validator --reset --quiet' http://localhost:8899/health start",
   "clean": "npm run clean:program-c && npm run clean:program-rust",
   "build:program-rust": "cargo build-bpf --manifest-path=./src/program-rust/Cargo.toml --bpf-out-dir=dist/program",
   "clean:program-rust": "cargo clean --manifest-path=./src/program-rust/Cargo.toml && rm -rf ./dist"
 },
 "dependencies": {
   "@solana/buffer-layout": "^4.0.0",
   "@solana/web3.js": "^1.7.0",
   "borsh": "^0.7.0",
   "buffer": "^6.0.3",
   "mz": "^2.7.0",
   "yaml": "^1.10.2"
 },
 "devDependencies": {
   "@tsconfig/recommended": "^1.0.1",
   "@types/eslint": "^8.2.2",
   "@types/eslint-plugin-prettier": "^3.1.0",
   "@types/mz": "^2.7.2",
   "@types/prettier": "^2.1.5",
   "@types/yaml": "^1.9.7",
   "@typescript-eslint/eslint-plugin": "^4.6.0",
   "@typescript-eslint/parser": "^4.6.0",
   "eslint": "^7.12.1",
   "eslint-config-prettier": "^6.15.0",
   "eslint-plugin-prettier": "^4.0.0",
   "prettier": "^2.1.2",
   "start-server-and-test": "^1.11.6",
   "ts-node": "^10.0.0",
   "typescript": "^4.0.5"
 },
 "engines": {
   "node": ">=14.0.0"
 },
 "author": "",
 "license": "ISC"
}


Next, we need to install all the listed dependencies above. Enter the following into the command line:

npm install



Next step is to configure the Cargo.toml file with the correct values. Replace the contents of the file with this:

[package]
name = "token-program"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
[features]
no-entrypoint = []

[dependencies]
borsh = "0.9.1"
borsh-derive = "0.9.1"
solana-program = "=1.7.9"
thiserror = "1.0"

[dev-dependencies]
solana-program-test = "=1.7.9"
solana-sdk = "=1.7.9"

[lib]
name = "token_program"
crate-type = ["cdylib", "lib"]



Now we need to tell the Solana CLI that we want to use a local cluster.
solana config set --url localhost


Now we can create a new CLI Keypair. We will use this for interacting with our local cluster.  Note If you get an error stating ‘Refusing to overwrite without –force flag’, you can skip this step, as you’re simply using the key created from the previous exercise.

solana-keygen new


Because we want to run a local cluster in our existing VS Code terminal, you can open up a new terminal using the ‘+’ button near the bottom right of VS Code, then use a second terminal for starting the valiator, leaving your first terminal for entering commands::




Next step is to start our local cluster. This step can be skipped if you still have your local validator running from the previous exercise. Note, you may need to do some system config and possibly restart your machine to get the local cluster working. Type the following command into the terminal:

solana-test-validator





If you can’t get the local validator running, then you can just use the public Devnet network. You can configure your setup as follows. Please do not run this command if you are able to successfully start the local validator.


solana config set --url https://api.devnet.solana.com
solana airdrop 2 


You’re now ready to start creating the program


Creating the Program
In this section, we’ll create a new Rust program that acts as a simplified version for the SPL Token Program. The SPL Token Program is a built-in Solana program that defines the standard for tokens on Solana.


The first step is to create the other required rust files in the ‘src’ folder. Using the new file icon, create the following files in the ‘src’ folder:
entrypoint.rs
instruction.rs
processor.rs
state.rs

The end result should be a folder and file structure like the image below:




The next step is to create the contents of the lib.rs file. This code will simply define all the modules required for the program. Each module is contained in it’s own separate file:

pub mod entrypoint;
pub mod instruction;
pub mod processor;
pub mod state;


 

The next step is to populate the contents of the entrypoint.rs file. The code here performs the following:
Defines the program as a Solana program
Defines the entrypoint for the program as the ‘process_instruction’ function
Creates the process_instrunction function, which simply calls the process_instruction instruction from the Processor implementation, passing in all the input parameters received



use crate::processor::Processor;

use solana_program::{
   account_info::{AccountInfo},
   entrypoint,
   entrypoint::ProgramResult,
   msg,
   pubkey::Pubkey,
};

// Declare and export the program's entrypoint
entrypoint!(process_instruction);


fn process_instruction(
   program_id: &Pubkey,
   accounts: &[AccountInfo],
   instruction_data: &[u8],
) -> ProgramResult {
   msg!(
       "process_instruction: {}: {} accounts, data={:?}",
       program_id,
       accounts.len(),
       instruction_data
   );

   Processor::process_instruction(program_id, accounts, instruction_data)
}



The next step is to populate the contents of the instruction.rs file. The code here defines an enum that defines all the possible instructions that can be sent to the program:
0 - Create a new token
1 - Create a new token account
2 - Mint some tokens to a token account
3 - Transfer tokens between token accounts

Take note that the Mint and Transfer values can contain an additional piece of data called ‘amount’




use borsh::{BorshDeserialize, BorshSerialize};

#[derive(BorshSerialize, BorshDeserialize, Debug, Clone)]
pub enum TokenInstruction {
   CreateToken,
   CreateTokenAccount,
   Mint { amount: u64 },
   Transfer { amount: u64 },
}


The next step is to populate the contents of the state.rs file. The code here defines the data that will be stored for the program, which is split across two structs, a header level ‘Token’ struct that defines header level information about the token, as well as a ‘TokenAccount’ struct, that defines information about an account for the specified token. In addition to this, there are some helper functions included for each struct to help the program retrieve and save data

use borsh::{BorshDeserialize, BorshSerialize};
use solana_program::{
   account_info::AccountInfo, entrypoint::ProgramResult, program_error::ProgramError,
   pubkey::Pubkey,
};

#[derive(BorshSerialize, BorshDeserialize, Debug, Clone)]
pub struct Token {
   pub authority: Pubkey,
   pub supply: u64,
}

impl Token {
   pub fn load_unchecked(ai: &AccountInfo) -> Result<Self, ProgramError> {
       Ok(Self::try_from_slice(&ai.data.borrow())?)
   }
   pub fn save(&self, ai: &AccountInfo) -> ProgramResult {
       Ok(self.serialize(&mut *ai.data.borrow_mut())?)
   }

   pub fn load(ai: &AccountInfo) -> Result<Self, ProgramError> {
       let token = Self::try_from_slice(&ai.data.borrow())?;
       Ok(token)
   }
}

#[derive(BorshSerialize, BorshDeserialize, Debug, Clone)]
pub struct TokenAccount {
   pub owner: Pubkey,
   pub token: Pubkey,
   pub amount: u64,
}

impl TokenAccount {
   pub fn load_unchecked(ai: &AccountInfo) -> Result<Self, ProgramError> {
       Ok(Self::try_from_slice(&ai.data.borrow())?)
   }
   pub fn save(&self, ai: &AccountInfo) -> ProgramResult {
       Ok(self.serialize(&mut *ai.data.borrow_mut())?)
   }

   pub fn load(ai: &AccountInfo) -> Result<Self, ProgramError> {
       let account = Self::try_from_slice(&ai.data.borrow())?;
       Ok(account)
   }
}


The final step is to populate the contents of the processor.rs file. The top section defines it as a Solana program, and tells the compiler we want to use the Borsh libraries, as well as the instruction and state modules that we created earlier. In addition to this, it defines a process_instruction function, which takes in the standard Solana Rust parameters, it retrieves the passed in instruction via the instruction_data parameter, then depending on what value it has (0,1,2,3), it calls different logic to perform the specified function. We’ll fill in details for each instruction next

use borsh::{BorshDeserialize};
use solana_program::{
   account_info::{next_account_info, AccountInfo},
   entrypoint::ProgramResult,
   msg,
   program_error::ProgramError,
   pubkey::Pubkey,
};

use crate::instruction::TokenInstruction;
use crate::state::{Token, TokenAccount};

pub struct Processor {}

impl Processor {
   pub fn process_instruction(
       _program_id: &Pubkey,
       accounts: &[AccountInfo],
       instruction_data: &[u8],
   ) -> ProgramResult {
       let instruction = TokenInstruction::try_from_slice(instruction_data)
           .map_err(|_| ProgramError::InvalidInstructionData)?;
       let accounts_iter = &mut accounts.iter();
       msg!("Instruction: {:?}",instruction);
       match instruction {
           TokenInstruction::CreateToken => {
               msg!("Instruction: Create Token");

           }
           TokenInstruction::CreateTokenAccount => {
               msg!("Instruction: Create Token Account");

           }
           TokenInstruction::Mint { amount } => {
               msg!("Instruction: Mint");

           }
           TokenInstruction::Transfer { amount } => {
               msg!("Instruction: Transfer");
           }
       }
       Ok(())
   }
}


Now that we’ve defined the main body of our function, let's complete the logic for each branch of the match statement. The first is ‘CreateToken’, in this section we simply read in from the accounts array a new account to create a token for, as well as a ‘token authority’ account that acts as an owner of the token, and has authority to do things like mint new tokens etc.  Then we simply set some default values for the token such as the supply, set the authority/owner to the passed in token authority account, then we save the token data into the passed in token master account.

                //get account info for master token account
               let token_master_account = next_account_info(accounts_iter)?;
               let token_authority = next_account_info(accounts_iter)?;
               let mut token = Token::load_unchecked(token_master_account)?;

               //set default values and save master token account
               token.authority = *token_authority.key;
               token.supply = 0;
               token.save(token_master_account)?


The next step is to fill out the logic for the ‘CreateTokenAccount’ section. The code takes in three accounts from the accounts array:
New account to create a token account for
The master token account that we want to create a token account under
The owner of the new token account that we’re creating

The program then sets the relevant owner and master token values, as well as sets the initial balance to 0, then saves the data in the passed in new token account


                //get account info for master token account and token account to be created
               let token_account_acct = next_account_info(accounts_iter)?;
               let token_master_account = next_account_info(accounts_iter)?;
               let owner = next_account_info(accounts_iter)?;
               let mut token_account = TokenAccount::load_unchecked(token_account_acct)?;

               //set default values and save token account
               token_account.owner = *owner.key;
               token_account.token = *token_master_account.key;
               token_account.amount = 0;
               token_account.save(token_account_acct)?



The next section to fill in is the Mint branch. In this part, the program looks for three accounts in the accounts array:
The token account that wants to receive the minted tokens
The master token account of the tokens that we want to mint
The token authority account, that has access to mint new tokens

The logic then does some basic validation to ensure that the passed in token authority account is the one that signed the transaction, otherwise it returns an error. After this check passes, it simply increases the total supply in the master token account, and then increases the balance of the token in the specified token account by the passed in value, and saves the state of the accounts


                 //get account info for master token account and token account to mint to
                let token_account_acct = next_account_info(accounts_iter)?;
                let token_master_account = next_account_info(accounts_iter)?;
                let mut token_account = TokenAccount::load(token_account_acct)?;
                let mut token = Token::load(token_master_account)?;

                //basic validation, ensure its the master token authority trying to mint
                let token_authority = next_account_info(accounts_iter)?;
                if !token_authority.is_signer {
                    msg!("Only the token owner can mint tokens");
                    return Err(ProgramError::MissingRequiredSignature);
                }

                //update total supply of the master token, and update balance of token account that received the mint
                token.supply += amount;
                token_account.amount += amount;

                //save updated contents of both accounts
                token_account.save(token_account_acct)?;
                token.save(token_master_account)?;


The final part of the logic that needs completing is the Transfer section. In this part of the program it looks for three accounts in the accounts array:
The token account that is sending funds
The token account receiving funds
The owner of the token account sending funds

The logic then performs some basic validation on the passed in accounts, ensuring the sender has enough funds, and that they are the one that signed the transaction. Once these checks pass, it simply updates the balances in the sender and receiver token accounts, and saves the new state



     

          //get account info for from and to token accounts, as well as master token account
               let from_token_acct = next_account_info(accounts_iter)?;
               let to_token_acct = next_account_info(accounts_iter)?;
               let owner = next_account_info(accounts_iter)?;
               let mut src_token_account = TokenAccount::load(from_token_acct)?;
               let mut dst_token_account = TokenAccount::load(to_token_acct)?;

               //basic validation, ensure sender has enough funds
               if src_token_account.amount <= amount {
                   msg!("Not enough tokens to transfer");
                   return Err(ProgramError::InsufficientFunds);
               }

               //ensure the owner of the from account is the one signing the transaction
               if !owner.is_signer {
                   msg!("Not the token owner signing the transaction");
                   return Err(ProgramError::MissingRequiredSignature);
               }

               //ensure the owner passed in is the actual owner of the token account
               if !(src_token_account.owner == *owner.key) {
                   msg!("Not the token account owner signing the transaction");
                   return Err(ProgramError::MissingRequiredSignature);
               }

               //update values in from and to accounts, then save new contents of both accounts
               src_token_account.amount -= amount;
               dst_token_account.amount += amount;
               src_token_account.save(from_token_acct)?;
               dst_token_account.save(to_token_acct)?;


Awesome work! We’re now ready to build and deploy the program to your local Solana cluster



Building and Deploying the Program
Use the following command to build the program. You should see a similar output:

cargo build-bpf --manifest-path=./Cargo.toml --bpf-out-dir=dist/program





Now we’re ready to deploy our program to the localnet cluster (or the Devnet public network if you’re not running a local cluster):

solana program deploy dist/program/token_program.so


Great work, you’ve deployed your program to your local cluster! Now let’s create a client to interact with it.

Creating the Client
First, let's create a new folder to store all our client code. Create a new folder in VS code under your ‘token-program’ folder, and call it ‘client’




Inside the client folder, create the following files with the new file icon:
token_program.ts
main.ts
utils.ts






Enter the following code into main.ts. This will simply act as an entry point into the client, and then call all the functions required in the token_program file. Be sure to save your file once it’s modified.

import {
   establishConnection,
   establishPayer,
   checkProgram,
   createToken,
   createTokenAccounts,
   mint,
   transfer
 } from './token_program';

 async function main() {
   console.log("Let's create a token...");

   // Establish connection to the cluster
   await establishConnection();

   // Determine who pays for the fees
   await establishPayer();

   // Check if the program has been deployed
   await checkProgram();

   // Create the master token
   await createToken();

   // Create two accounts to mint and receive tokens
   await createTokenAccounts();

   // mint some tokens to one of the two accounts
   await mint();

   // Send some tokens from the account that recieved the mint to the second account
   await transfer();

   console.log('Success');
 }

 main().then(
   () => process.exit(),
   err => {
     console.error(err);
     process.exit(-1);
   },
 );


Next, enter the following code into utils.ts. These helper functions will be used to get config stored locally, get the RPC URL required to connect to a cluster, as well as generate a new keypair to be used for interacting with the deployed program. Be sure to save your file once it’s modified.

Note: If you aren’t running a local validator and are deploying to the public Devnet, please replace the ‘http://127.0.0.1:8899’ string in the getRpcUrl() function with ‘https://api.devnet.solana.com’


import os from 'os';
import fs from 'mz/fs';
import path from 'path';
import yaml from 'yaml';
import {Keypair} from '@solana/web3.js';

/**
* @private
*/
async function getConfig(): Promise<any> {
 // Path to Solana CLI config file
 const CONFIG_FILE_PATH = path.resolve(
   os.homedir(),
   '.config',
   'solana',
   'cli',
   'config.yml',
 );
 const configYml = await fs.readFile(CONFIG_FILE_PATH, {encoding: 'utf8'});
 return yaml.parse(configYml);
}

/**
* Load and parse the Solana CLI config file to determine which RPC url to use
*/
export async function getRpcUrl(): Promise<string> {
   return 'http://127.0.0.1:8899';
}

/**
* Load and parse the Solana CLI config file to determine which payer to use
*/
export async function getPayer(): Promise<Keypair> {
 try {
   const config = await getConfig();
   if (!config.keypair_path) throw new Error('Missing keypair path');
   return await createKeypairFromFile(config.keypair_path);
 } catch (err) {
   console.warn(
     'Failed to create keypair from CLI config file, falling back to new random keypair',
   );
   return Keypair.generate();
 }
}

/**
* Create a Keypair from a secret key stored in file as bytes' array
*/
export async function createKeypairFromFile(
 filePath: string,
): Promise<Keypair> {
 const secretKeyString = await fs.readFile(filePath, {encoding: 'utf8'});
 const secretKey = Uint8Array.from(JSON.parse(secretKeyString));
 return Keypair.fromSecretKey(secretKey);
}

 

Lastly, enter the following code into your token_program.ts. Be sure to save your file once it’s modified. This is the main part of the client, and performs the following functionality:
Establishes a connection to the cluster
Establishes which account will be paying for the generated transactions
Ensures the token program has been deployed to the cluster
Creates a new master token
Creates two accounts and associated token accounts to be used for minting and sending tokens
Mints some tokens to a specified token account
Transfers some tokens from one token account to another


import {
   Keypair,
   Connection,
   PublicKey,
   LAMPORTS_PER_SOL,
   SystemProgram,
   TransactionInstruction,
   Transaction,
   sendAndConfirmTransaction,
} from '@solana/web3.js';

import fs from 'mz/fs';
import path from 'path';
import * as borsh from 'borsh';
import { Buffer } from 'buffer';
import { getPayer, getRpcUrl, createKeypairFromFile } from './utils';

/**
* Connection to the network
*/
let connection: Connection;

/**
* Keypair associated to the fees' payer
*/
let payer: Keypair;

/**
* Token program id
*/
let programId: PublicKey;

/**
* The public key of the account that stores the token info
*/
let tokenPubkey: PublicKey;
let tokenFromAccountPubkey: PublicKey;
let tokenToAccountPubkey: PublicKey;
let tokenAccountPubkey: PublicKey;

/**
* Path to program files
*/
const PROGRAM_PATH = path.resolve(__dirname, '../dist/program');

/**
* Path to program shared object file which should be deployed on chain.
* This file is created when running either:
*   - `npm run build:program-c`
*   - `npm run build:program-rust`
*/
const PROGRAM_SO_PATH = path.join(PROGRAM_PATH, 'token_program.so');

/**
* Path to the keypair of the deployed program.
* This file is created when running `solana program deploy dist/program/gm_program.so`
*/
const PROGRAM_KEYPAIR_PATH = path.join(PROGRAM_PATH, 'token_program-keypair.json');

const TOKEN_NAME = 'GLASS_COIN'
const FROM_ACCT_SEED='FROM_ACCT_SEED'
const TO_ACCT_SEED='TO_ACCT_SEED'

/**
* Borsh class and schema definition for accounts
*/


class TokenInstruction {
   instruction = 0
   amount = 0
   constructor(fields: { instruction: number, amount: number } | undefined = undefined) {
       if (fields) {
           this.instruction = fields.instruction;
           this.amount = fields.amount;
       }
   }
   static schema = new Map([[TokenInstruction,
       {
           kind: 'struct',
           fields: [
               ['instruction', 'u8'],
               ['amount', 'u64']]
       }]]);
}

class TokenInstructionNoAmount {
   instruction = 0
   constructor(fields: { instruction: number } | undefined = undefined) {
       if (fields) {
           this.instruction = fields.instruction;
       }
   }
   static schema = new Map([[TokenInstructionNoAmount,
       {
           kind: 'struct',
           fields: [
               ['instruction', 'u8']]
       }]]);
}

class Token {
   authority = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
   supply = 0
   constructor(fields: { authority: [32], supply: number } | undefined = undefined) {
       if (fields) {
           this.authority = fields.authority;
           this.supply = fields.supply;
       }
   }
   static schema = new Map([[Token,
       {
           kind: 'struct',
           fields: [
               ['authority', [32]],
               ['supply', 'u64']]
       }]]);
}


class TokenAccount {
   owner = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
   token = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
   amount = 0
   constructor(fields: { owner: [32], token: [32], amount: number } | undefined = undefined) {
       if (fields) {
           this.owner = fields.owner;
           this.token = fields.token;
           this.amount = fields.amount;
       }
   }
   static schema = new Map([[TokenAccount,
       {
           kind: 'struct',
           fields: [
               ['owner', [32]],
               ['token', [32]],
               ['amount', 'u64']]
       }]]);
}

/**
* The expected size of each greeting account. Used for creating the buffer
*/
const NEW_TOKEN_SIZE = borsh.serialize(
   Token.schema,
   new Token())
   .length;

const TOKEN_ACCOUNT_SIZE = borsh.serialize(
   TokenAccount.schema,
   new TokenAccount())
   .length;

/**
* Establish a connection to the cluster
*/
export async function establishConnection(): Promise<void> {
   console.log('getting connection')
   const rpcUrl = await getRpcUrl();
   connection = new Connection(rpcUrl, 'confirmed');
   const version = await connection.getVersion();
   console.log('Connection to cluster established:', rpcUrl, version);
}

/**
* Establish an account to pay for creating the new token and performing transactions
*/
export async function establishPayer(): Promise<void> {
   let fees = 0;
   if (!payer) {
       const { feeCalculator } = await connection.getRecentBlockhash();

       // Calculate the cost to fund the greeter account
       fees += await connection.getMinimumBalanceForRentExemption(NEW_TOKEN_SIZE);

       // Calculate the cost of sending transactions
       fees += feeCalculator.lamportsPerSignature * 100; // wag

       payer = await getPayer();
   }

   let lamports = await connection.getBalance(payer.publicKey);
   if (lamports < fees) {
       // If current balance is not enough to pay for fees, request an airdrop
       const sig = await connection.requestAirdrop(
           payer.publicKey,
           fees - lamports,
       );
       await connection.confirmTransaction(sig);
       lamports = await connection.getBalance(payer.publicKey);
   }

   console.log(
       'Using account',
       payer.publicKey.toBase58(),
       'containing',
       lamports / LAMPORTS_PER_SOL,
       'SOL to pay for fees',
   );
}

/**
* Check if the token BPF program has been deployed
*/
export async function checkProgram(): Promise<void> {
   // Read program id from keypair file
   try {
       const programKeypair = await createKeypairFromFile(PROGRAM_KEYPAIR_PATH);
       programId = programKeypair.publicKey;
   } catch (err) {
       const errMsg = (err as Error).message;
       throw new Error(
           `Failed to read program keypair at '${PROGRAM_KEYPAIR_PATH}' due to error: ${errMsg}. Program may need to be deployed with \`solana program deploy dist/program/token_program.so\``,
       );
   }

   // Check if the program has been deployed
   const programInfo = await connection.getAccountInfo(programId);
   if (programInfo === null) {
       if (fs.existsSync(PROGRAM_SO_PATH)) {
           throw new Error(
               'Program needs to be deployed with `solana program deploy dist/program/token_program.so`',
           );
       } else {
           throw new Error('Program needs to be built and deployed');
       }
   } else if (!programInfo.executable) {
       throw new Error(`Program is not executable`);
   }
   console.log(`Using program ${programId.toBase58()}`);
   console.log('-----------------------------------------------------------------------------------------------------------------')
}

/**
* Say GM
*/
export async function createToken(): Promise<void> {

   // First we'll check to see if the master token account has been created aready, and if not we'll create it
   tokenPubkey = await PublicKey.createWithSeed(
       payer.publicKey,
       TOKEN_NAME,
       programId,
   );

   const masterTokenAccount = await connection.getAccountInfo(tokenPubkey);

   if (masterTokenAccount === null) {
       //master token doesn't exist, create it

       console.log(
           'Creating account',
           tokenPubkey.toBase58(),
           'for our new token',
       );
       const lamports = await connection.getMinimumBalanceForRentExemption(
           NEW_TOKEN_SIZE,
       );

       const transaction = new Transaction().add(
           SystemProgram.createAccountWithSeed({
               fromPubkey: payer.publicKey,
               basePubkey: payer.publicKey,
               seed: TOKEN_NAME,
               newAccountPubkey: tokenPubkey,
               lamports,
               space: NEW_TOKEN_SIZE,
               programId,
           }),
       );
       await sendAndConfirmTransaction(connection, transaction, [payer]);


       console.log('Creating token ', TOKEN_NAME, ' with key ', tokenPubkey.toBase58());

       // Create new master token
       //first we serialize the name data.
       let tokenInstruction = new TokenInstructionNoAmount({ "instruction": 0}) //instruction is 0 (create token)
       let data = borsh.serialize(TokenInstructionNoAmount.schema, tokenInstruction);
       let dataBuffer = Buffer.from(data)

       //now we generate the instruction
       const instruction = new TransactionInstruction({
           keys: [
               { pubkey: tokenPubkey, isSigner: false, isWritable: true },
               { pubkey: payer.publicKey, isSigner: false, isWritable: false },
           ],
           programId,
           data: dataBuffer
       });
       await sendAndConfirmTransaction(
           connection,
           new Transaction().add(instruction),
           [payer],
       );

       console.log('Token successfully created at address ', tokenPubkey.toBase58())

   } else {
       console.log('Master Token already created at address ', tokenPubkey.toBase58())
   }
   console.log('-----------------------------------------------------------------------------------------------------------------')
}

export async function createNewKeyPair(seed: string): Promise<PublicKey> {

   console.log('Creating new keypair for seed: ', seed);

   //first we create the account and see if it exists already on-chain
   tokenAccountPubkey = await PublicKey.createWithSeed(
       payer.publicKey,
       seed,
       programId,
   );

   //only create account if it doesn't already exist
   const tokenAcct = await connection.getAccountInfo(tokenAccountPubkey);
   if (tokenAcct === null) {

       const lamportsTokenAccount = await connection.getMinimumBalanceForRentExemption(
           TOKEN_ACCOUNT_SIZE,
       );

       //build up the instruction to create the account
       const transaction = new Transaction().add(
           SystemProgram.createAccountWithSeed({
               fromPubkey: payer.publicKey,
               basePubkey: payer.publicKey,
               seed: seed,
               newAccountPubkey: tokenAccountPubkey,
               lamports: lamportsTokenAccount,
               space: TOKEN_ACCOUNT_SIZE,
               programId,
           }),
       );
       await sendAndConfirmTransaction(
           connection,
           transaction,
           [payer],
       );

       console.log('created account for Public Key ', tokenAccountPubkey.toBase58())


       //now that we've created the account, we can register is as a token account
       await createTokenAccount(tokenAccountPubkey)

   } else {
       console.log('Token Account ', tokenAccountPubkey.toBase58(), ' already exists, skipping creation')
   }


   return tokenAccountPubkey
}


export async function createTokenAccounts(): Promise<void> {

   //first we need to create two public keys for the from and to accounts
   console.log('Creating from and to accounts for token ', tokenPubkey.toBase58());
   tokenFromAccountPubkey = await createNewKeyPair(FROM_ACCT_SEED)
   tokenToAccountPubkey = await createNewKeyPair(TO_ACCT_SEED)

   console.log('-----------------------------------------------------------------------------------------------------------------')
}

export async function createTokenAccount(tokenKey: PublicKey): Promise<void> {

   //first we serialize the instruction data
   let tokenInstruction = new TokenInstructionNoAmount({ "instruction": 1 })  //1 = create token account
   let data = borsh.serialize(TokenInstructionNoAmount.schema, tokenInstruction);
   let dataBuffer = Buffer.from(data)

   //now we build up the instruction
   const instruction = new TransactionInstruction({
       keys: [
           { pubkey: tokenKey, isSigner: false, isWritable: true },
           { pubkey: tokenPubkey, isSigner: false, isWritable: true },
           { pubkey: payer.publicKey, isSigner: false, isWritable: false }
       ],
       programId,
       data: dataBuffer
   });
   await sendAndConfirmTransaction(
       connection,
       new Transaction().add(instruction),
       [payer],
   );


   console.log('Token Account created for ', tokenKey.toBase58())
}



export async function mint(): Promise<void> {
   const MINT_AMOUNT = 100
   console.log('Minting ', MINT_AMOUNT, 'tokens of ', TOKEN_NAME, ' with key ', tokenPubkey.toBase58(), ' to account ', tokenFromAccountPubkey.toBase58());

   //first we serialize the instruction data
   let tokenMintInstruction = new TokenInstruction({ "instruction": 2, "amount": MINT_AMOUNT })  //2 = mint tokens to account
   let data = borsh.serialize(TokenInstruction.schema, tokenMintInstruction);
   let dataBuffer = Buffer.from(data)


   //now we build up the instruction
   const instruction = new TransactionInstruction({
       keys: [
           { pubkey: tokenFromAccountPubkey, isSigner: false, isWritable: true },
           { pubkey: tokenPubkey, isSigner: false, isWritable: true },
           { pubkey: payer.publicKey, isSigner: false, isWritable: false }
       ],
       programId,
       data: dataBuffer
   });
   await sendAndConfirmTransaction(
       connection,
       new Transaction().add(instruction),
       [payer],
   );


   console.log('getting account info for ', tokenFromAccountPubkey.toBase58())
   await getAccountTokenInfo(tokenFromAccountPubkey)
   console.log('-----------------------------------------------------------------------------------------------------------------')
}


export async function transfer(): Promise<void> {
   const TRANSFER_AMOUNT = 5
   console.log('Transferring', TRANSFER_AMOUNT, 'of', tokenPubkey.toBase58(), 'tokens from', tokenFromAccountPubkey.toBase58(), 'to', tokenToAccountPubkey.toBase58());

   //first we serialize the instruction data
   let tokenTransferInstruction = new TokenInstruction({ "instruction": 3, "amount": TRANSFER_AMOUNT })  //3 = transfer tokens
   let data = borsh.serialize(TokenInstruction.schema, tokenTransferInstruction);
   let dataBuffer = Buffer.from(data)


   //now we build up the instruction
   const instruction = new TransactionInstruction({
       keys: [
           { pubkey: tokenFromAccountPubkey, isSigner: false, isWritable: true },
           { pubkey: tokenToAccountPubkey, isSigner: false, isWritable: true },
           { pubkey: payer.publicKey, isSigner: false, isWritable: false }
       ],
       programId,
       data: dataBuffer
   });
   await sendAndConfirmTransaction(
       connection,
       new Transaction().add(instruction),
       [payer],
   );
   await getAccountTokenInfo(tokenFromAccountPubkey)
   await getAccountTokenInfo(tokenToAccountPubkey)
   console.log('-----------------------------------------------------------------------------------------------------------------')

}

// Helper function to print the balance of a token account
export async function getAccountTokenInfo(account: PublicKey): Promise<void> {
   const acct = await connection.getAccountInfo(account, 'processed');
   const data = Buffer.from(acct!.data);
   const accountInfo = borsh.deserializeUnchecked(TokenAccount.schema, TokenAccount, data)
   console.log('account info for ', account.toBase58() + ': token address:', new PublicKey(accountInfo.token).toBase58(), ', balance:', accountInfo.amount.toString())
}


You’re now ready to run the client!

Running the Client
To run the client, you simply use the pre-defined ‘start’ npm script in the package.json file:

npm run start


You should see all of the actions defined earlier in the program output. If you run the script a second time, you should see the minted tokens increase, as well as the transfer take place again.





Congratulations, you’ve successfully written, deployed and interacted with a token program!


Completed code repository

Bonus Exercise:
You can attempt to complete these exercises if you’ve completed the main exercise ahead of schedule:


Add a ‘burn’ instruction to the program and client. The burn function should do the exact opposite of the ‘mint’ function, it should remove tokens from the authority account, and decrement the total supply. Remember to include the required checks necessary
Create a new SPL token using the actual SPL Token program with the Solana CLI


Appendix A:

Common errors:

Error during compilation on a Mac, especially after upgrading to Monterey: xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun


Solution: Install the command line tools pacakge



