---
title: "Retrospecting: idea behind my own AI-powered terminal"
date: 2025-11-19
draft: false
author: "Vyacheslav Mayorskiy, dev team"
summary: "A personal retrospective on the origins and development of Aye Chat, an AI-powered terminal tool."
tags: ["origin-story", "ai", "terminal", "productivity"]
---

I just started thinking back and reminiscing on how it all started (not that long ago: back in September) – and decided to put it in writing. This is a walk down memory lane to the very beginning.

With AI assistants becoming more and more powerful, I as a Python coder found myself using them more and more, but I operate mostly in a terminal SSH sessions, so the practical aspect of such use was "do something in a terminal -> ok, need to solve this problem -> copy/paste into Chat GPT -> receive response -> copy/paste into the terminal. Repeat".

At some point this process of switching back and forth and back and forth became so frustrating that I decided to do something about it.

The very first thing I did was to install a CLI AI assistant. I thought "Aha! I am not the first: someone already solved this pain". Alas, the experience was terrifying: after install - it sent me back to web to log on, and then - it started working, and Oh My God was it full of itself! Every little sneeze - it had to tell me about it, and every little thing it generated - it needed approval from me. So it was acting like a little puppy who's looking for validation: "Did I do it right? Do you like me? Wait, I have some more, do you like it as well?"

I was even more annoyed to say the least - to the degree that I decided to build something from scratch and so that it would address all those pain points that accumulated from web copy/pasting and from CLI assistant asking for approval.

So Aye Chat was born (it had different name at the time of course). The main problem that I wanted to solve was this constant nagging by the famous CLI assistant. It seemed unnatural that in our days when LLMs are reasonably solid and require correction maybe 20-30% of time if that - I would need to approve them for every little thing. I decided that my tool would make updates automatically.

The main problem with that of course is the catastrophic disaster when LLM does screw up and you lose all that you worked so hard on.

Digression: in the 1990-s, PalmPilot became one of the most successful hand-held device companies not because of the device itself (many were doing them) but because they introduced a safety net: syncing your content to a PC. With that – even if you dropped it in a water and it became turtle food (or nest, or mirror – whatever), - your data was safe.

With LLM making updates automatically it was very obvious that there needed to be some kind of rock-solid safety net. Implementation of it became technicality: before every update – just save the file version, and if the update was result of LLM being drunk or whatever – just restore that version.

That was the very first feature that went into this tool: get a response from LLM – if files need to change – save them first – then apply update. Well, "very first feature" after the trivial integration with LLM endpoint of course: nowadays everybody and their guppy do that it seems.

With automatic updates we resolved the first pain point: having to approve every suggestion from LLM.

I was not looking for complex implementations: again, this was just another custom tool for myself – so did not care what others would think because there was no others. There was no fancy extracting of file fragments before sending to LLM, there was no cumbersome aggregation of fragments when receiving them back: simplicity was the name of the game. I was sending full file content and receiving updates for full file content. Moreover, because my projects are fairly small in nature (bunch of AWS Lambda functions, bunch of terraform, bunch of shell scripts) - I would send entire codebase at once and it would still fit into LLM context window. On the upside: there was no missing content and no ambiguity on the LLM side: it had everything it needed to make edits.

With those 2 things: saving files automatically with rudimentary version control and with sending all files – all of a sudden I had a miniature powerhouse, which eliminated the need to go to web for copy/pasting, and that alone reduced the wasted time at least by 30%. Not bad for a 2-day implementation. And as I was sending all files - it did not even occur to me to have a flag to name files individually to put into prompt name by name: of course it's a wildcard mask to do the job, what else.

After that I became greedy. And not “good greedy” when you want something more but when everybody wins: I wanted it all for myself. The next thing I noticed was that I was switching back and forth from the tool session where I was talking to AI to another terminal where I was doing edits and command execution. (Yes, you guessed it: I am not a tmux user. Sue me). “We eliminated going to web already: what’s stopping us from eliminating switching between terminals?”

And so the next major thing went in: shell integration. With the tool (let’s start calling it “Aye Chat”: it was that already at that point), and the lightning fast speed of iterating from version to version, shell integration took less than an hour. And not just “ls -la” type of commands: I was able to open vim right from that very session. Later came “cd” as well – when I started missing it and confinement of a single directory started being painful.

All of the above is in a span of one week mind you: nights and a weekend. I spent another week I can’t even remember on what – but it became an obsession: there was no downtime, there was only my main work, and then – all free time would go into Aye Chat, and it was getting better and better.

By the end of the third week it had so many features that it became rather obvious that this is now not just a side project: it’s a product in making that can help others that are in the same position as me improve productivity tremendously. And not just because of AI-assisted code generation but because of how User Experience was built on top of it. I tried to eliminate most friction points that I would come over – and the experience is now exhilarating: you think of something – you ask it to do it for you – and it does. You run the tests – you see failures – you ask the tool to fix them – and it does.

Of course it’s a fresh product and many things are continuing to be improved – but the pleasure of having a tool work for you instead of you having to fight a tool to be useful – this pleasure is real already.

Who knows what's to come: we now share the development load with our small team (we are a consulting company that my friend and I started). We keep building on it and keep using it ourselves in our projects, and now have a roadmap, a sprint board, and 3-a-week scrums. Aye Chat can now handle larger projects - with built-in RAG (Retrieval-Augmented Generation) capabilites, it has privacy-oriented enterprise-grade features such as offline operation mode, and others. And it keeps growing. And what's more important: however few users we have - they seem to like it.

Or using the words of one of our users: “It looks very promising!” Let’s leave it at that.

---
If you liked what you read - star 🌟 our GitHub repository (https://github.com/acrotron/aye-chat). It helps new users discover Aye Chat. 


