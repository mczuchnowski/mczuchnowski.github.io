---
layout: post
title: "Reflog - your last, best hope"
---

Git is very flexible. It allows you to 'save' the current state of your code, do
some experiments and move back in time if your experiments go wrong. It also
allows you to tamper with history - you can rename your old commits, merge them
together and even delete them. But what if you tampered with history and
realized that it was a huge mistake? You can't get back to a point in history
if you removed that point in history. Or can you?

<h2>Reflog - the safety net</h2>

I once tampered with my code's history and lost 5 or 6 last commits. I felt
stupid and hopeless. But git has a special place where it records history of you
changing history. If you run ```git reflog``` command in your console, you will
see a slightly different list of commits for your code. It will tell you when
you switched between branches, merged them, rebased, modified past commits or
wiped out some of them. You can then bring back your code to a state in which it
was before you started playing God. It works the same way as going back in time
through normal commits, i.e. ```git reset```.

There's only one catch - reflog is only stored locally. It is not transferred
through ```git push```, ```git fetch``` and other commands for sharing and
getting code. It does not end up at GitHub. It's good to know that such last
resort solution is available, but you should still be careful when changing
history.
