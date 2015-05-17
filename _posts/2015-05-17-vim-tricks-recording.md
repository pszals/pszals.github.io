---
layout: post
title: "Vim: When find and replace isn't good enough"
categories: []
tags:
- vim
- tools
- productivity
status: publish
type: post
published: true
author: Philip Szalwinski
---

I was asked recently by a client whether I am more productive using Vim as opposed to a 'regular' text editor, like Sublime, Textmate, or Atom.
I replied that I felt that I'm equally productive in both, but mostly I use Vim because I like the way it makes me feel like a `1337 h4x0r`.
Upon further reflection though, I realized there are some things I perform with relative ease using Vim that other text editors don't elegantly support.
Automating keystrokes is an example where Vim's productivity enhancement shines.

####Enter: recording mode

Recording mode allows you to map a series of keystrokes on the fly to a single key.
The single key, or register, can be invoked as many times as needed.
This means many complicated keystrokes can be executed with trivial ease.
The more keystrokes involved, and the more times the same action has to be repeated, the more powerful this becomes.

As an example, let's look at the specs for the Prime Factors kata.
They are currently using old Rspec syntax, and we want to refactor them away from the monkey-patching use of `.should` and towards the newer syntax that wraps expectations in an `expect()` method.
Rather than changing every line by hand, let's record the keystrokes to a register just once and have our actions play out at the touch of a button.

####Follow along with the screencast:
<iframe width="420" height="315" src="https://www.youtube.com/embed/kDlVxtraZoY" frameborder="0" allowfullscreen></iframe>

####Begin recording and assign the register
Starting from the top, press the 'q' key to begin recording.
Next, assign the following keystrokes to a register.
I'll pick the number 2 key.

####Execute the keystrokes to be recorded
Find all instances of of '.should': `/.should\enter`.
Go to the beginning of the line and begin wrapping the expectation: `Iexpect(\esc`
Close the expectation method and add the equality wrapper: `ni).to eq(\esc`
Delete the `==`: `ndt[`
Close off the last parenthesis: `A)\esc`

####Finish recording
Hit 'q' to complete the recording of the sequence of keystrokes.
The keystrokes are now saved to the register 2.

####Reuse the recording
Because we started the recording with finding all instances of `.should`, by executing the keystrokes we will find the next instance of `.should` and replace it with the new Rspec expectation syntax.
To do so, @ccess the register using the @ key, then press the number 2.

####Rinse and repeat
One of the biggest drawbacks to using Vim is its steep learning curve.
This means that until enough practice is put in, using Vim will actually slow you down.
If you're on a tight deadline and need to get code written, go with what you know and use whatever editor you are comfortable with.
But if you have a bit of time to spare, try out some Vim, you just might find you like it.
