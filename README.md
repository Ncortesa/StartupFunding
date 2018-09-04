# StartupFunding
**Author**: Nuno Cortesao
**Version**: 0.0.1
----

This project highlights a Smart Contract that can receive information from startups looking for funding. After that investors can vote to invest on a solution. 
It was based on an exercice in order to practice solidity 


As described in the exercise it was necessary to enable a SC that would fully automatize the process of voting on a Startup Contest.
The contest would require 2 steps: A phase where startups could apply to the contest and a phase for investors to vote.

I’ve constructed a Smart contract that is sufficient in solving this topic based on remix and latter compiled via truffle.
The SC is deployed into the **Rinkeby Network**
` 0x8DBB9836d0299307325C5D10C64c427AB2A0a54a `
- - - -

### Breaking down the code ###

The main Contract is called ** StarTroopChallenge** - Star wars Reference to Startup/Startroop dictomy 

As the challenge script was very low on details various assumptions and options have been made
* **Assumption One_ This contract will apply only once:**
So after challenge phases are finished the contract will be locked;_On a future scale up of this code, the smart contract could handle with easyiness the deployment of multiple challenges at the same time, or sequential challenges._

The phases of the challenge are controlled with a variable **status** instaciated by a an enum variable (challengeStatus) that has 4 options: 
0-	Initiated
1-	Registration
2-	Voting
3-	Closed

The Phase one (startup registration) holds information about the idea;
The mandatory information is the Target Ether, the whitepaper URL and the hashcommit; This information is sent by the user and on a future implementation this could be translated by an DAPP._It would be a nice feature to retrieve automatically the hash commit from the url, based on the github API_
I’ve created a structure **idea** to hold this information.  This structure also holds some other properties:
-	Address of the submitter idea: The goal of saving this info is to be able to send to the winner ether on the last phase of the challenge;
-	voteCount: _This variable was initatilizated but was not used and could be removed on a detailed analysis of the code;_ 

Then we have a struct to hold the challenge:
The challenge holds the information who has started the application and some extra information, as the name of the challenge (a preparation for a future expansion of sequencial challenges), a description and now some more tricky information:
* **Assumption 2_ Deadlines:** 
As it was not expressed the type of deadline to be applied in each phase its assumed that phase:
1.	Registration can have x Blocks of deadline or a number x of participants; The decision is applied using a bool variable **typeRegistration**; The deadline for phase one is then expressed using an generic integer that can be related to a nº of participants or the nº of blocks;
2.	Voting phase is defined in number of days, counting from the moment where the challenge is passed to the registration phase. The variable is an integer that will be multiplied after by days to be added in the solidity;
The challenge is mapped to an integer pointer, also providing a bridge for a future escalation;
It as created a pointer to hold all the ideas **idealist**, The Integer pointing to the idea list will be a sequencial ID that it will also relate with other functions and mappings during the code;
In order to hold the votes during the voting phase it was created a structure to hold the nº votes (redundancy with the idea votes) and 2 pointers:
1.	One pointer (**votedAddress**) to easily compare the address Investors voting with the votes of the idea; This will enable and stop double voting per the same users;
2.	A pointer (**SequencialVoter**) to sequencialy detect all the related address’s that voted into the idea;
A variable *nonZeroVotes* array was also created to list all the ideas that have votes during the phase;

A set of variables related to Investors were also creted:
1.	Pointer to address the amount of ether (in wei) invested by the investor - *investmentStack*
2.	A pointer to count the investors and to hold its address - *investorDetails*
A set of helper counters and events was also created(not all events are used). _A future plan to better provide information to Dapps and to better map errors /problems would pass to a detailed analysis of the application of this events_

A set of modifiers were created to apply restriction on the execution of functions
1.	**requesterOnly** - Generic function to allow only one specific address(provided) to interact with the function;
2.	**statusTypeOnly** - Generic function to allow only some status (select the phase) to interact with the specific function
3.	**DeadlineNOTReached** - Function to guarantee the voting phase was not ended
4.	**DeadlineReached** - Function to guarantee the voting phase was finished. This is a redundancy and the modifier 3 could be improved to address both ideas, but the time constrains didn’t allowed for a better judgment. 
5.	**checkZero** - Check zero of a specific integer / Multi purpose;

A Constructor was defined just to start the game and define the submitter of the contract;

## Principal Functions ##

*StartChallenge*
This function is the first function to address the game. Will hold the information about the game and save the information into the contract. Address the business logic of defining the deadlines (blocks/participants) and the deadline of time for the voting phase;
Full details are expressed in the comment area;

### Registration Phase ###

**submittingIdea**
After the challenge is opened this function is enabled (statusTypeOnly). Some details
1.	Inside the function a validation is made to check if the challenge is ready to pass to the voting phase (nº participants is reached or the block number is reached)
2.	The information is registred to the idea being submitted and the ideaNumber is increased each time an idea is applied;
3.	_On a future expansion this function could also verify the commits to guarantee the startups were not being replicated_;

**closingRegistration**
The submittingIdea is the main function during the registration phase. After the conditions to cross to the second phase are reached the challenge has to be changed to a second phase – The voting phase;
This function address this by changing the *status* variable and by adding the nº of days defined during the challenge setup to the “now” moment; 
_An expansion plan should address the non existence of registered ideas. This is checked for some conditions at other functions but a on this function it can be implemented a break to contest as it doesn’t have business logic to continue_

### Voting Phase ###

**addInvestment**
This function allows to investors to add ether to their stacks. As the amount of ether they have will have impact on the voting power this is extremely important function. No proper stress testing or audict have been made due to time constrains. Also *a strong analysis should be conducted to all the payable functions*

Some topics:
1.	The function will check if it’s the first time an investor is allocating money into the contract. 
2.	Also if one request to be an investor and doesn’t send any amount of ether will be excluded;
3.	An investor can add money to its pile while the time deadline was not reached;

**voting**
This function will conduct a very big part of the business logic. It enables one to vote;
1.	**Assumption 3 _Double voting from the same account is not allowed:** As it was not expressed in the challenge, by straight definition of the challenge any investor could vote for multiple times (theoretically  for multiple ideas). It was then defined to accept multiple votes but on different ideas, cutting the possibility of voting on the same idea from the same account twice;

2.	Using the counting variable it was checked if the idea being voted existed or was out of scope. Any attempt of voting for a non existent idea would be discarded;

3.	If the vote was ok it would be added to voting pool. All the address voting for an idea are saved into the variable votesPerIdea. This variable controlled the number of voting, the address voted and a Boolean check to fast analysis of double votes;

4.	It was created a distinction between  the first set of votes on an idea and already voted ideas due to problems on the initialization of the mappings of the structure votePerIdea. I was having troubles to create the instancies with out allocating direct memory on the script;

**closeChallange**
When the  temporal deadline is reached all the functions get locked except for the closeChallange. This function have some local variables that are used for some loop iterations.
On a proper defined contract almost all loops should be done offchain  due to the gas constrains. But in order to simplify and exploit the  SC possibilities the loops were made as no restrictions of ether/money existed;

**Note: This last function was poorly tested. I’m hoping that all the logic is correct as it was created by rampage during a after work night sprint. Concrete testing, stress testing and auditing should be done before any kind of release**

The first "for" checks for the existence of votes on all the ideas. If no ideas have votes then all the money is refunded to the investors;

`        for (uint i=0;i<ideaNumber-1;i++){

            if(votesPerIdea[i].voteNumber>0){
            
                counterNotNulls=counterNotNulls+1;
                
                nonZeroVotes.push(i);
                
            }
            
        }`


The second part of this function will check for the winner idea:
As investors have heights due to its investment, but all the investors can vote only once per each idea I reached the conclusion the summing up all the funds of the investors of an idea would be the same as multiply each vote by a division height. So all the address of each idea are counted and their values summed up.

**Assumption 4_ A second standard was applied to find the winner, in case of vote*weight draw first idea to be submitted would be the winner:** As in the exercice was specified “the winner” I assumed that only one winner could take the funds and as a result a second standard was applied to ensure this; 

A double  for loop was then created to sum all the heighted votes of each idea and to determine the winner;
After the winner was decided the last steps are made.

**Assumption 5_ The winner idea takes all the funds of each investor investing on it** - As the script was not very clear on the amount of ether to pass to the winner idea, and due to time constrains I have went into an easy solution to address this grey area. _A future expansion should make a proper algorithm to pass only the requested money defined in the registration idea step, and to take a proportion of each investor (all the investors or only the investor voting on the idea) to top the requested value.

The last step was to provide all the remaining ether to the others investors that didn’t invest in the idea and to change the status to a closed position;

----

### Final Conclusions ###
The challenge was demanding as it has some gray areas in the scripting that could lead to multiple interpretations.
The social problems addressed on the definition of the algorithm doesn’t have in mind the possibility of attacks using fallback functions, or small logic mistakes. 

The possibility of locking a challenge for lacking of participants could also be a problem.
Proper frameworks to improve security, maths and good practices should have been used (like open zeppelin for example)
A set of automated tests using truffle was also a good feature to check for stress of variables (bad defined integers, sums of big values etc)

A strong review of the necessity of variables should also been made as this is crucial to spare gas consumption. Also the usage of multiple loops are a risk because we can reach a point were is to expensive to close the challenge (millions of ideas/ Votes/ Investors) etc. All this logic should be properly addressed by balancing the backend offchain calculations, front end Dapp restrictions and a more detailed contract;

**End**




