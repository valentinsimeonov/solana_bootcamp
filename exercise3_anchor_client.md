**GM Anchor Project**

In this exercise we will create, deploy and interact with a solana program that takes your name as an input, and simply outputs a message to the program output. This exercise is exactly the same as the first exercise from day 1, except this time we will use Anchor. This exercise will teach you about the basics of using Anchor in your Solana smart contracts.


Link for the Exercise:
https://docs.google.com/document/d/e/2PACX-1vTm2gQPzKGtoZtTeXJGw6ux69gKDrAtiC8qD6GqWTQwfLaokAv9nnTgnGaniHOOLTZoKosRy0FgvGVy/pub

Installing Anchor and Initiating the project
First step is to check if you have Anchor installed. Run the following command and see if you get back some meaningful output or not:

anchor --version





If you get back the anchor version number, then you have Anchor installed, and you can skip to the next step’. If the command isn’t recognized, you need to install anchor:


First you need to install Yarn if you don’t have it installed


yarn -v


If you don’t get a version back, you can install yarn via the following command:


npm install -g yarn


Now you’re ready to install Anchor


cargo install --git https://github.com/project-serum/anchor --tag v0.20.1 anchor-cli --locked


On Linux systems, you may need to install additional dependencies if cargo install fails:


sudo apt-get update && sudo apt-get upgrade && sudo apt-get install -y pkg-config build-essential libudev-dev


Now check to ensure Anchor is successfully installed



anchor --version







Now that we have Anchor installed, the next step is to initiate a new project using the Anchor CLI. Ensuring you are in the folder that you want your exercise folder created, enter the following command into the terminal. It will create the necessary folders and files for your Anchor project. Once created, ensure you can see the folder structure in your VS Code folder explorer. In this example we enter ‘cd ..’ first to go back a folder, as we were still in the folder of our previous exercise:

cd ..
anchor init gm-anchor
cd gm-anchor


You should see a ‘gm-anchor’ folder created that contains all the necessary files and folders for your Anchor project. Ensure you can navigate this folder structure in the VS Code folder navigator (hint: if you need to find it, in VS Code go File → Open and find/choose the folder)






The next step is to install the minimist npm package. We will use it when we create the client later on


npm install minimist

npm install


Now we need to tell the Solana CLI that we want to use a local cluster. Note, If you're on Windows, it is recommended to use WSL to run these commands

solana config set --url localhost


Next step is to start our local cluster. Note, you may need to do some system config and possibly restart your machine to get the local cluster working. Type the following command into the terminal. Note, you may want to open up a second terminal for this command, so you leave your existing one available for future CLI commands:

solana-test-validator


If you’re using WSL and a build from source, you need to run the local validator not via the Solana CLI:

cd ../solana/validator
./solana-test-validator







If you can’t get the local validator running, then you can just use the public Devnet network. You can configure your setup as follows. Please do not run this command if you are able to successfully start the local validator.


solana config set --url https://api.devnet.solana.com




Now we can create a new CLI Keypair. We will use this for interacting with our local cluster.

solana-keygen new -o id.json



Now that we have a new keypair, lets use the Solana airdrop program to retrieve some lamports that we can use to pay for transaction fees:

solana airdrop 2 $(solana-keygen pubkey ./id.json)  



Now that we have a new keypair, let’s also explicitly tell Anchor to use this keypair when creating transactions. We will do this by setting the ANCHOR_WALLET environment variable. If you are using windows, you may need to set these in your system environment variables. Note we’re appending a ‘..’ to it, because our client will be in a folder under the project root folder:

export ANCHOR_WALLET='../id.json'



You’re now ready to start building the on-chain program!



Creating the Program
In this section, we’ll create a new Rust program that takes in a name parameter, and says ‘GM' to that name by outputting text to the program output, and storing the name in an account which is then read by an off-chain client.


The first step is to open the programs/gm-anchor/src/lib.rs file and enter in the following code. This code
Defines a new anchor program
Defines a new ‘execute’ function, which takes an account from the context, and a ‘name’ parameter, then stores the name string into the specified account, and prints the name out the program output
Creates an ‘Execute’ struct that defines the accounts passed into the execute instruction, and the deserialization of the gm_account account into a ‘GreetingAccount’ struct
Defines the GreetingAccount struct, that stores the name string

use anchor_lang::prelude::*;

declare_id!("75PsQyHnFLhmF1jb4m31f2Gt35HFou3vJsaBCVAhYKAo");

#[program]
pub mod gm_anchor {
   use super::*;
   pub fn execute(ctx: Context<Execute>, name: String) -> ProgramResult {
       let gm_account = &mut ctx.accounts.gm_account;

       gm_account.name = name;
       msg!("GM {}", gm_account.name);
       Ok(())
   }
}

#[derive(Accounts)]
pub struct Execute<'info> {
   #[account(init, payer = user, space = 8 + 32)]
   pub gm_account: Account<'info, GreetingAccount>,
   #[account(mut)]
   pub user: Signer<'info>,
   pub system_program: Program<'info, System>,
}

#[account]
pub struct GreetingAccount {
   pub name: String,
}


Make sure your file is saved.! We’re now ready to build and deploy the program to your local Solana cluster

Building and Deploying the Program
Use the following command to build the program with anchor. You should see a similar output:

anchor build





Now that the program has been built, we need to extract the generated program ID and insert it back into the ‘declare_id’ line in the Rust program. We can obtain the program ID using the following command:

solana address -k ./target/deploy/gm_anchor-keypair.json


Copy the program ID and paste it into the lib.rs program, replacing the string in the ‘declare_id’ line near the top of the program. This will tell Anchor what the program ID is so it can successfully connect and interact with the program once it’s deployed. Save the file once.

declare_id!("75PsQyHnFLhmF1jb4m31f2Gt35HFou3vJsaBCVAhYKAo");


Because you modified the Rust program, you need to now build it again


anchor build



Now we can deploy the program to the local cluster, using the following commands:

anchor deploy


If you’re using the devnet network instead of a local validator, you may need to specifically state devnet:


anchor deploy --provider.cluster devnet


Congratulations, you just deployed your first Anchor Solana program! Now let’s create a client to interact with it.

Creating the Client

Inside the generated ‘app’ folder, create a new file called ‘client.js’





Enter the following code into client.js. This client does the following:
Takes two parameters from the command line:
Program - the program ID of the deployed program
Name - the name we want to say GM to
Uses the passed in program ID parameter, and the generated IDL file for the program, creates a connection to the deployed program
Generates a new keypair to be used for storing the name data
Calls the ‘execute’ instruction on the deployed program, passing in the account to store the name in, the user account paying for the transaction fees, and the system program ID. We also sign the transaction with the account storing the GM name
Prints out the program log messages obtained once the transaction is confirmed
Obtains the name stored in the account that was passed into the program, and prints the value to the console

const anchor = require("@project-serum/anchor");
const provider = anchor.Provider.local();
// Configure the cluster.
anchor.setProvider(provider);
const args = require('minimist')(process.argv.slice(2));

async function main() {
 // Read the generated IDL.
 const idl = JSON.parse(
   require("fs").readFileSync("../target/idl/gm_anchor.json", "utf8")
 );

 // Address of the deployed program.
 const programId = new anchor.web3.PublicKey(args['program']);
 const name = args['name'] || "Glass Chewer";

 // Generate the program client from IDL.
 const program = new anchor.Program(idl, programId);

 //create an account to store the GM name
 const gmAccount = anchor.web3.Keypair.generate();

 console.log('GM account public key: ' + gmAccount.publicKey);
 console.log('user public key: ' + provider.wallet.publicKey);

 // Execute the RPC.
 let tx = await program.rpc.execute(name,{
   accounts: {
     gmAccount: gmAccount.publicKey,
     user: provider.wallet.publicKey,
     systemProgram: anchor.web3.SystemProgram.programId
   },
   options: { commitment: "confirmed" },
   signers: [gmAccount],
 });

 console.log("Fetching transaction logs...");
 let t = await provider.connection.getConfirmedTransaction(tx, "confirmed");
 console.log(t.meta.logMessages);
 // #endregion main

 // Fetch the account details of the account containing the price data
 const storedName = await program.account.greetingAccount.fetch(gmAccount.publicKey);
 console.log('Stored GM Name Is: ' + storedName.name)
}

console.log("Running client...");
main().then(() => console.log("Success"));

Running the Client

Anchor requires a couple of environment variables set to interact with deployed programs. Set the following environment variables on your terminal, to tell Anchor to use your previously created wallet account.Note if you’re using the Devnet network instead of a local validator, set the ANCHOR_PROVIDER_URL to https://api.devnet.solana.com

export ANCHOR_PROVIDER_URL='http://127.0.0.1:8899'


You can run the client with the following commands, passing in the program and name parameters. You can put whatever you wish for the name:

cd app

node client.js --program $(solana address -k ../target/deploy/gm_anchor-keypair.json) --name Harry






Congratulations, you’ve successfully written your first solana program, as well as deployed it to a local Solana cluster and interacted with it with an off-chain client!


Completed code repository

Bonus Exercise:
You can attempt to complete these exercises if you’ve completed the main exercise ahead of schedule:


Modify your program and client so that it also stores a number that counts how many times the account has had GM said to it. To do this, you should initially set the state of the number variable to be 0, then increment it each time, storing it in the account.