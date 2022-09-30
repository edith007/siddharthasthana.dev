---
title: GSoC Week 15 — The Final GSoC Blog!
date: 2022-09-26
---

![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/Week%2015%20and%20The%20Final%20GSoC%20Blog!%20ec5ecf7d0a5140219fe4d6b8051c2a3c/Untitled.png)

Well Hello friend 👋🏻

So, this has been the final week of GSoC 2022. It passed so fast. It has been an amazing journey! I contributed to two great open source projects during this period for my project — Git and GitLab!

Let’s look back, summarizing what was done, the current state of the project and what’s next. I’ve created and submitted the final report for the project. You can check it out [here](https://gitlab.com/groups/gitlab-org/-/epics/8765).

You may also want to check out my weekly posts on the project, which can be found [here](https://siddharthasthana.dev/archives/). You can check by project proposal [here](https://docs.google.com/document/d/16zWn9zSv5r-O_ICZoKX0alMA4tXQs67hoc_CQEd9cJY/edit?usp=sharing).

## The Project

The goal of the project was to make GitLab support Git’s mailmap feature, so that updated contributor information could be shown, instead of possibly outdated information recorded when commits were made. This will allow for a contributor who has changed their email address or name to still be considered as the same person.

Now let’s discuss how we approached to add mailmap support in GitLab!

The first task was to explore how GitLab gets the author and committer identity information. GitLab uses an RPC service called Gitaly for handling all git calls made by GitLab. So, I started exploring Gitaly and found that to fetch the author and committer identity information, Gitaly made use of one of Git’s plumbing command called `git-cat-file`. A lot of commands like `git-log`, `git-shortlog`, `git-show` etc, support the mailmap feature already but `git-cat-file` didn’t. So, the first action item on my list was to make `git-cat-file` support the mailmap feature.

Then, we can enable `--use-mailmap` option wherever we use `git-cat-file` in Gitaly and thus enable GitLab to support Git’s mailmap feature!

### Add Support for mailmap in `git cat-file`

I started off by exploring how other commands which support mailmap feature do it. In my research I came across a function called `commit_rewrite_person()` which replaced the idents in the header part of the commit object with their canonical versions using the mailmap mechanism. So, we decided to use it. When I sent my patch series to the Git’s mailing list, it was suggested to improve this function before making use of it. Some of the problems with this function were:

1. It was designed to find and replace an ident string in the header part of the commit object, but the implementation didn’t make any effort to limit itself to the header part by locating the blank line that appears after the header part and stopping the search there.
2. We wanted to extend this function to also work with tag objects, for which the function enforced the caller to make multiple calls if it wanted to replace idents on multiple headers. That shouldn’t be the case!

So, we first improved this function to support the existing caller by:

1. Making a single pass in the input buffer to located header named “author” and “committer” and replace idents on them.
2. Stopping at the end of the header, ensuring that nothing in the body of the commit object is modified.

We also exposed the function as a public function and renamed it to `apply_mailmap_to_header()`. 

After making these changes, we could make use of the `apply_mailmap_to_header()` function in `cat-file.c` and achieve our goal of making `git-cat-file` support mailmap!

I received a lot of help from my mentors and the reviewers from Git’s mailing list to improve upon my patches and eventually get them merged. I sent 6 patch-sets before my changes were merged. Here are the patches in the v6 patch-set:

- [1/4: revision: improve commit_rewrite_person()](https://lore.kernel.org/git/20220718195102.66321-2-siddharthasthana31@gmail.com/)
- [2/4: ident: move commit_rewrite_person()  to ident.c](https://lore.kernel.org/git/20220718195102.66321-3-siddharthasthana31@gmail.com/)
- [3/4: ident: rename commit_rewrite_person() to apply_mailmap_to_header()](https://lore.kernel.org/git/20220718195102.66321-4-siddharthasthana31@gmail.com/)
- [4/4: cat-file: add mailmap support](https://lore.kernel.org/git/20220718195102.66321-5-siddharthasthana31@gmail.com/)

All of the above were already merged into `master` and are part of Git v2.38.0.

### Update documentation for git-cat-file

Now that `git-cat-file` supports mailmap when used with `--use-mailmap` option, some other options like `--batch-check`, `--batch-command`, `-s`, `-p`, `--batch`  can also be combined with it. So, we should also update the documentation to state that as well. I have sent that patch to Git’s mailing list as well, and it is currently under review. Here is the patch: [https://lore.kernel.org/git/20220926091442.222876-1-siddharthasthana31@gmail.com/](https://lore.kernel.org/git/20220926091442.222876-1-siddharthasthana31@gmail.com/) 

### Make `-s` and `--batch-check` options report correct object size when used with `--use-mailmap`

`git-cat-file` supports two options called `-s` and `--batch-check`, which return the size of the object passed to them. They can be combined with `--use-mailmap` option, but they don’t make use of it and hence the size they return when used with `--use-mailmap` is not correct. They should report the size by considering the effect of mailmap on the idents in the object. Since, the name and email of author and committer in the object can change because of mailmap, so should the size reported by `-s` and `--batch-check` options. 

We discovered this problem when we enabled `--use-mailmap` option wherever we used `git-cat-file` command in Gitaly. Gitaly was reporting `streaming errors` and on debugging the issue with my mentors, we found the issue. So, we decided to fix this issue by making `-s` and `--batch-check` options honor the mailmap mechanism when they are used with `--use-mailmap` option.

Here are the patches to do so:

- [1/2: cat-file: add mailmap support to -s option](https://lore.kernel.org/git/20220926105343.233296-2-siddharthasthana31@gmail.com/)
- [2/2: cat-file: add mailmap support to --batch-check option](https://lore.kernel.org/git/20220926105343.233296-3-siddharthasthana31@gmail.com/)

This patch is not in a state to get merge into master. I am working on v3 patch and I will soon send my patches to the ailing list!

**Note:** You can check all the merged patches in the [official repository](https://git.kernel.org/pub/scm/git/git.git/), searching for “Siddharth Asthana” in the respective branches. For example, here are the searches in [master](https://git.kernel.org/pub/scm/git/git.git/log/?qt=grep&q=Siddharth+Asthana).

### Make Gitaly use the --use-mailmap option implemented in git cat-file

Now that `git-cat-file` supports mailmap, we can make Gitaly use this newly added functionality to enable GitLab to support mailmap.

Here is the [MR](https://gitlab.com/gitlab-org/gitaly/-/merge_requests/4822) which adds the `--use-mailmap` flag in Gitaly wherever `git-cat-file` command is used.

This MR will be merged once all my patches are merged in Git and become a part of a future release and Gitaly upgrades to that version.

## What’s next ********

We already have a good speedup, but the project isn’t finished yet. So, my next task will be to:

- Work on the patch series to enable mailmap mechanism in `-s` and `--batch-check` option of `git cat-file` and look after the bug fixes if any.
- Work on Gitaly side to make sure that the MR for adding the `--use-mailmap` flag is merged after the git patches are merged into Git.

## Thanks 🙇🏻‍♂️

It was really a roller coaster journey for the last 3 months and I didn’t even realize how the time flew by so fast. I want to thank my mentors, Christian and John, for their guidance and support throughout the project. I have really been blessed to have them as my mentors.

And finally, I thank Junio, Phillip, Johannes, Ævar  for the reviews and helping me improve the patches!

Being part of GSoC with GitLab was really amazing and a dream come true! It is something that I will cherish for the rest of my life :)  

So yeah, that was my GSoC journey. Thanks a lot for reading 🙇🏻‍♂️