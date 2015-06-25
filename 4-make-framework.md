To make a framework, we have to abstract game data out of the parser, the logic handler, and the state. Creating a game instance then becomes initializing the engine with game data.

Let's start with the parser. The parser's job is to pass an instruction to the logic handler from the player's input, so the only game knowledge it needs is what words to recognize. My text adventure game has a whitelist of accepted words that represent commands, players, things, rooms, and directions -- in other "words", only words that represent concrete things or concepts in the game are recognized. Imagine my parser was given this sentence to parse for a Pride and Prejudice themed game:

"It has been coming on so gradually, that I hardly know when it began. But I believe I must date it from my first seeing his beautiful grounds at Pemberley."
    
Of all the words in this sentence, only two words represent concepts or things in the game: "date", which is a game-specific command that takes a character name as an argument; and "Pemberley", which is a location in the game. The parser passes ["date", "Pemberley"] off to the logic handler--which doesn't make sense to the logic handler, but hey, the parser tried. This approach means that abstracting game data from the parser becomes a matter of just initializing the parser with a custom list of words.

Abstraction in the logic handler is a little more difficult. We can keep the same general principle--logic handler receives instructions from parser, dispatches instructions to appropriate Command--but Commands have to be totally customizable. I want my HP-themed game to have accio, for example, but I don’t want accio to exist in a great American road-trip adventure. Some commands should come built-in with the engine. Chances are very high that you’ll need go, for example, and you shouldn’t have to re-implement it for every game. But you should be able to modify the built-in commands. Maybe in your road-trip adventure, go west will move you one unit west if you’re not in a car, and will move you and your car 10 units west if you are in a car.

So how do we do this?

Well, first we have to decide what we mean by a command. What does a command need to do? I expect the command to do something based on my input, if my input is valid, and then return a response. For example, if I type some garbage like go wand, I expect the go command to do nothing and return something like “I don’t know where you want me to go.” If I type go south and there is a room to the south, the go command should move me south and then return the description of my new location. If there is *not* a room to the south, the go command should not move me.

I distilled those requirements down to four responsibilities (syntax checks, logic checks, (optional) state change, and response formulation), and wrote a Command class that is instantiated with helper functions for each of these responsibilities and that, when executed, calls those helper functions in this order.

