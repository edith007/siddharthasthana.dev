---
title: GSoC Week 11 & Week 12 — Things are coming along 🚀
date: 2022-09-04
---

Well Hello friend 👋🏻

Time really flies! It’s been two months since GSoC began, and now it is about to end in a couple of weeks 😔.

A lot has happened since I published my previous blog. I attended my first ever in-person conference in Pune. It was amazing!! I met so many friendly people at this conference, had a lot of good conversations, and made a number of `#rubyconfriends` & `#gopherconfriends` — whom I wish to meet very soon. Now, let's talk about project.

## Bad object size returned by git cat-file

As I have mentioned in my previous blog, I got stuck at a point where we were getting the `streaming commit` error whenever we tried to access the commit history or contributors graph with the `--use-mailmap` flag enabled. 

After the patch for `adding mailmap support in git cat-file` got merged in git master branch, I started working on Gitaly side of project where I have to add `--use-mailmap` flag to `cat-file --batch` processes. 

While testing things locally and trying to debug the error, we found that `git cat-file` is returning bad object size when uses `--use-mailmap`. So, the issue is actually in `Git`. The problem is, when mailmap is applied to a commit object, the size of the commit as returned by `cat-file` doesn’t change. This is a problem for `Gitaly` because it uses this size to read the commit object. So what’s happening is that if mailmap changes the committer name or email, Gitaly will end up reading either less or more of commit object than it should.

In order to verify that we are getting bad object size, I printed out the object information and object size of a commit. The following screenshots will help us understand the problem,

![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/Week11%2612/GSoC%20Week%2011%20%26%20Week%2012%20%E2%80%94%20Things%20are%20coming%20along%20%F0%9F%9A%80%209518c792a62a4b668984c4f0f89005d9/Untitled.png)

![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/Week11%2612/GSoC%20Week%2011%20%26%20Week%2012%20%E2%80%94%20Things%20are%20coming%20along%20%F0%9F%9A%80%209518c792a62a4b668984c4f0f89005d9/Untitled%201.png)

As we can see, the size of object remains same irrespective of `--use-mailmap` flag. The size should have changed when `--use-mailmap` flag was used with `--batch-check` and `-s` options, as the author’s and committer’s name and email have been replaced to their canonical versions using the mailmap mechanism. This is the problem! Thanks to John for the pair debugging session and helping me figure out the root cause of the problem!! 😀

Here is a screenshot of me with my very cool mentor, John Cai!

![John_and_sid.png](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/Week11%2612/GSoC%20Week%2011%20%26%20Week%2012%20%E2%80%94%20Things%20are%20coming%20along%20%F0%9F%9A%80%209518c792a62a4b668984c4f0f89005d9/John_and_sid.png)

Gitaly uses `--batch-all-objects --batch-check --buffer --unordered` to read the size of every object in a repo. As we saw in the previous screenshot, `--batch-check` option in `git cat-file` doesn’t support the mailmap feature because of which we were getting the bad object size from `git cat-file`.  So, my next round of tasks will be to add the mailmap support in `--batch-check` and `-s` option. As `--batch-check`  and `-s` options print object’s size information, and they should also consider the change in object size if `--use-mailmap` option is used with them. 

I am cooking the next version of my patches here:

- add mailmap support in `--batch-check` option → [https://gitlab.com/edith007/git/-/commits/add_mailmap_support_in_batch_check/](https://gitlab.com/edith007/git/-/commits/add_mailmap_support_in_batch_check/)
- add mailmap support in `-s` option → [https://gitlab.com/edith007/git/-/commits/add_mailmap_support_in_s_option/](https://gitlab.com/edith007/git/-/commits/add_mailmap_support_in_s_option/)

I am working with my mentors to improve them before submitting to the git’s mailing list for review from the entire community

And to verify if my changes are working, I printed out the following information on terminal. .

![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/Week11%2612/GSoC%20Week%2011%20%26%20Week%2012%20%E2%80%94%20Things%20are%20coming%20along%20%F0%9F%9A%80%209518c792a62a4b668984c4f0f89005d9/Untitled%202.png)

And it is indeed working! It is very clear from the above screenshot that `--batch-check` and `-s` options are supporting mailmap feature.

So, now after making changes in Git side of project, I tried to test things in Gitaly, to make sure now it is using the right object size. I made Gitaly use the git binary, which have mailmap enabled in `--batch-check` and `-s` options. I then added the `--use-mailmap` option in Gitaly wherever we are using git-cat-file. Here is the [MR](https://gitlab.com/gitlab-org/gitaly/-/merge_requests/4822) which --use-mailmap flag in Gitaly.

Now that all the changes are done, let's verify if the mailmap feature works on contributors graph on GitLab. I added the `.mailmap` file in `Flightjs/Flight` project mapping a user who has made contribution with 2 different emails, and adding the `--use-mailmap` flags in Gitaly.

```bash
angus croll <angus@twitter.com> angus croll <anguscroll@gmail.com>
```

AND It worked 🎉🚀

- Without `--use-mailmap` flag
    
    ![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/Week11%2612/GSoC%20Week%2011%20%26%20Week%2012%20%E2%80%94%20Things%20are%20coming%20along%20%F0%9F%9A%80%209518c792a62a4b668984c4f0f89005d9/Untitled%203.png)
    
- With `--use-mailmap` flag
    
    ![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/Week11%2612/GSoC%20Week%2011%20%26%20Week%2012%20%E2%80%94%20Things%20are%20coming%20along%20%F0%9F%9A%80%209518c792a62a4b668984c4f0f89005d9/Untitled%204.png)
    
    We can see all `angus croll <angus@twitter.com>` has 134 commits with `--use-mailmap` flag enabled, which is the sum of commits made using both of his emails. We can also see that with `--use-mailmap` flag enabled, there is just one entry for angus croll instead of having different entries for both of his emails. Which tells us that GitLab is now supporting mailmap feature on it's contribution graph :)
    
    It really feels great! I have been stuck on this problem for the past few weeks and finally seeing the mailmap feature taking effect on contributors graph is really a happy moment for me! 
    
    ## **The road ahead** 🛣️
    
    So, my next task will be to:
    
    - Send my patch to git mailing list for review.
    - Ensure no feature on GitLab breaks because of the change. If something is breaking, fix it!
    - Work on contribution graph 📊 to make it use `git shortlog` instead of `git cat-file + git log`.
    
    So yeah, that was the week 11 & week 12. Feels really great to share that things are coming along 😀
    
    Thanks a lot for reading 🙂 Will be back next week with another blog, Peace! ✌🏻