---
title: GSoC Week 2 — Striving to make the patch better!
date: 2022-06-26
---

As I have mentioned in my previous blog, my commits had still a few rough edges, so I spent time fixing those this week!

The first issue that I worked on was fixing the memory leak. Let me show the function where memory leak was:

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

I thought that the strbuf that is created on line 3 is not getting freed and hence causing the memory leak. But, I was completely wrong. My mentor, Christian, helped in identifying the real cause of memory leak.

The real reason for the memory leak was:

- We passed `commit_buf` to the function `replace_persons_using_mailmap()`. Let’s say the address of the `commit_buf` was X.
- Then on line 4, we add the `commit_buf` to a `strbuf`. This call allocates memory to store  a copy of the `commit_buf` buffer inside the `sb.buf` field. Let’s say this newly allocated buffer has the address Y.
- So, we re-write the ident lines on this new buffer (whose address is Y).
- Further, on line 9, when we execute `return strbuf_detach(&sb, NULL);`, we are returning the address of the newly allocated buffer. Basically, we are returning the buffer stored at address Y.
- This returned buffer is later freed in the caller of the function.

So, we basically didn’t operate on the original buffer passed to the function and also in the process lost its address, so it never got freed! Hence, causing the memory leak!

So, in order to fix this memory leak, I thought we need to operate on the original buffer and create a `strbuf` where we don’t allocate a new buffer, instead we use the same buffer in it. On looking in `strbuf.c` I came across the function `strbuf_attach()`. So, I just replaced the call to `strbuf_addstr()` with `strbuf_attach()`. Hence, fixing the memory leak!

In order to verify if this actually fixed the memory leak, I printed the address of the `commit_buf` before passing to the `replace_persons_using_mailmap()` and after returning from the function.

![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/GSoC%20Week%202%20%D1%82%D0%90%D0%A4%20Striving%20to%20make%20the%20patch%20better!%2024180b65a4eb4f388274a3060e9c5b8a/Untitled.png)

As we can see, the address is same, so we are not losing the ownership of the buffer and eventually freeing it!

In order to debug this leak, I also took help of `gdb`. I am writing another detailed blog on how I used `gdb` to debug git.

The next issue that we had was that mailmap was always enabled in `git cat-file --batch`. In order to fix it, I created a static global flag called `use_mailmap`, which was enabled when `--use-mailmap` option is passed.

The next and very important task was to add tests for the changes that I have made in `git-cat-file`. This took some exploration on how the existing tests have been written. I added two tests in `t4203-mailmap.sh` as all the mailmap related tests are written there. In my tests, I have checked when giving `--no-use-mailmap` the mailmap mechanism is disabled and when using `--use-mailmap` it is enabled.

All my changes which I have discussed in this blog can be found [here](https://gitlab.com/edith007/git/-/commits/mailmap-support-in-cat-file2/).

So yeah! that's all for this week! I will keep working on improving my patch and will be back with updates the next week!

Till then, goodbye and thanks for reading :)