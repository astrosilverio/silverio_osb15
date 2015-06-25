When you play a text adventure there are some things that you'll notice. First off, the game has state. For example, at this point in my Harry Potter game:

```
Filch's Office

You are in a small, spotless room. A filing cabinet stands in the corner and a discontented cat hisses at you from the floor. The door is to the south.
```

there is a room. The room contains a cat. The cat is alive. There is a player. The player is in Filch's Office. These states can change -- this player might not always be in this room. The cat might not always be alive. However, there are rules about how that state can change. Try interacting with the things in the room.

```
> take cat
You can't take that.

> examine cabinet
You pull open a drawer labeled "Confiscated and Highly Dangerous". Something falls out.

> look
You are in a small, spotless room. A filing cabinet stands in the corner and a discontented cat hisses at you from the floor. The door is to the south.

A shimmering cloak lies crumpled in liquid-like folds.
```

If you try and take the cat -- change the cat's state from being in the room to being in the player's possession or inventory -- the game doesn't let you do that. If you try to open the filing cabinet, the game adds an invisibility cloak to the things in the room (the room's inventory).

If you've ever played a Twine game, or another choose-your-own-adventure game, you may already be familiar with this state-machine kind of setup. The difference between a choose-your-own-adventure and a text adventure is that as a player you're not at all limited in what kind of (text) input you can give the game, and the game has a little bit more work cut out for it in interpreting your input.

What needs to happen when a player user types some input? Well, first the game has to figure out what they're talking about -- parse their input into an instruction for the game. Once it thinks it knows what the player wants, it has to try and do that thing. So the game state has a nice conversation with the game rules, and between them they decide whether the change that the player wants to make can happen. Finally, it reports back to the player.

For my text adventure game, I chose to implement three types of stateful things. Rooms, like Filch's Office, have descriptions, can connect to other Rooms, and can contain Things and Players (in other words, have an inventory). Things, like the filing cabinet, have descriptions, may have an inventory, may be edible, etc. Players ("you") have a location, an inventory, and if they've progressed far enough in the game to be Sorted, have a House. The entire state of the game is represented by the collective state of all the Rooms, Things, and Players in the game.

Why? The short answer is, it's easier this way. Imagine if the state was NOT split up. The game state at the point where I try to pick up Mrs Norris is "Player in Filch's Office, Filch's Office contains cat and cabinet, cabinet contains cloak, player holds wand, player's house is Gryffindor..." and much much more (Hogwarts is a big place with a lot of things in it!). When I try to execute "take cat", the game has to decide if I can go from that initial state (call it State A) to "Player is in Filch's Office, Filch's Office contains cabinet, cabinet contains cloak, player holds wand and cat, player's house is Gryffindor..." etc.

In theory, the game would do this by looking up in a table of allowed transitions whether it is allowed to go from State A to State B. That is very simple for the game to do. However, generating a complete list of states and allowed transitions between states, is an absolute nightmare for a gamemaker when the number of possible states, and the number of allowable transitions between states, are as huge as they are in even a moderately sized text adventure.

So then how does the game decide whether or not a transition is allowable, if it can't look for a link between two global states? Well, in a word: logic. Instead of writing out all the situations in which the player could pick up the cat, the gamemaker instead writes out all the conditions that would have to be true for the player to be able to pick up the cat. I find it easiest to encapsulate known potential player requests as functions that I call Commands, which make the requisite state checks and changes. The logic handler's job on receiving processed input from the parser become calling the correct Command with the appropriate arguments.

And here we have an example of generalization improving UX -- to make it easier for the gamemaker to set and regulate game state, we split the game state up into bits and use commands with logic (if statements) to determine whether the request change can happen.