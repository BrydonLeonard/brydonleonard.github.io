---
layout: post
title:  "Flashcard Maker"
date:   2026-05-17
categories: automation
tags: automation
---

My wife and I moved to Amsterdam about a year and a half ago now. We wanted to make a concerted effort to integrate and really _experience_ the country we're living in so we started learning Dutch. It was the first language I'd learned as an adult and one of the most useful resources I came across was [Fluent Forever](https://www.goodreads.com/book/show/19661852-fluent-forever). The book is a really fun read that dips its toes into neuroscience and theory around language acquisition while giving very practical tips on how you can use flash cards (I use [Anki](https://apps.ankiweb.net/)) to learn a language.

Several thousand words later the approach is still working really well, but making flash cards by hand has become a chore. When I first started, making new flash cards was exciting. Now, however, it's more difficult to find the motivation to do so. I _could_ download some generic pack of flash cards, but I don't really want to. I want to learn the words that I'm coming across in the wild; they're both easier to learn and more applicable to _me_. Given that we're living in 2026, the correct answer to this problem is obviously LLMs[^1], so I built a little Telegram bot to help!

## Flash cards?

Before we get into it, let's take a very quick look at flashcards, why they're useful, and what makes a good flashcard. 

Remembering things is hard, but we often make it harder for ourselves by choosing sub-optimal ways to learn. If your plan is to just read information and hope it sticks, it's unlikely to go well. **When you're learning, it actually needs to _feel_ hard** for the information to stick[^0]. That feeling of your brain melting isn't _just_ a feeling - it's a real signal that you're working harder and that you're going to retain the information better. It's not fun, but it is necessary in much the same way as exercise needs to be _at least_ a little difficult if you want to see the benefits. 

Retention also improves if you learn things at the right cadence. **Remembering something _right as you're about to forget it_ helps you retain the information much more durably.**

Both making your brain work and revising things at the right time are things that flash cards do really well[^2]:
* **The cards make you recall information _actively_** - Once side of the card has a question, the other the answer. You're not just reading.
* **They help you revise at the right time** - Flash card apps like Anki have built in space repetition, so they'll automatically try to bring back cards right before they think you're going to forget them.

So flashcards are great to begin with, but there are a few things you can do to make them even better:
* **Make them personal** - It's much easier to remember that your friend Jack caught a "papieren vliegtuigje" (paper plane) with his eye than to remember some arbitrary sentence about paper planes.
* **Keep them in context** - Sentences are easier than words in isolation.
* **Use pictures** - If you have the inclination, finding a picture that ties to the card also makes them much easier to remember. 
* **Don't translate** - Use only the target language and pictures on the cards. You have to stop translating in your head at some point, so you may as well do it immediately.

## How it works for us 

Now that you've got some more context on flash cards, let's look at how we make them work for us in our household. Prior to the bot, my wife and had a Whatsapp group to which we'd send new words as we came across them. We'd then sit over the weekends and transform those long lists of words into flash cards. Collecting our own words in a group is something we still really want to do because we can then learn words that we care about. As such, I just needed to replace the manual work at the end. Because Whatsapp bots are a bit of a hassle (you need a Meta business account), I opted to build a Telegram bot instead.

The rough workflow looks like:
1. The bot listens to a Telegram group. It saves every message to a local sqlite DB.
2. Once per day, it triggers the flashcard making flow
  1. It pulls all messages from the DB
  2. It sends a prompt to ChatGPT to ask it nicely to make some flashcards
  3. It writes the flashcards to a CSV and sends them back to the telegram group
3. My wife and I import the flash cards to Anki

![flashcard workflow](/assets/images/2026-05-16-flashcard-maker/flashcard-workflow.png)

## How it went

Given that it's May 2026 and writing code by hand is _so_ February 2026, I vibe-coded the bot. It's not a particularly complex problem so I don't feel too deprived of a challenge, at least. The most difficult part has actually been writing a prompt that results in flash cards that are actually good. Through the months of making flash cards myself, I'd implicitly created several classes of card. I had some for plain vocabulary, others for idioms, some for _interesting_ sentence structures, and even more to work on verb conjugation mistakes I make often. Translating those implicit groupings into a prompt that the AI could follow took several rounds of iteration, but it's now reliably producing cards of _reasonable_ quality. I also made the mistake of trying to use cheaper models to generate the cards. That was a bit of a disaster and the card quality was too inconsistent to be useful, so I've since just moved to GPT 5.5.

All-in-all, I'm happy with the results. The cards aren't quite as good as those I could make myself, but that is the trade-off; they're not as good, but they actually exist.

[^0]: Though there is a limit - if you go past "hard" and into "too hard", your retention goes back down. So, be honest with yourself and make sure you're actually challenging your brain, but keep it reasonable.
[^1]: I joke, but in my mind, this is one of the cases where LLMs _actually_ make sense. They're _made_ to generate text, so generating text in a target language is one of the most appropriate uses for the tool that I can imagine.
[^2]: There are a bunch of other benefits, but these are the two I'd consider the most important.
