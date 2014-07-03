In the fourth novel of Douglas Adams' [Hitchhiker's Guide to the Galaxy
trilogy][hg2g-trilogy] – *So Long and Thanks for All the Fish* – there is a
chapter in which the author humorously lays down the reasons for not
describing *everything* that Arthur Dent does. Chapter 25. Read it. And while
you're at it, read the rest of the novel, and the other four. Because Douglas
Adams. Can. Write.

Douglas Adams' thesis is this: he could explain how Arthur Det brushes his
teeth. He could explain how he rolls around in his sleep. But it doesn't
advance the action. It's a level of minute detail that is completely
irrelevant.

And so it is reading a rambling commit log in git. It's tedious. It's boring.
And it's completely unnecessary. It doesn't advance the action.

Take for example this (made up) series of commits to a git repo.

* Add the button to the HTML
* Style button
* Refactor Request class
* Database changes
* Button should be blue rather than green
* Add confirm method
* fix confirm method
* fix confirm method again
* There we go... Got it working
* call confirm method on button click
* display message
* update message wording from marketing

Sorry, what? I fell asleep. Compare that to this commit message.

* Add "Confirm request" button

Ah. No teeth brushing or sleep rolling. Just advancing the history of the
project.

[Git is a communication tool][communication-tool], not a verbose record of
every tedious change that occurs on a project. Keep it tight.

So how do we squash those commits? How do we turn our history from a rambling
stream-of-consciousness tome into a sharp-witted novella?

I know of two ways. As this is git we're talking about, there are likely 40
ways I don't know about. But I know two.

## The interactive rebase

The interactive rebase was the first method I learned for squashing commits,
and still one I fall back on from time to time.

It's particularly useful when you want to rewrite commits on your current
branch. So, for example, if you're humming along on `master` and suddenly
realize you're producing more wordy cruft than you should be, whip out the
interactive rebase and rewrite your history. (But as always, be sure not to
rewrite published history.)

So how do you use this?

```
$ git rebase -i SOME_PREVIOUS_COMMIT
```

As the section header would suggest, that `-i` stands for "interactive". The
command will drop you into an interactive editor where you can reorder
commits, choose which commits you want to squash together, rewrite the message
for commits, and leave some commits untouched.

When you exit the editor, the history gets rewritten to your will, and you're
done. Boom. A readable history.

## The squash merge

If you frequently use feature branches, the squash merge may be for you. A
squash merge works like a normal merge: you're on the `master` branch, and you
want to merge in changes from the `feature/confirm-request` branch.

```
$ git checkout master
$ git merge --squash feature/confirm-request
```

But the little `--squash` in there tells git that you don't actually want a
merge commit. Instead, you want to take all of the commits in
`feature/confirm-request`, squash them into a single commit, and put that
commit on top of master. Essentially, your feature branch turns into a single
commit.

If this is what you want – a single commit – then perfect. But the drawback
compared to the interactive rebase is that the squash merge *always* results
in a single commit, where the interactive rebase can result in a series of
logical commits. It's a matter of using the right method at the right time.

## Your flow

Git is only a tool. Your workflow is what matters. So I can't tell you what
the best method to use is; try it out and see what works for you. But please
do us all a favor and save us from the "how Arthur Dent brushes his teeth"
commit. Make your history tight.

[hg2g-trilogy]: http://en.wikipedia.org/wiki/The_Hitchhiker%27s_Guide_to_the_Galaxy
[communication-tool]: /git-is-a-communication-tool