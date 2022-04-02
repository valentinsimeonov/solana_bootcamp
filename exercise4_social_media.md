**Solana Social**

**In this exercise we will create, deploy and interact with a solana program that stores and retrieves social media posts or blogs on-chain, using the Anchor framework**

Link for the Exercise:
https://docs.google.com/document/d/e/2PACX-1vTm2gQPzKGtoZtTeXJGw6ux69gKDrAtiC8qD6GqWTQwfLaokAv9nnTgnGaniHOOLTZoKosRy0FgvGVy/pub


Initiating the project

The first step is to initiate a new project using the Anchor CLI. Ensuring you are in the folder that you want your exercise folder created, enter the following command into the terminal. It will create the necessary folders and files for your Anchor project. Once created, ensure you can see the folder structure in your VS Code folder explorer. In this example we enter ‘cd ../..’ first to go back two folders, as we were still in the app folder of our previous exercise:

cd ../..
anchor init solana-social
cd solana-social


You should see a ‘solana-social’ folder created that contains all the necessary files and folders for your Anchor project. Ensure you can navigate this folder structure in the VS Code folder navigator (hint: if you need to find it, in VS Code go File → Open and find/choose the folder)






The next step is to install the minimist npm package. We will use it when we create the client later on


npm install minimist

npm install


Now we need to tell the Solana CLI that we want to use a local cluster. Note, If you're on Windows, it is recommended to use WSL to run these commands

solana config set --url localhost


Now we can create a new CLI Keypair. We will use this for interacting with our local cluster.

solana-keygen new -o id.json


Next step is to start our local cluster. Note, you can skip this step if you still have a running local cluster from the previous exercise:

solana-test-validator





If you can’t get the local validator running, then you can just use the public Devnet network. You can configure your setup as follows. Please do not run this command if you are able to successfully start the local validator.


solana config set --url https://api.devnet.solana.com




Now that we have a new keypair, let’s also explicitly tell Anchor to use this keypair when creating transactions. We will do this by setting the ANCHOR_WALLET environment variable. If you are using windows, you may need to set these in your system environment variables. Note we’re appending a ‘..’ to it, because our client will be in a folder under the project root folder:

export ANCHOR_WALLET='../id.json'


Now that we have a new keypair, lets use the Solana airdrop program to retrieve some lamports that we can use to pay for transaction fees:

solana airdrop 2 $(solana-keygen pubkey ./id.json)


You’re now ready to start building the on-chain program!



Creating the Program
In this section, we’ll create a new Rust program that takes in a ‘post’ account, and then depending on if it's a ‘create post’ or ‘update post’ instruction, it will either store a social media post in the account, or it will update the values in a social media post.


The first step is to open the programs/solana-social/src/lib.rs file and enter in the following code. This code
Defines a new anchor program
Defines various constant variables to define the length of each field in a post, as well as the size of other fields to be stored in the post account
Defines a new ‘create_post’ function, which takes an account from the context, and ‘title’ and ‘’content’ parameters, validates the two input strings to ensure their length is valid, then creates the specified account with a new post containing the title, content, current timestamp and the author as the public key of the signing account
Defines a new ‘update_post’ function, which takes an account from the context, and ‘title’ and ‘’content’ parameters, validates the two input strings to ensure their length is valid, then updates the specified account with the new title, content and timestamp
Creates an ‘CreatePost’ struct that defines the accounts passed into the create_post instruction, and the deserialization of the post account into a ‘Post’ struct
Creates an ‘UpdatePost’ struct that defines the accounts passed into the update_post instruction, and the deserialization of the post account into a ‘Post’ struct
Defines the Post struct, that stores the social media post data

use anchor_lang::prelude::*;
use anchor_lang::solana_program::system_program;

declare_id!("BNDCEb5uXCuWDxJW9BGmbfvR1JBMAKckfhYrEKW2Bv1W");

const DISCRIMINATOR_LENGTH: usize = 8;
const PUBLIC_KEY_LENGTH: usize = 32;
const TIMESTAMP_LENGTH: usize = 8;
const STRING_LENGTH_PREFIX: usize = 4; // Stores the size of the string.
const MAX_TITLE_LENGTH: usize = 100 * 4; // 50 chars max.
const MAX_CONTENT_LENGTH: usize = 500 * 4; // 280 chars max.

impl Post {
   const LEN: usize = DISCRIMINATOR_LENGTH
       + PUBLIC_KEY_LENGTH // Author.
       + TIMESTAMP_LENGTH // Timestamp.
       + STRING_LENGTH_PREFIX + MAX_TITLE_LENGTH // Topic.
       + STRING_LENGTH_PREFIX + MAX_CONTENT_LENGTH; // Content.
}

#[program]
pub mod solana_social {
   use super::*;
   pub fn create_post(ctx: Context<CreatePost>, title: String, content: String) -> ProgramResult {
       let post: &mut Account<Post> = &mut ctx.accounts.post;
       let author: &Signer = &ctx.accounts.author;
       let clock: Clock = Clock::get().unwrap();

       if title.chars().count() > 50 {
           return Err(ErrorCode::TitleLength.into())
       }

       if content.chars().count() > 280 {
           return Err(ErrorCode::ContentTooLong.into())
       }

       post.author = *author.key;
       post.timestamp = clock.unix_timestamp;
       post.title = title;
       post.content = content;

       Ok(())
   }

   pub fn update_post(ctx: Context<UpdatePost>, title: String, content: String) -> ProgramResult {
       let post: &mut Account<Post> = &mut ctx.accounts.post;

       if title.chars().count() > 50 {
           return Err(ErrorCode::TitleLength.into())
       }

       if content.chars().count() > 280 {
           return Err(ErrorCode::ContentTooLong.into())
       }

       post.title = title;
       post.content = content;

       Ok(())
   }
}

#[derive(Accounts)]
pub struct CreatePost<'info> {
   #[account(init, payer = author, space = Post::LEN)]
   pub post: Account<'info, Post>,
   #[account(mut)]
   pub author: Signer<'info>,
   #[account(address = system_program::ID)]
   pub system_program: AccountInfo<'info>,
}

#[derive(Accounts)]
pub struct UpdatePost<'info> {
   #[account(mut, has_one = author)]
   pub post: Account<'info, Post>,
   pub author: Signer<'info>,
}

#[account]
pub struct Post {
   pub author: Pubkey,
   pub title: String,
   pub content: String,
   pub timestamp: i64,
}

#[error]
pub enum ErrorCode {
   #[msg("The provided title should be 50 characters long maximum.")]
   TitleLength,
   #[msg("The provided content should be 280 characters long maximum.")]
   ContentTooLong,
}



Make sure your file is saved.! We’re now ready to build and deploy the program to your local Solana cluster



Building and Deploying the Program
Use the following command to build the program with anchor. You should see a similar output:

anchor build





Now that the program has been built, we need to extract the generated program ID and insert it back into the ‘declare_id’ line in the Rust program. We can obtain the program ID using the following command:

solana address -k ./target/deploy/solana_social-keypair.json


Copy the program ID and paste it into the lib.rs program, replacing the string in the ‘declare_id’ line near the top of the program. This will tell Anchor what the program ID is so it can successfully connect and interact with the program once it’s deployed. Save the file once.

declare_id!("4BdMXdR5cgNTQiggcovRwTToydiiwXajdykiJn4VVVxr");


Because you modified the Rust program, you need to now build it again


anchor build



Now we can deploy the program to the local cluster, using the following commands:

anchor deploy


Congratulations, you just deployed your second Anchor Solana program! Now let’s create a client to interact with it.

Creating the Client

Inside the generated ‘app’ folder, create a new file called ‘client.js’





Enter the following code into client.js. This client does the following:
Takes multiple parameters from the command line:
Program - the program ID of the deployed program
Action - create or update a post
Title - the title of the blog post
Content - the content of the blog post
Uses the passed in program ID parameter, and the generated IDL file for the program, creates a connection to the deployed program
Generates a new keypair to be used for creating a new post
Depending on the action parameter, it will call either the createPost or updatePost instruction, passing in the required accounts and string parameters
Prints out the program log messages obtained once the transaction is confirmed
Obtains the post data from the account that was passed into the program, and prints the values to the console

// Parse arguments
// --program - [Required] The account address for your deployed program.
// --action - Create or Update a tweet
// --title - [Required] The title of the post
// --content - [Required] The content of the post
// --post - [Required] The account address of the post you wish to update

const args = require('minimist')(process.argv.slice(2));


// Initialize Anchor and provider
const anchor = require("@project-serum/anchor");
const provider = anchor.Provider.env();
// Configure the cluster.
anchor.setProvider(provider);


//parse params
const PROGRAM_ID = args['program']  // Address of the deployed program.
const ACTION = args['action']  //action to take (create or update)
const TITLE = args['title']  //title of the post
const CONTENT = args['content']  //content of the post
const POST_ACCT = args['post']

console.log('Program ID:', PROGRAM_ID);
console.log('Action: ' + ACTION);
console.log('user public key: ' + provider.wallet.publicKey);
console.log('creating post title: ' + TITLE, 'content: ' + CONTENT);
console.log('post account: ' + POST_ACCT)


async function main() {
   // Read the generated IDL.
   const idl = JSON.parse(
       require("fs").readFileSync("../target/idl/solana_social.json", "utf8")
   );


   // Generate the program client from IDL.
   const programId = new anchor.web3.PublicKey(PROGRAM_ID);
   const program = new anchor.Program(idl, programId);

   let tx

   if (ACTION === 'create') {
       // Create a Post
       //create an account to store the post
       const postAccount = anchor.web3.Keypair.generate();
       console.log('postAccount public key: ' + postAccount.publicKey);

       tx = await program.rpc.createPost(TITLE, CONTENT, {
           accounts: {
               post: postAccount.publicKey,
               author: provider.wallet.publicKey,
               systemProgram: anchor.web3.SystemProgram.programId
           },
           options: { commitment: "confirmed" },
           signers: [postAccount],
       });


       console.log("Fetching transaction logs...");
       let t = await provider.connection.getConfirmedTransaction(tx, "confirmed");
       console.log(t.meta.logMessages);

       // Fetch the account details of the account containing the post
       const postData = await program.account.post.fetch(postAccount.publicKey);
       console.log('Title Is: ' + postData.title)
       console.log('Content Is: ' + postData.content)
   }

   else if (ACTION === 'update') {
       //Update a post
       tx = await program.rpc.updatePost(TITLE, CONTENT, {
           accounts: {
               post: POST_ACCT,
               author: provider.wallet.publicKey
           },
           options: { commitment: "confirmed" },
       });


       console.log("Fetching transaction logs...");
       let t = await provider.connection.getConfirmedTransaction(tx, "confirmed");
       console.log(t.meta.logMessages);

       // Fetch the account details of the account containing the post
       const postData = await program.account.post.fetch(POST_ACCT);
       console.log('Title Is: ' + postData.title)
       console.log('Content Is: ' + postData.content)

   } else {
       console.error('invalid action');
       return
   }

}

console.log("Running client...");
main().then(() => console.log("Success"));


Running the Client

Anchor requires a couple of environment variables set to interact with deployed programs. Set the following environment variables on your terminal, to tell Anchor to use your previously created wallet account. If you’re using windows, you may need to set these in your windows environment variables.

export ANCHOR_WALLET='../id.json'
export ANCHOR_PROVIDER_URL='http://127.0.0.1:8899'


Note: If you’re using the public Devnet network instead of a local validator, set the ANCHOR_PROVIDER_URL to the devnet RPC endpoint:


export ANCHOR_PROVIDER_URL='https://api.devnet.solana.com'


You can run the client with the following command, passing in the program parameter, as well as the create action, then you can choose whatever you want for the post title and contents. This will create a new post for the specified post account that gets created:

cd app

node client.js --program $(solana address -k ../target/deploy/solana_social-keypair.json) --action create --title "this is a title" --content "this is the content for the social media post"




Now that you’ve created a post, you can now run the client again with the `update` action to update the post. Be sure to put different values this time for the post title and contents. For the `post` parameter, you need to enter the postAccount public key that was outputted to the console in the previous step. This is so that we updated the correct account that was populated previously.

node client.js --program $(solana address -k ../target/deploy/solana_social-keypair.json) --action update --title "this is a new title" --content "I have updated the post contents" --post <insert acct from previous tx>







Congratulations, you’ve successfully written, deployed and interacted with the solana social program!


Completed code repository

Bonus Exercise:
You can attempt to complete these exercises if you’ve completed the main exercise ahead of schedule:


Modify the program to update the timestamp on the update post function
Add a ‘delete_post’ function to remove/clear the contents of an account. Hint: you don’t need to manually remove/clear the elements in the instruction logic, you can simply define the post account as follows in the DeletePost struct, and the ‘close = ‘ will handle the removing/clearing of the account, and will send any remaining lamports back to the author

#[account(mut, has_one = author, close = author)]

