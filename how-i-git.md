It was late afternoon on Friday and I was more than ready to go home after a
long week when I opened an email about a regression on a client website.
Which, of course, needed to be fixed before the weekend started. So I brought
up the page, determined that yes, there was a problem, and started to look for
when and where it was introduced. I'm somewhat competent with git, so I did a
quick `git bisect` that lead me to the exact commit. Awesome. Well, awesome
until I saw the amazingly worthless commit message, "Oh, and some more
changes". Thanks, Bryan-from-the-Past, you're a jerk.

## The git repository

A git repository is a lot of things: a series of changes to a project, a way
for multiple developers to work on a project, a confusing tangle of concepts
and unintuitive command line arguments. But a git repository is also a story;
a written history of how a project has developed and changed over its
lifetime. And sometimes, if the story is written well enough, the story itself
will tell you where a bug was introduced.

After getting mad at Bryan-from-the-Past far too many times, I started to
refine my git workflow. Essentially, I needed it to support the following
three qualities of a git repository.

1. The resulting git repository should have good forensics. By this, I mean
that I should be able to effectively use the repository to find the root cause
of bugs and to report a log of what changed in a given timeframe.

2. The workflow should not take me out of my programming flow.

3. My workflow should not force my coworkers to accomodate me, and should not
force itself on them.

## Why I need a git workflow

When actively developing, I commit frequently. After any minor change, I
commit. Maybe too often. Maybe I have a problem. Hi, I'm Bryan, and I'm an
over-committer. But I find it's helpful to have so many commits, especially
when I get too far down a rabbit hole and need to revert to a previously good
starting point. However, when I look back at my commit log, I don't want to
see a slew of hardly-understandable commits with quick, off the cuff commit
messages. "Oh, and some more changes," doesn't add anything to the story;
short, meaningless commit messages do not help me get the big picture of what
I've done, and especially don't help anybody else. Which is why, over the
course of my experience with git, I've developed and refined my workflow.

## The process

My process is not very complicated, and is very likely not novel. But it works
for me. The process is, essentially this: (1) use feature branches, and (2)
when I'm convinced that a feature branch works, `git merge --squash` it into
master.

For every feature that I start working on, I create a new branch. I typically
give it a name like `newfeature-wip`; something short but understandable, with
a "Work In Progress" tag at the end. On this branch, I do my thing: frequent
commits, commit messages that are short and don't take me out of my flow, junk
commits, digressions, reverts. I plow forward, making progress in my typical
trial-and-error fashion. And I don't care. The commits are ugly and don't make
sense on their own. Whatever.

Once my meandering coding is done and the feature is complete, I do a `git
checkout master` followed by a  `git merge --squash newfeature-wip`. Its at
this point, out of my programming flow, that the feature is finished and I can
think about the story, think about exactly what the feature does and what it
affects, and write a good commit message. A commit message that Bryan-from-
the-Future will be proud of; one that another developer can read to find out
exactly what happened with the change; one that helps others or future me
understand how the commit affects other parts of the system, and outlines any
special considerations that need to be known.

That's it. Feature branch; squash merge. That simple.

## Forensics

If I follow this workflow, I usually end up with good forensics. First of all,
the git repository ends up being straight-line with no merge commits, [which
does not screw up git bisect](https://sandofsky.com/blog/git-workflow.html).
Secondly, every commit message is useful, which means running a `git log
--author=Bryan --since="2 weeks ago"` gives a full description of everything
I've changed in two weeks, and I don't need to read between the lines to
figure out what I did. And I especially don't have to gloss over commit
messages like, "Oh, and some more changes," a hundred times in a row.

## Programming Flow

Programmers love uninterrupted bursts of programming, where they can keep
their head in the code and make real progress. Stopping to write meaningful
commit messages while in the "programming flow" is counter-productive. This
workflow allows me to stay in the programming flow without sacrificing the
ability to commit often, and I can still take the time to advance the story of
the project when I'm finished and out of the flow.

## Coworkers

What I love most about this workflow is that it usually does not affect my
coworkers at all. They keep working with git in a way that makes sense to
them. What they see from me is the commits on master that I make, the ones
with good commit messages. And if they even see my "Work In Progress"
branches, they can safely ignore them. It also plays nicely by the git laws;
this workflow never rewrites published history and never needs a `git push
--force`.

## Final thoughts

git is a system that is flexible enough to allow many workflows. There is no
shortage of them that you can find on the internet. Over time, as I have used
git, I have found that the "feature branch, squash merge" workflow works
particularly well with how I work. Use it, or use another one. Find something
that works for you.
