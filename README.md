# solana_bootcamp

**Solana Blockchain Developer Bootcamp Day 1 Exercises*****


**Exercise 1: GM Smart Contract**
In this exercise we will create, deploy and interact with a solana program that takes your name as an input, and simply outputs a message to the program output. This exercise will teach you about the basics of Solana programs and accounts, as well as serialization/deserialization.

Initiating the project
First step is to create a new project using cargo. Enter the following command into the terminal. It will create the necessary folders and files

cargo new gm-program --lib





The next step is to initiate a new project in that folder with npm using the following command in the terminal. You can accept all default values

cd gm-program
npm init -y



Next, we need to add all the required dependencies for our program. Replace the contents of your package.json file with the text below:

{
 "name": "01-gm-program",
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
npm install -g ts-node



Next step is to configure the Cargo.toml file with the correct values. Replace the contents of the file with this:

[package]
name = "gm-program"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
[features]
no-entrypoint = []

[dependencies]
borsh = "0.9.1"
borsh-derive = "0.9.1"
solana-program = "=1.7.9"

[dev-dependencies]
solana-program-test = "=1.7.9"
solana-sdk = "=1.7.9"

[lib]
name = "gm_program"
crate-type = ["cdylib", "lib"]


Now we need to tell the Solana CLI that we want to use a local cluster.

solana config set --url localhost


Now we can create a new CLI Keypair. We will use this for interacting with our local cluster.

solana-keygen new


Next step is to start our local cluster. Note, you may need to do some system config and possibly restart your machine to get the local cluster working. Type the following command into the terminal. If you still have the solana local validator running from the setup instructions, then you don’t need to run this command


solana-test-validator






Now that you have a local cluster running, you can open a second terminal in VS code to continue entering CLI commands. You can do this with the ‘+’ button near the top right of the terminal. You’re now ready to start building the on-chain program





If you can’t get the local validator running, then you can just use the public Devnet network. You can configure your setup as follows. Please do not run this command if you are able to successfully start the local validator.


solana config set --url https://api.devnet.solana.com
solana airdrop 2 


You’re now ready to start creating the program

Creating the Program
In this section, we’ll create a new Rust program that takes in a name parameter, and says ‘GM' to that name by outputting text to the program output, and storing the name in a new account which is then read by an off-chain client.

The first step is to open the /src/lib.rs file and enter in the following code. This code
Tells the compiler we want to use Borsh for serializing and deserializing data
Defines the program as a Solana program that takes the standard parameter inputs
Creates a struct ‘GreetingAccount’ that will define how we read and store data in accounts
Defines the entrypoint of the program as the process_instruction function
Defines the skeleton of the process_instruction function

use borsh::{BorshDeserialize, BorshSerialize};
use solana_program::{
   account_info::{next_account_info, AccountInfo},
   entrypoint,
   entrypoint::ProgramResult,
   msg,
   program_error::ProgramError,
   pubkey::Pubkey,
};

/// Define the type of state stored in accounts
#[derive(BorshSerialize, BorshDeserialize, Debug)]
pub struct GreetingAccount {
   pub name: String,
}

// Declare and export the program's entrypoint
entrypoint!(process_instruction);

// Program entrypoint's implementation
pub fn process_instruction(
   program_id: &Pubkey, // Public key of the account the GM program was loaded into
   accounts: &[AccountInfo], // The account to say GM to
   input: &[u8], // String input data, contains the name to say GM to
) -> ProgramResult {


 
   Ok(())
}



The final step is to flesh out the process_instruction function. Complete the function logic as per the text below. Be sure to save your file once it’s modified. This function does the following
Grabs the account that we want to store the GM name in
Ensures the account can be written to
Deserializes the input data from a byte array into a GreetingAccount struct
Prints a message to the program output
Serializes the GreetingAccount struct (turns it back into a byte array), and stores it in the passed in account



msg!("GM program entrypoint");

   // Iterating accounts is safer than indexing
   let accounts_iter = &mut accounts.iter();

   // Get the account to say GM to
   let account = next_account_info(accounts_iter)?;

   // The account must be owned by the program in order to modify its data
   if account.owner != program_id {
       msg!("Greeted account does not have the correct program id");
       return Err(ProgramError::IncorrectProgramId);
   }

   // Deserialize the input data, and store it in a GreetingAccout struct
   let input_data = GreetingAccount::try_from_slice(&input).unwrap();

   //Say GM in the Program output
   msg!("GM {}", input_data.name);

   //Serialize the name, and store it in the passed in account
   input_data.serialize(&mut &mut account.try_borrow_mut_data()?[..])?;


Nice work! We’re now ready to build and deploy the program to your local Solana cluster



Building and Deploying the Program
Use the following command to build the program. You should see a similar output:

cargo build-bpf --manifest-path=./Cargo.toml --bpf-out-dir=dist/program





Now we’re ready to deploy our program to the localnet cluster:

solana program deploy dist/program/gm_program.so


Congratulations, you just deployed your first Solana program! Now let’s create a client to interact with it.

Creating the Client
First, let's create a new folder to store all our client code. Create a new folder in VS code, and call it ‘client’




Inside the client folder, create the following files with the new file icon:
gm_program.ts
main.ts
utils.ts






Enter the following code into main.ts. This will simply act as an entry point into the client, and then call all the functions required in the gm_programs file. Be sure to save your file once it’s modified.

import {
   establishConnection,
   establishPayer,
   checkProgram,
   sayGm,
   reportGm,
 } from './gm_program';

 async function main() {
   console.log("Let's say GM anon...");

   // Establish connection to the cluster
   await establishConnection();

   // Determine who pays for the fees
   await establishPayer();

   // Check if the program has been deployed
   await checkProgram();

   // Say hello to an account
   await sayGm();

   // Find out how many times that account has been greeted
   await reportGm();

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

 

Lastly, enter the following code into your gm_program.ts. Be sure to save your file once it’s modified. This is the main part of the client, and performs the following functionality:
Establishes a connection to the cluster
Establishes which account will be paying for the generated transactions
Ensures the GM program has been deployed to the cluster
Generates and sends a transaction to the deployed program to say GM to a specific name, passing in an account to be used to store the result
Reads the account data that was sent to the GM Program, and extracts the name that was stored in the account

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
* Hello world's program id
*/
let programId: PublicKey;

/**
* The public key of the account we are saying hello to
*/
let greetedPubkey: PublicKey;

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
const PROGRAM_SO_PATH = path.join(PROGRAM_PATH, 'gm_program.so');

/**
* Path to the keypair of the deployed program.
* This file is created when running `solana program deploy dist/program/gm_program.so`
*/
const PROGRAM_KEYPAIR_PATH = path.join(PROGRAM_PATH, 'gm_program-keypair.json');

const NAME_FOR_GM='Glass Chewer'

/**
* Borsh class and schema definition for greeting accounts
*/

class GmAccount {
   name = "";
   constructor(fields: {name: string} | undefined = undefined) {
     if (fields) {
       this.name = fields.name;
     }
   }
   static schema = new Map([[GmAccount,
       {
           kind: 'struct',
           fields: [
               ['name', 'string']]
       }]]);
}


/**
* The expected size of each greeting account. Used for creating the buffer
*/
const GREETING_SIZE = borsh.serialize(
   GmAccount.schema,
   new GmAccount({ name: NAME_FOR_GM }))
.length;

/**
* Establish a connection to the cluster
*/
export async function establishConnection(): Promise<void> {
   const rpcUrl = await getRpcUrl();
   connection = new Connection(rpcUrl, 'confirmed');
   const version = await connection.getVersion();
   console.log('Connection to cluster established:', rpcUrl, version);
}

/**
* Establish an account to pay for everything
*/
export async function establishPayer(): Promise<void> {
   let fees = 0;
   if (!payer) {
       const { feeCalculator } = await connection.getRecentBlockhash();

       // Calculate the cost to fund the greeter account
       fees += await connection.getMinimumBalanceForRentExemption(GREETING_SIZE);

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
* Check if the hello world BPF program has been deployed
*/
export async function checkProgram(): Promise<void> {
   // Read program id from keypair file
   try {
       const programKeypair = await createKeypairFromFile(PROGRAM_KEYPAIR_PATH);
       programId = programKeypair.publicKey;
   } catch (err) {
       const errMsg = (err as Error).message;
       throw new Error(
           `Failed to read program keypair at '${PROGRAM_KEYPAIR_PATH}' due to error: ${errMsg}. Program may need to be deployed with \`solana program deploy dist/program/gm_program.so\``,
       );
   }

   // Check if the program has been deployed
   const programInfo = await connection.getAccountInfo(programId);
   if (programInfo === null) {
       if (fs.existsSync(PROGRAM_SO_PATH)) {
           throw new Error(
               'Program needs to be deployed with `solana program deploy dist/program/gm_program.so`',
           );
       } else {
           throw new Error('Program needs to be built and deployed');
       }
   } else if (!programInfo.executable) {
       throw new Error(`Program is not executable`);
   }
   console.log(`Using program ${programId.toBase58()}`);

   // Derive the address (public key) of a greeting account from the program so that it's easy to find later.
   greetedPubkey = await PublicKey.createWithSeed(
       payer.publicKey,
       NAME_FOR_GM,
       programId,
   );

   // Check if the greeting account has already been created
   const greetedAccount = await connection.getAccountInfo(greetedPubkey);
   if (greetedAccount === null) {
       console.log(
           'Creating account',
           greetedPubkey.toBase58(),
           'to say hello to',
       );
       const lamports = await connection.getMinimumBalanceForRentExemption(
           GREETING_SIZE,
       );

       const transaction = new Transaction().add(
           SystemProgram.createAccountWithSeed({
               fromPubkey: payer.publicKey,
               basePubkey: payer.publicKey,
               seed: NAME_FOR_GM,
               newAccountPubkey: greetedPubkey,
               lamports,
               space: GREETING_SIZE,
               programId,
           }),
       );
       await sendAndConfirmTransaction(connection, transaction, [payer]);
   }
}

/**
* Say GM
*/
export async function sayGm(): Promise<void> {

   console.log('Saying hello to ',NAME_FOR_GM, ' with key ', greetedPubkey.toBase58());

   //first we serialize the name data

   let gm = new GmAccount({
       name: NAME_FOR_GM
   })


   let data = borsh.serialize(GmAccount.schema, gm);
   const data_to_send = Buffer.from(data);
   console.log(data_to_send)

   const instruction = new TransactionInstruction({
       keys: [{ pubkey: greetedPubkey, isSigner: false, isWritable: true }],
       programId,
       data: data_to_send
   });
   await sendAndConfirmTransaction(
       connection,
       new Transaction().add(instruction),
       [payer],
   );
}

/**
* Report the name of the account that we said GM to
*/
export async function reportGm(): Promise<void> {
   const accountInfo = await connection.getAccountInfo(greetedPubkey);
   if (accountInfo === null) {
       throw 'Error: cannot find the greeted account';
   }
   const greeting = borsh.deserialize(
       GmAccount.schema,
       GmAccount,
       accountInfo.data,
   );
   console.log(
       greetedPubkey.toBase58(),
       'GM was said to ',
       greeting.name
   );
}

Running the Client
To run the client, first you can optionally set the value of the NAME_FOR_GM constant in the gm_program.ts file to be whatever name you wish. It’s currently set to ‘Glass Chewer’. Be sure to save your file once it’s modified.

const NAME_FOR_GM='Glass Chewer'


Finally, you can now run your client. You should see output similar to the screenshot below

npm run start




Congratulations, you’ve successfully written your first solana program, as well as deployed it to a local Solana cluster and interacted with it with an off-chain client!
