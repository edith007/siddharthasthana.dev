---
title: GSoC Week 1 -- The Beginning!
---

Even though GSoC is a remote mentorship program, I didn’t want to be at home during the period, so I came to live with my brother in Hyderabad! 

After settling up, I opened up my laptop and realized that it was the day when it all begins! And it began with a Zoom call with one of my mentor, Christian!

We discussed a lot of things, ranging from “dividing the project into several small goals” to “how should I approach working on the project and importance of communication”.

So, as per our discussions, the project has been divided into 3 broad issues:

1. make `git cat-file` support mailmap  —  [https://gitlab.com/gitlab-org/git/-/issues/106](https://gitlab.com/gitlab-org/git/-/issues/106) 
2. study if/how could the contributor’s graph use `git short log`  —  [https://gitlab.com/gitlab-org/git/-/issues/107](https://gitlab.com/gitlab-org/git/-/issues/107) 
3. add mailmap support to Gitaly using 1.

I started working on the first issue, make `git-cat-file` support mailmap.

Since there are a lot of other commands that already support mailmap, I thought I should spend sometime exploring how they do it. The command that I picked as a reference for my study was `git-show`. I started off by reading its documentation and trying them out, because up until this point I had only used it to see the HEAD commit. After a lot of “print statements” debugging, I figured out that it uses `pretty` library to parse the header and map the user's name and email to their canonical name and email. After discussing this with my mentor, I got to know about another GSoC contributor for the Git project (not GitLab) who is working on unifying the "pretty" and the "ref-filter" formatting codes. In fact, his goal is to improve the ref-filter code so that it can replace the pretty code. So, avoiding code from ref-filter and pretty should be the direction I should head towards.

So, I started exploring the git codebase for references on how can I parse the buffer after reading the object file. I searched for keywords “parse commit” and somehow stumbled upon a static function in `revision.c` called `commit_rewrite_person`. This function takes in the whole buffer, and as the function’s name suggests, rewrites the name and email to their canonical versions in the buffer. It can be used to replace either the committer, author or tagger identification info from the buffer. I thought this function can be moved to `ident.c` where other ident related functions are kept. This way, we can use this in `cat-file.c` as well. Christian even agreed to that approach!

The next task was to use `commit_rewrite_person` in `cat-file.c`. I came up with the following function to achieve the task:

```c
char *replace_persons_using_mailmap(char *commit_buf, unsigned long *size)
{
	struct strbuf sb = STRBUF_INIT;
	strbuf_addstr(&sb, commit_buf);
	commit_rewrite_person(&sb, "\nauthor ", &mailmap);
	commit_rewrite_person(&sb, "\ncommitter ", &mailmap);
	commit_rewrite_person(&sb, "\ntagger ", &mailmap);
	*size = sb.len;
	return strbuf_detach(&sb, NULL);
}
```

The function takes the buffer and replaces the author, committer and tagger ident information from the buffer. Since, the size of the buffer can change when replacing the name and email, the size of the buffer also changes, this function takes care of updating the size as well.

So, after all the research and the changes, `git-cat-file` is indeed able to use mailmap. The following GIF demonstrates it:

[Here](https://gitlab.com/edith007/git/-/commits/mailmap-support-in-cat-file) is the branch where all the code changes I made can be found, along with the reviews from my mentor! 

There are still a lot of rough edges which I have to address, like:

- The function that I have shown above has a memory leak, the strbuf that is created has to be freed!
- I have moved the function `commit_rewrite_person` to `ident.c` but haven’t yet written the much-needed comments about it
- As of now, mailmap is enabled by default in `git cat-file --batch`. It has to be only enabled when `--use-mailmap` flag is given. I am working on figuring out a solution for it.
- Tests and documentation have also to be added

My mentor suggested to me early on that I should keep notes about what I researched, how and my findings in a version controlled text file. Because sometimes it takes a number of weeks before the subject is researched and understood enough so that a detailed and good enough action plan can be found and real coding work following it can start. That's when keeping notes can be very useful. Otherwise, I might not remember what I researched a few weeks ago. So, I am writing all my findings here: [https://gitlab.com/gitlab-gsoc-22/gsoc-2022-reseach-and-priority/-/blob/main/README.md](https://gitlab.com/gitlab-gsoc-22/gsoc-2022-reseach-and-priority/-/blob/main/README.md)

So, yeah! That was all about the week 1! It was pretty exciting, and I am very much looking forward to improving my patches and post it to the git mailing list for a review from the entire git community!