---
title: GSoC Week 4 — Improving Patches
date: 2022-07-10
---

Well Hello Friend!! 👋🏻

Time really flies! It's been a month since GSoC began!

I remember discussing with my brother when I got selected for working on this project if I will be able to understand or even work on it. Thanks to my mentors for being there at every step, it's going smoothly! I am very grateful for them and can’t really thank them enough!

I am used to development approach where we create merge requests and then getting reviews on them on platforms like GitLab, GitHub. Git uses its mailing list for everything. People send patches, ask questions, get reviews and post review on patches… It seemed a lot daunting at first, but I think I am getting used to it. That is mostly because of the welcoming community! Thanks a lot Phillip, Danh, Ævar, Johannes and Junio for all the reviews and suggestions and taking time to help me make my patches better! 🙇🏻

So, this week, my aim was to improve my patches based on the reviews I had received on my second patch series.

## Improve commit_rewrite_person()

As I explained in my previous in my previous blog why `commit_rewrite_person()` is not currently in a shape where it can be exposed as a public function. [https://siddharthasthana.dev/blog/week 3 — let's see what the git community has to say/#Simplify-the-commit-rewrite-person-interface-before-exposing-it](https://siddharthasthana.dev/blog/week%203%20%E2%80%94%20let's%20see%20what%20the%20git%20community%20has%20to%20say/#Simplify-the-commit-rewrite-person-interface-before-exposing-it) 

So, in the second patch series it has been fixed. The function now takes three arguments:

1. `buf` — The commit buffer where idents on the headers have to be replaced with their canonical versions
2. `headers` — An array of headers where the idents have to be replaced. For example, `["author", "committer", NULL ]`
3. `mailmap` — And mailmap itself!

So, now we iterate only through the header part of the buffer and replace idents on the passed `headers`only.

The changes can be seen in this commit: [https://lore.kernel.org/git/xmqqmtdky1gq.fsf@gitster.g/T/#mcbf58a4f9137a84a6a6a93b23ce54137cfc48d2b](https://lore.kernel.org/git/xmqqmtdky1gq.fsf@gitster.g/T/#mcbf58a4f9137a84a6a6a93b23ce54137cfc48d2b) 

Thanks a lot for suggesting this change Junio and Danh for helping in improving it even further!

## **Simplify some string processing in the tests!**

As I mentioned in my previous blog, Phillip had suggested some changes which can simplify some string processing in the tests. [https://siddharthasthana.dev/blog/week 3 — let's see what the git community has to say/#Simplify-some-string-processing-in-the-tests](https://siddharthasthana.dev/blog/week%203%20%E2%80%94%20let's%20see%20what%20the%20git%20community%20has%20to%20say/#Simplify-some-string-processing-in-the-tests)

So, in the second patch, I have incorporated them and even added some more tests to test if the changes are working for tag object buffers as well.

The changes can be seen in this commit: [https://lore.kernel.org/git/xmqqmtdky1gq.fsf@gitster.g/T/#m828eec43475bff8df96c3f936e914d77715d497f](https://lore.kernel.org/git/xmqqmtdky1gq.fsf@gitster.g/T/#m828eec43475bff8df96c3f936e914d77715d497f) 

## Make the patches work on Windows and 32-bit Linux

This is something I was not even thinking about! On the v2 patch set series, Johannes pointed out that the patches won’t work on 32-bit Linux nor on Windows because `size_t` and `unsigned long` are not the same thing on those platforms. In my patches, `replace_idents_using_mailmap()` takes two arguments — the buffer (a pointer to `char`) and size (a pointer to `size_t`). But since, the `size` variable we are passing to `replace_idents_using_mailmap()` is of `unsigned long` and hence the problem!

So, wherever we call `replace_idents_using_mailmap()` we have to do as follows:

```c
if (use_mailmap) {
	size_t s = size;
	buf = replace_idents_using_mailmap(buf, &s);
	size = cast_size_t_to_ulong(s);
}
```

Thanks a lot for explaining and suggesting the change, Johannes!

After making all the changes, I have sent the v3 of my patches to the mailing list! 

It can be found here: [https://lore.kernel.org/git/xmqqmtdky1gq.fsf@gitster.g/T/#mbc125288d725bcaa879929d2828198e78f58f780](https://lore.kernel.org/git/xmqqmtdky1gq.fsf@gitster.g/T/#mbc125288d725bcaa879929d2828198e78f58f780) 

---
## My first MR in Gitaly got merged!
Another highlight of this week was that my [first ever merge request on Gitaly](https://gitlab.com/gitlab-org/gitaly/-/merge_requests/4460) got merged!! 🎉

So, in Gitaly for every spawned we sometimes want to see the git version of the spawened command and there was no metric that could expose that. Initially, I had implemented a metric to expose that information, but later it was suggested by my mentor, John, that since we already have metrics for spawned commands, it would make more sense to have a label in them for the git version. So, my merge request added a git version label to the command related metrics.

Thanks a lot for all the help on this, Christian, John and Patrick!! 

## The road ahead

- The tests that I have added in my patches are failing on windows, so I am actively debugging that and will send another patch series to fix that!
- I have started studying what information does GitLab needs to generate the contribution graphs and if `git-shortlog` can be used instead of `git-cat-file`. So, I am going to write my findings in the future blogs! So, stay tuned!

So yeah, that was the week 5. Thanks a lot for reading 🙂

Will be back next week with another blog, Peace! ✌🏻