Communication is important. There are so many ways it happens: face to face,
phone, email, text, instant message. It's often not thought of as such, but
git is also a tool for communication.

![Old telephones at the Museum of Communication](images/communication/phones.jpg "Copyright Adam Foster. Flickr.")

I've been thinking about the *communication* aspects of git recently. Git *is*
just as much a communication tool as email, IM, or project management
software. It helps you communicate with your coworkers, with maintainers of
open source projects, and even with yourself.

## Coworkers

This happens to me. I'm in a status meeting, or informally talking with a
coworker who is working on the same project, and I get asked, "What did you do
last week?"

```bash
$ git log --since="1 week ago" --author=bryan
```

*That* is what I did. Copy and paste the output.

I don't do it to be a jerk. (I hope my coworkers don't think I do it to be a
jerk.) I do it because it's the most accurate way to communicate what I did.

But for that to be an effective form of communication, every commit has to
make sense. Tell a story. Communicate clearly and concisely exactly what was
done. No more, no less.

## Future me

This also happens to me. I'm working on a project, and I run across old code
that *I* wrote. But I can't remember when it was written or why it's there.

Because of situations like that, I like to think of a hypothetical *future me*
as an audience.

My wife is right: *I can't multitask*. Once I pick up a new task, I'll forget
all about an old one. Which means I won't remember why I made a change a year
from now. Or a month. Or – let's face it – even tomorrow.

So I write my git commit messages today with good communication in mind,
knowing that that I can read them in the future and actually know why a change
was made or get a refresher on what has been done.

It's a little gift to future me.

## Maintainers

Git also facilitates communication with maintainers of software projects that
I will never meet.

As a maintainer of open source projects, I'm more likely to review a pull
request where the author has communicated clearly and conscisely exactly what
was done. I'm less likely to try to decipher the meaning of a mess of
in-progress commits.

So when I submit a pull request, I want to live up to that communication
standard. I recently [landed a commit in a popular library][pfcommit] where
the commit message is longer than the change itself. So what. The commit
message communicates *exactly* what the change is and why it was necessary. No
more, no less.

Was my pull request merged because it was clear? I can't say. But it is what I
would have wanted from a pull request. So I like to think so.

## This sounds like work

This does take work. But it pays off. Creating logical commits and writing a
clear commit messages *now* reduces the mental effort I need to do *later*
when maintaining code. It makes life easier for everybody who is involved in
the project.

It also requires an intermediate knowledge of git. It requires knowing the
difference between unpublished and published history. It requires knowing how
to rebase, to squash, to amend. It requires being able to take a whole slew of
commits that aren't communicating well, packing them up, and making sure they
tell a coherent story.

So take the time. Learn to use git not only as a code repository, but also as
a communication tool. Then, communicate well.

[pfcommit]: https://github.com/scottjehl/picturefill/commit/e8d7f32d7d5f9685451a9ca06a976de1789202db
