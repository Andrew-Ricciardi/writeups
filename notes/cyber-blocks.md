# When reading about security gets you blocked

A short note about something that took me longer to figure out than it should have.

I spend a lot of time reading advisories, patch notes and the odd suspicious page pulled down to look at offline. For a
while, roughly every other session with an AI assistant ended in a refusal, and the annoying part was that it was not
consistent. Same topic, same files, one session fine, the next one blocked. I assumed it was the subject matter. It
mostly was not.

What I think is actually going on, after reading a pile of bug reports about it:

- Some of it is a real bug. There are open issues where the block fires on a filename or a word in a file, with no
  request attached to it at all. That part is not on me.
- The rest is framing. A wide request that mentions a security topic gets read as "help me build the thing". A narrow
  request that names a file and a change gets read as the mechanical edit it is.

Things that changed my hit rate:

1. Name the file, name the change. "Read checker.html and list every host it talks to" gets an answer. "Help me with
   this security project" gets a wall.
2. Keep anything I do not own out of the folder I am chatting about. An agent reads what is on disk, so a folder full
   of somebody else's product code is context I did not mean to give it.
3. Say what the thing is. Evidence I am reading and a codebase I am editing are different tasks, and they get treated
   differently. Being clear about which one I mean is not a trick, it is just accurate.
4. Honest context files. An AGENTS.md or CLAUDE.md that describes the repo truthfully helps, because the agent stops
   guessing. Mine says what the folder is and what it is not. It is not a permission slip and it does not argue with
   anything.
5. Split the questions. "Explain how this class of attack works" and "help me run this against something" are two
   different requests, and mixing them into one sentence gets the second answer applied to the first question.

Things that did not help, and I tried them:

- Magic phrases. Adding "for the defence section" to a request changes nothing if the rest of the request is vague.
- Declaring authorization with nothing behind it. It reads as noise.
- Rewriting the same ask five ways in a row. If the fifth version works, I have learned nothing about why.

The real fix for actual dual use work is a verification route, and Anthropic has one, the Cyber Verification Program.
It is application based, it is scoped to an organisation, and it leans on being able to point at public work. That last
part is most of the reason this repo exists. Writing things up in public is the honest version of proving you do this
work, and it beats arguing with a classifier at 2am.

If you hit a block that is genuinely wrong, say so, there is an issue tracker with hundreds of examples and they do get
looked at. If you hit one that is arguably right, the fix is usually on your side of the keyboard: narrower scope, and
a truthful description of what you are actually doing.
