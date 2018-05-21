---
layout: single
title: Responsive Discord Help Commands
categories:
  - programming
tags: 
  - javascript
---

You would think that the help command would be one of the easiest commands to implement, *right?*. I mean, it should ideally just print out the command's name followed by a helpful hint on how it works. However, my concern wasn't really about how the information was displayed, but more so about **how the information is navigated.**

Lets break it down for a second. You type `help`. What do you expect? A list, right? But what happens if there is more information than what could be ideally displayed at one time? You would probably have to type something like `help 1` or `help 2` and so on and so forth to access the different pages of the manual. Surely we can make this better.

I am proud to introduce the **_COMPLETELY ORIGINAL_** responsive help command!

![Help command demonstration]({{ "/assets/img/help-command.gif" | absolute_url }})

Let me first quickly explain how it works just in case you're lost. When the `help` command is entered, the bot sends back the first page of the manual, followed by a bunch of buttons that correspond to the other pages. By clicking on each number, you can see that page of the manual. Clicking the `X` results in the bot deleting the message. Neat.

Lets try and go over how to make it. By the way, you can take a look at all of the code in my <a href="https://github.com/KMCGamer/usc_bot" target="_blank">github repository!</a> I will be going over the help command in `commands > help.js`. 

To preface all this, all of my commands have a `module.exports.run` that defines how the commands functions. Also, all the commands contain additional information that is exported such as the name, syntax, description, and more. Specifically, we will be taking a look at `module.exports.run`, which contains the meat of how this works. **Please note:** I am only going to over displaying all commands, so there will be bits of code that are skipped.

## The Code

I first start off by declaring an array that contains my buttons.

```javascript
const buttons = [
  reactions.zero, reactions.one, reactions.two, reactions.three, reactions.four, 
  reactions.five, reactions.six, reactions.seven, reactions.eight, reactions.nine,
];
```

These buttons are imported from another module called <a href="https://github.com/KMCGamer/usc_bot/blob/master/modules/reactions.js" target="_blank">reactions.js</a> which ultimately handles emojis and defines how they're meant to be used. So really, this is just an array of emojis. I then go on to define the how many commands I want per page as well as declare the `pages` variable.

```javascript
const commandsPerPage = 3;
let pages;
```

The `pages` variable will ultimately contain the exact content that the bot will send to the channel for each help page. For now, we are going to just populate it with commands.

```javascript
pages = _.chunk(client.commands, commandsPerPage);
```

`client.commands` is an array that contains the `module.exports` for all commands. Using this, we are going to chunk that array using <a href="https://lodash.com/" target="_blank">lodash</a> based on the amount of commands we specified we wanted per page. In this case, we chose 3. The <a href="https://lodash.com/docs/4.17.4#chunk" target="_blank">chunk method</a> basically takes in an array and spits out an array of arrays of equal size (our size being 3).

Now all we need to do is iterate over each chunk, or group, in `pages` and replace them with the information for all the commands in that chunk in a sendable message format for the bot.

```javascript
pages = pages.map((page) => {
  // Generate the command fields
  const fields = page.map(command => ({
    name: `__${command.name}__`,
    value: `Description: ${command.description}\nSyntax: \`${command.syntax}\``,
  }));
  
  return {
    embed: {
      color: 12388653,
      author: {
        name: 'Click Here For Full List',
        url: 'https://goo.gl/eFN6wF',
        icon_url: 'https://user-images.githubusercontent.com/6385983/34427109-5772d042-ec0c-11e7-896d-7e9096b92856.png',
      },
      fields, // Here are the commands!
    },
  };
});
```
The above code first creates a `fields` array that will contain the command names, descriptions, and syntax in that chunk. We then place those fields directly into another object that represents the entire embed message. We are using embeds because they look super nice compared to regular text messages. I used this <a href="https://leovoel.github.io/embed-visualizer/" target="_blank">embed visualizer</a> to make quick work of setting up the template above. 

Alright, home stretch. We now need to configure how the bot will send the message and listen for the button presses.

```javascript
message.channel.send(pages[0]).then(async (msg) => { // send the first command page
  // Display all the number buttons
  for (const [index, _] of pages.entries()) {
    await msg.react(buttons[index]);
  }

  // Display the X button after the buttons
  await msg.react(reactions.x);
  msg.delete(60000).catch();

  // Create a collector to listen for button presses
  const collector = msg.createReactionCollector((reaction, user) => user !== client.user);

  // Every time a button is pressed, run this function.
  collector.on('collect', async (messageReaction) => {
    // If the x button is pressed, remove the message.
    if (messageReaction.emoji.name === reactions.x) {
      msg.delete(); // Delete the message
      collector.stop(); // Delete the collector.
      return;
    }

    // Get the index of the page by button pressed
    const pageIndex = buttons.indexOf(messageReaction.emoji.name);

    // Return if emoji is irrelevant or the page doesnt exist (number too high)
    if (pageIndex === -1 || !pages[pageIndex]) return;

    // Edit the message to show the new page.
    msg.edit(pages[pageIndex]);

    /*
    Get the user that clicked the reaction and remove the reaction.
    This matters because if you just do remove(), it will remove the bots
    reaction which will have unintended side effects.
    */
    const notbot = messageReaction.users.filter(clientuser => clientuser !== client.user).first();
    await messageReaction.remove(notbot);
  });
}).catch(err => console.log(err));
```

**Lets go through this step by step.**

First off, we send the first page of the manual `message.channel.send(pages[0])`. We then write `.then` to signify that after the prior code completes, we want to do more things. The `msg` variable is the message that was sent by the bot.

We then display all of the necessary page number buttons by looping over `pages.entries()` in a <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of" target="_blank">for...of</a> loop. I would normally do this in a <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in" target="_blank">for...in</a> loop but that caused asynchronous errors as the buttons would appear out of order. I also display the `X` button after the loop and set a timer for the message to delete after 60 seconds.

We then create a <a href="https://discord.js.org/#/docs/main/stable/class/ReactionCollector" target="_blank">ReactionCollector</a> on `msg` and apply a filter (function) onto it that says, "Hey bot, dont listen to your own reactions ya dummy". Had we not done this, the bot probably would have listened to its own button presses.

Now we specify what we want to happen when the collector receives some button presses. First, I address what happens if the `X` button is pressed. If the `X` button is pressed, I want the message to be deleted, and I also want to destroy the <a href="https://discord.js.org/#/docs/main/stable/class/ReactionCollector" target="_blank">ReactionCollector</a>. If the X button wasn't the button that was pressed, we move on.

I start off by getting the index of the page according to the button they pressed. Then, immediately after, I handle any errors that could arise if users react with an emoji to a page that doesn't exist, or react with an emoji that has nothing to do with the help command at all. So, if the `pageIndex` is `-1`, we just do nothing.

Finally, in a *hacky* kind of way, we have to determine which reaction isn't the bots. This is because if you don't specify exactly which reaction you want to remove, it will remove the bots, which has unintended side effects. So we first access the <a href="https://discord.js.org/#/docs/main/stable/class/MessageReaction" target="_blank">messageReaction</a> which contains information about the emoji that was reacted. We access the `users` property to see all the people that reacted with that particular emoji. Ideally, the two people that should be in the `users` property is the bot, and the person interacting with the help command. We then filter out the bot by using <a href="https://discord.js.org/#/docs/main/stable/class/Collection?scrollTo=filter" target="_blank">filter()</a> and then call `first()` to access the user. Then call `await messageReaction.remove(notbot)` to remove the reaction, setting the emoji counter back down to 1.

**_TADA!_** All done!

You can take a look at the rest of the my code to get some inspiration on what kind of arguments you can supply to the help command. Currently, my bot supports a `[command]` argument that allows you to see extra help for a specific command, as well as filtering out certain commands based on privileges!
