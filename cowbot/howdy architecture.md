# Some notes on the architecture of Howdy

Howdy, the small discord bot framework made for [[cowbot/Cowbot]] y'know.

## Creating the bot
Howdy implements a handful of helper functions to define the bot's functionality. Here's a small example:

```hs
exampleBot :: Bot
exampleBot = bot $ do
    prefixes ["!"]

    command "ping" $ do
        desc "ping the bot"
        run $ send "pong"
```

In this example a bot is created with the `bot` function, the valid command prefixes are defined and an example command `ping` implemented. All elements follow the same basic structure: a function that creates an 'element' followed by modifiers inside a do block. This is accomplished with the creation of a free monad structure that is then parsed and condensed into multiple record data types.

The issue with this approach is that taking all the data from these functions and rearranging it into something howdy can use to run the bot happens at runtime, when the process is started. There's no reason why this couldn't happen at compile time, though, but it would require implementing it at the type level.

## Lifecycle
Howdy uses `discord-haskell`'s event callback to execute its commands. This requires (as mentioned previously) converting the user defined data to simple data types that can be easily used by commands and reactions, as well as processing event data. The procedure is this:
1. When Howdy received a new message it tries to parse a valid prefix at the beginning of it
2. If that prefix is found, it tries to parse the next word and looks for a command with that name
3. If that command is found it's retrieved and the rest of the necessary event data is collected to be passed to the command
4. Before the command runs a permission check is made
5. The command runs

It's worth noting that commands don't have access to the raw MessageCreate event data, but only the one that's been processed in step 3.

## Error handling
This is the most clunky part of the framework. Errors can happen at multiple levels and they can't always be logged to a discord channel. Right now there's multiple handlers for when they can be forwarded and when they can't, as well as a big Error data type so different errors can be handled differently. It's pretty messy, not nice.

## TO DO?
It would be nice to fix some of the issues here. Primarily error handling and adding some out of the box behaviour (help command). Ideally, data processing should happen at compile time, but I don't know how feasible that would be.
