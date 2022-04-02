**Chainlink Price Feeds Consumer**

**In this exercise we will create, deploy and interact with a solana program that will consume data from a Chainlink Data Feed. The program will take an account (to store the price data), and a Chainlink Data Feed account, then it will read the latest price from the data feed, and store the value in the first account.**

Link for the Exercise:
https://docs.google.com/document/d/e/2PACX-1vTm2gQPzKGtoZtTeXJGw6ux69gKDrAtiC8qD6GqWTQwfLaokAv9nnTgnGaniHOOLTZoKosRy0FgvGVy/pub


Initiating the project

First step is to initialize a new project using the Anchor CLI. Ensuring you are in the folder that you want your exercise folder created, enter the following command into the terminal. It will create the necessary folders and files for your Anchor project. Once created, ensure you can see the folder structure in your VS Code folder explorer. In this example we enter ‘cd ..’ first to go back a folder, as we were still in the folder of our previous exercise:

cd ../..
anchor init solana-chainlink
cd solana-chainlink


You should see a ‘solana-chainlink’ folder created that contains all the necessary files and folders for your Anchor project. Ensure you can navigate this folder structure in the VS Code folder navigator (hint: if you need to find it, in VS Code go File → Open and find/choose the folder)






Now we need to tell the Solana CLI that we want to use the devnet cluster. Note, If you're on Windows, it is recommended to use WSL to run these commands

solana config set --url https://api.devnet.solana.com


Now we can create a new CLI Keypair. We will use this for interacting with the devnet cluster:

solana-keygen new -o id.json



Now that we have a new keypair, lets use the Solana airdrop program to retrieve some lamports that we can use to pay for transaction fees. In this instance, we’re going to run the command twice to get 4 SOL worth of lamports:


solana airdrop 2 $(solana-keygen pubkey ./id.json) && solana airdrop 2 $(solana-keygen pubkey ./id.json)


You should be able to search for your public key on the Devnet explorer and see the SOL in your account. You can get your account public key with the following command:

solana-keygen pubkey id.json






Then head to the Devnet explorer, and search for your account public key, and look for the SOL in your account. Ensure you have 4 SOL before continuing:




Now that we have a new keypair, let’s also explicitly tell Anchor to use this keypair when creating transactions, and to tell it to use the public Devnet RPC endpoint for sending transactions. We will do this by opening the `Anchor.toml` file in the project root directory, and entering the following into the [provider] section, replacing anything already there:

cluster = "devnet"
wallet = "./id.json"


The final step is to add the Chainlink Solana crate to our project, so we can use Chainlink data feeds. Open the Cargo.toml file in the programs/solana-chainlink folder , and add this line to the dependencies section, underneath the anchor_lang definition, then save your file.

chainlink_solana = "0.1.1"


You’re now ready to start building the on-chain program!



Creating the Program
In this section, we’ll create a new Rust program that takes in a ‘consumer’ account, as well as a chainlink data feed account, and the account of the chainlink data feeds program, and it will obtain the latest price of the specified data feed, and store the result in the specified consumer account.

The first step is to open the /src/lib.rs file and enter in the following code in place of the existing code already there. This code:
Defines a new anchor program
Creates an Decimal struct that defines how the price data is stored in the specified consumer account
Defines a fmt function for formatting price data with the correct decimals and zeros etc
Defines a new ‘execute’ function, which takes the following as inputs:
The consumer account to store the price data
The specified chainlink data feed account (eg SOL/USD) to obtain price data from
The chainlink price feeds program account on Devnet
Defines the execute function body, which performs the following:
Calls the `get_latest_round_data`, `description` and `decimals` functions of the chainlink data feed program for the specified price feed account
Stores the result of the calls above in the specified consumer account via the Decimal struct
Prints out the latest price to the program log output
Defines the execute function context, and what accounts are expected as input when it’s called

use anchor_lang::prelude::*;
use anchor_lang::solana_program::system_program;

use chainlink_solana as chainlink;

declare_id!("7Y7nxA4fDs1uWzFbjNiURHUe4Pcs9XJTyvrBufM6KE6F");

#[account]
pub struct Decimal {
   pub value: i128,
   pub decimals: u32,
}

impl Decimal {
   pub fn new(value: i128, decimals: u32) -> Self {
       Decimal { value, decimals }
   }
}

impl std::fmt::Display for Decimal {
   fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
       let mut scaled_val = self.value.to_string();
       if scaled_val.len() <= self.decimals as usize {
           scaled_val.insert_str(
               0,
               &vec!["0"; self.decimals as usize - scaled_val.len()].join(""),
           );
           scaled_val.insert_str(0, "0.");
       } else {
           scaled_val.insert(scaled_val.len() - self.decimals as usize, '.');
       }
       f.write_str(&scaled_val)
   }
}

#[program]
pub mod solana_chainlink {
   use super::*;
       pub fn execute(ctx: Context<Execute>) -> ProgramResult  {
       let round = chainlink::latest_round_data(
           ctx.accounts.chainlink_program.to_account_info(),
           ctx.accounts.chainlink_feed.to_account_info(),
       )?;

       let description = chainlink::description(
           ctx.accounts.chainlink_program.to_account_info(),
           ctx.accounts.chainlink_feed.to_account_info(),
       )?;

       let decimals = chainlink::decimals(
           ctx.accounts.chainlink_program.to_account_info(),
           ctx.accounts.chainlink_feed.to_account_info(),
       )?;

       // Set the account value
       let decimal: &mut Account<Decimal> = &mut ctx.accounts.decimal;
       decimal.value=round.answer;
       decimal.decimals=u32::from(decimals);

       // Also print the value to the program output
       let decimal_print = Decimal::new(round.answer, u32::from(decimals));
       msg!("{} price is {}", description, decimal_print);
       Ok(())
   }
}

#[derive(Accounts)]
pub struct Execute<'info> {
   #[account(init, payer = user, space = 100)]
   pub decimal: Account<'info, Decimal>,
   #[account(mut)]
   pub user: Signer<'info>,
   pub chainlink_feed: AccountInfo<'info>,
   pub chainlink_program: AccountInfo<'info>,
   #[account(address = system_program::ID)]
   pub system_program: AccountInfo<'info>,
}



Make sure your file is saved! We’re now ready to build and deploy the program to the Devnet cluster.



Building and Deploying the Program
Use the following command to build the program with anchor. You should see a similar output:

anchor build





Now that the program has been built, we need to extract the generated program ID and insert it back into the ‘declare_id’ line in the Rust program. We can obtain the program ID using the following command:

solana address -k ./target/deploy/solana_chainlink-keypair.json


Copy the program ID and paste it into the lib.rs program, replacing the string in the ‘declare_id’ line near the top of the program. This will tell Anchor what the program ID is so it can successfully connect and interact with the program once it’s deployed. Save the file once you have finished modifying it

declare_id!("7Y7nxA4fDs1uWzFbjNiURHUe4Pcs9XJTyvrBufM6KE6F");


Because you modified the Rust program, you need to now build it again

anchor build



Now we can deploy the program to the devnet cluster, using the following commands:

anchor deploy --provider.cluster devnet






Congratulations, you just deployed your Chainlink Solana program! Now let’s create a client to interact with it.

Creating the Client

Inside the generated ‘app’ folder, create a new file called ‘client.js’





Enter the following code into client.js. This client does the following:
Takes two parameters from the command line:
Program - the program ID of our deployed program
Feed - the chainlink data feed account that we want to obtain price data from
Uses the passed in program ID parameter, and the generated IDL file for the program, creates a connection to the deployed program
Generates a new keypair to be used for storing the price data
Calls the ‘execute’ instruction on the deployed program, passing in the account to store the price data in in, the user account paying for the transaction fees, the Chainlink data feed account address of the data feed we are obtaining price data from, the Chainlink Price Feeds program on Devnet, and the system program ID. We also sign the transaction with the account storing the price data
Prints out the program log messages obtained once the transaction is confirmed
Obtains the latest price data stored in the consumer account that was passed into the program, and prints the value to the console

// Parse arguments
// --program - [Required] The account address for your deployed program.
// --feed - The account address for the Chainlink data feed to retrieve
const args = require('minimist')(process.argv.slice(2));

// Initialize Anchor and provider
const anchor = require("@project-serum/anchor");
const provider = anchor.Provider.env();
// Configure the cluster.
anchor.setProvider(provider);

const CHAINLINK_PROGRAM_ID = "CaH12fwNTKJAG8PxEvo9R96Zc2j8qNHZaFj8ZW49yZNT";
const DIVISOR = 100000000;

// Data feed account address
// Default is SOL / USD
const default_feed = "EdWr4ww1Dq82vPe8GFjjcVPo2Qno3Nhn6baCgM3dCy28";
const CHAINLINK_FEED = args['feed'] || default_feed;

async function main() {
 // Read the generated IDL.
 const idl = JSON.parse(
   require("fs").readFileSync("../target/idl/solana_chainlink.json", "utf8")
 );

 // Address of the deployed program.
 const programId = new anchor.web3.PublicKey(args['program']);

 // Generate the program client from IDL.
 const program = new anchor.Program(idl, programId);

 //create an account to store the price data
 const priceFeedAccount = anchor.web3.Keypair.generate();

 console.log('priceFeedAccount public key: ' + priceFeedAccount.publicKey);
 console.log('user public key: ' + provider.wallet.publicKey);

 // Execute the RPC.
 let tx = await program.rpc.execute({
   accounts: {
     decimal: priceFeedAccount.publicKey,
     user: provider.wallet.publicKey,
     chainlinkFeed: CHAINLINK_FEED,
     chainlinkProgram: CHAINLINK_PROGRAM_ID,
     systemProgram: anchor.web3.SystemProgram.programId
   },
   options: { commitment: "confirmed" }, //start with the most recent block that's confirmed
   signers: [priceFeedAccount],
 });

 console.log("Fetching transaction logs...");
 let t = await provider.connection.getConfirmedTransaction(tx, "confirmed");
 console.log(t.meta.logMessages);

 // Fetch the account details of the account containing the price data
 const latestPrice = await program.account.decimal.fetch(priceFeedAccount.publicKey);
 console.log('Price Is: ' + latestPrice.value / DIVISOR)
}

console.log("Running client...");
main().then(() => console.log("Success"));


Install all the other required dependencies used in the client. This includes installing
the minimist npm package, used for parsing command line arguments:


npm install minimist
npm install


You’re now ready to execute the client!

Running the Client

Anchor requires a couple of environment variables set to interact with deployed programs. Set the following environment variables on your terminal, to tell Anchor to use your previously created wallet account, and the public Devnet RPC URL when interacting with deployed programs:

export ANCHOR_PROVIDER_URL='https://api.devnet.solana.com'
export ANCHOR_WALLET='../id.json'



You can now run the client with the following commands, passing in the program and name parameters. You can choose whichever feed you like for the feed parameter

cd app

node client.js --program $(solana address -k ../target/deploy/solana_chainlink-keypair.json) --feed 5zxs8888az8dgB5KauGEFoPuMANtrKtkpFiFRmo3cSa9






Congratulations, you’ve successfully written, deployed and interacted with a Solana program that uses Chainlink data feeds!


Completed code repository

Commit the repository to GitHub

The final step is to commit your repository to GitHub and share your work! Perform the following steps to check in your hardhat project into a public repository for everyone else to see your work

Head over to https://github.com/ and sign-in, or create a new free account if you don’t have one already.
Click on your profile on the top-right corner, and goto Settings -> Developer Settings -> Personal access tokens





Generate a personal access token as per the GitHub instructions

Create a new repository on GitHub. Call it ‘chainlink-solana-bootcamp’, and leave the visibility as ‘public’. To avoid errors, do not initialize the new repository with README, license, or gitignore files. Once you get to the ‘setup page’, leave it open, you will come back to it soon.







Go back to your VS Code terminal for your completed project. Ensure you are in the project's root folder (cd ..) to go back a folder if you are still in the app folder

cd ..


Initialize the local directory as a Git repository.

git init -b main


Add the files in your new local repository. This stages them for the first commit.

git add .


Commit the files that you've staged in your local repository.


git commit -m "Solana Bootcamp".


Go back to GitHub. At the top of your GitHub repository's Quick Setup page, click the copy button  to copy the remote repository URL.




Back in VS code terminal, you need to add the URL for the remote repository where your local repository will be pushed. Enter in the command below, and replace the <PASTE_REMOTE_URL> with the value you copied above. If prompted for GitHub authentication details, use your email that you signed up to GitHub with, and your personal access token that you generated earlier as the password. If you don’t get prompted, you can continue.

git remote add origin  <PASTE_REMOTE_URL>



Verify the new remote URL with the following command. You should see it now pointing to your GitHub repository. This means when we push code, we will be pushing it to your GitHub repository.

git remote -v





Push the changes in your local repository up to GitHub. If prompted for GitHub authentication details, use your email that you signed up to GitHub with, and your personal access token that you generated earlier as the password. This will give your local git program access to push code up to your GitHub account.

git push -u origin main



Your repository should now be live on GitHub! If you still have GitHub open with your new repository, then you can just refresh the page. Otherwise, head to GitHub, then in the top right corner open the menu and choose ‘your repositories’, then click on the URL link to your project to open it up and see. Take note of the URL and share it in the Bootcamp chat to share your completed project with everyone, or share it on Twitter or social media with the tag #chainlink-solana-bootcamp








If you’ve made it this far, congratulations, you’ve completed the 2022 Solana Blockchain Developer Bootcamp!









Bonus Exercise:
You can attempt to complete these exercises if you’ve completed the main exercise ahead of schedule:


Modify the program and client to take two feed parameters, and then use them both to create a new price feed. Eg ETH/USD and SOL/USD, then divide one by the other and return that result (in this case, it would be ETH/SOL)