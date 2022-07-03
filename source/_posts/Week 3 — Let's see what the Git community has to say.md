---
title: GSoC Week 3 — Let's see what the Git community has to say!
date: 2022-07-03
---

Well Hello Friend!! 👋

After a lot of iterations and reviews from my mentors, the patch was in a good enough shape to posted on the [git’s mailing list](https://public-inbox.org/git/). 

I am very happy that the patch got reviews from Phillip wood, Đoàn Trần Công Danh and Junio C Hamano! Thanks a ton for reviewing my patches 🙂 🙇

And here is the link to my patch in the mailing list: [https://public-inbox.org/git/xmqqpmipdf48.fsf@gitster.g/T/#t](https://public-inbox.org/git/xmqqpmipdf48.fsf@gitster.g/T/#t)

Now, let’s talk about the reviews!

### Simplify the `commit_rewrite_person()` interface before exposing it

Thanks a lot Junio for pointing this out!! 😀

`commit_rewrite_person()` is a static function in `revision.c` which takes three arguments:

1. `commit_buf` — A string containing the commit buffer
2. `what` — A string containing the header whose idents (name and email) have to be replaced with their canonical versions. It can either be `"\nauthor "` or `"\ncommitter "` 
3. `mailmap` — A string list containing the mailmap itself

At present, the function searches for `what` in whole commit buffer using `strstr()` function. Then it parses that line, get the idents and calls `map_user()` to replace the idents with their canonical versions.

There are a few problems with this:

1. The function, `commit_rewrite_person()`, is designed to find and replace an ident string in the header part, and the way it avoids a random occurrence of `author A U Thor <author@example.com>` in the text is by insisting "author" to appear at the beginning of line by passing `"\nauthor "`as `what`. 
2. The implementation also doesn't make any effort to limit itself to the commit header by locating the blank line that appears after the header part and stopping the search there. Also, the interface forces the caller to make multiple calls if it wants to rewrite idents on multiple headers. It shouldn't be the case.

So, The following changes have to be made before exposing it:

1. Make a single pass in the input buffer to locate headers named "author" and "committer" and replace idents on them.
2. Stop at the end of the header, ensuring that nothing in the body of the commit object is modified.

I have made these changes and have updated in my latest branch of my git fork. Here is the link to the commit simplifying the interface of `commit_rewrite_person()` : [https://gitlab.com/edith007/git/-/commit/dbffbf6799d623a72ba2209603570537709815fd](https://gitlab.com/edith007/git/-/commit/dbffbf6799d623a72ba2209603570537709815fd)

### Come up with a better name for `commit_rewrite_person()`

Thanks a lot Phillip, Christian and Junio for helping to come up with a proper name!

It is getting renamed to `apply_mailmap_to_header()`. This clearly explains what this function is for — will only make changes to the header part of the buffer, does use the mailmap mechanism to replace the idents!

### Simplify some string processing in the tests!

Thanks a lot Phillip for suggesting the change! 😃

In the test when we execute `git cat-file --use-mailmap commit HEAD` we want to get the name and email only from the line, `author Siddharth Asthana <siddharthasthana31@gmail.com> 1656726223 +0530`, and I wrote the following to fetch it:

```bash
git cat-file --use-mailmap commit HEAD >log &&
grep author log >actual &&
sed -e "/^author/q" actual >log &&
sed -e "s/ [0-9][0-9]* [-+][0-9][0-9][0-9][0-9]$//" log >actual
```

Here, the command `sed -e "/^author/q" actual >log` doesn’t have any effect on the contents of the log.

More so, the whole series of commands can be simplified to:

```bash
git cat-file --no-use-mailmap commit HEAD >log &&
sed -n "/^author /s/\([^>]*>\).*/\1/p" log >actual &&
```

and there were a lot of other comments as well about how can I improve my cover letter and some commit messages!

I am cooking the next version of my patches here: [https://gitlab.com/edith007/git/-/commits/mailmap-support-in-cat-file-5/](https://gitlab.com/edith007/git/-/commits/mailmap-support-in-cat-file-5/)

It really feels great to send the changes to the mailing list and getting reviews! It’s a very humbling experience! 🤓

I will soon send the second version of my patches to the mailing list!

Also, I learned how to send patches to the mailing list by following this [https://git-scm.com/docs/MyFirstContribution#ready-to-share](https://git-scm.com/docs/MyFirstContribution#ready-to-share). If you are looking to contribute to git, this will really come handy!

Thanks a lot for reading! Have a wonderful day, dear friend 🙂