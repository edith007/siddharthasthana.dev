---
title: GSoC Week 5 — A week for experiments
date: 2022-07-18
---

Well Hello Friend!! 👋🏻

Thanks to all the reviews I have been receiving on my patches, I think I have improved my patches a lot. So, far I have submitted 6 patch sets. Let's hope this to be the last one also🤞🏼
Here is the link to my patches and all the discussions on them in the mailing list: [https://lore.kernel.org/git/20220718195102.66321-1-siddharthasthana31@gmail.com/T/#u](https://lore.kernel.org/git/20220718195102.66321-1-siddharthasthana31@gmail.com/T/#u)

Now, let’s talk about the reviews!

## The memory leak issue

Thanks a lot Johannes for pointing this out 🙇

This was another memory corruption. We were traversing the buffer and had a pointer, called `line`, to keep track of the line we are on the buffer. This buffer was passed to `rewrite_ident_line()` which can grow the buffer and return a new buffer. But `line` was still pointing to it and was even later used. The fix was to not use `line` after the call to `rewrite_ident_line()` function. `valgrind` helped us track error.

So, after making the fixes I verified the failing test by running it with `--valgrind-only` flag, and it passed! I am amazed by git’s test suite. It covers so many things, and the tests are easy to write as well.

## Improving the way we iterate over the buffer

Thanks a lot Junio for your suggestion on this one!!

Up until this point I thought of iterating the buffer line by line, so I was fixated on the idea of moving line by line, calculating the length of the line, adjusting the length of the line to account for the change in it’s length due to mailmap. But, then it was pointed out that we can just have an offset and focus on updating it to point to the start of the next line after each iteration. This idea really made the code much easier to understand! I received a lot of help in coming up with the change from my mentor as well! 

So, all the changes are in v6 now and let’s hope they are fine and community likes them, and they get accepted into the master!

Now let's talk about what I have been working on Gitaly this week!

## Exploring contribution graphs 📊

Just to give you an idea, following is the contribution graph on GitLab for the gitaly project.

![Untitled](GSoC%20Week%205%20%E2%80%94%20A%20week%20for%20experiments%2016272c5bb1744d20bcc35006c1ef784f/Untitled.png)

Now as far as I understand, in order to create this graph we need following information:

- Number of commits in the project every four months for the last 2 year and every month of the ongoing year
- Number of commits each contributor has made every four months for the last 2 years and every month of the ongoing year
- The name of contributor
- The email of contributor

as per my current understanding, all of that information is provided by `git-cat-file`. Since, we need information for such a long period (2 years + number of months passed in the current year), so we might have to make a lot of git queries (which gitaly handles), pass all of that to gitlab backend which maybe arranges the information in a way which is easier for the frontend to render. 

Now, the task I did to understand how GitLab gets this contributor’s data from gitaly. So, first task should be to check which RPC is called when GitLab sends a request to gitaly. I tried to load the contributors graph page and check that the only RPC call happening when the contributors graph page is loaded was `FindCommit`. So I tried to debug more and wrote some print statements in `find_commit.go` to get some header object while reloading the contributors graph… I made a file in /tmp and dump the commit objects which we are returning from FindCommit RPC call. 

Some field we find out where:

- commit
- subject
- body
- author
- date
- committer
- parent_ids
- tree_id

(basically the whole commit object)

So, my next set of task will be finding that:

- I don’t think that only `FindCommit` RPC is called when the contributors graph is loaded, so my next task is to find out what other things are happening when the contributors graph is loading.
- We need to find out that what is happening in GitLab part of project when contributors graph is loading.
- Once I have gathered how the contributors graph works, I will try to find if git-shortlog can be used to construct the contributors graph

## Experiment 🧪

Even though my patches are not merged in git, I just wanted to check how gitaly will behave with my changes. So, gitaly has many git binaries. I tried to just replace those binaries with my local binaries which has mailmap in cat-file. After replacing the binaries, I added the `—use-mailmap` to the places in gitaly where `git-cat-file` command is called and run `make` to build those changes. So, I concluded that there was no error in gitaly logs. But, the contributor's graph did not load, which indicates that there are more changes to make other than just adding the `—use-mailmap` flag in gitaly.