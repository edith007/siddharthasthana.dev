---
title: GSoC Week 8 — Making progress one step at a time
date: 2022-08-07
---

Well Hello friend 👋🏻

I am really happy to share that the patch for `adding support for mailmap in git-cat-file` is merged into git master branch. 🚀🎉. Thanks to Junio, Phillip, Đoàn, Ævar, Johannes, Christian and John for the review. Now that we are done with Git side of the project, we have to make sure that GitLab use the `--use-mailmap` feature implemented in Git. 

And here is the link for the issue and merged commit,

- Issue : [https://gitlab.com/gitlab-org/git/-/issues/106](https://gitlab.com/gitlab-org/git/-/issues/106)
- Commit : https://github.com/git/git/commit/87098a047be46ee69da056336109eee2139c1398

## Contributors Graph 📊

As mentioned in my previous blog, I shared my findings on how `FindCommits` RPC is called from GitLab and how Gitaly responds with the information from the commit objects which is further processed by GitLab and then contributors graph is generated. Now that we know how GitLab and Gitaly interact, we have to find out which Git commands are executed when a request comes to generate a contribution graph.

So, the main aim was to get what exact git commands do we issue using the FindCommits RPC. On digging, I found that we have something called command factory using which we create commands in gitaly. I added some debug print statements in `command.go` to get the command that we are issuing. On loading the contributors graph we execute the following Git commands:

```bash
/home/edith/gitlab-development-kit/tmp/gitaly-147787/git-exec-3738343929.d/git --git-dir /home/edith/gitlab-development-kit/repositories/@hashed/e7/f6/e7f6c0
11776e8db7cd330b54174fd76f7d0216b612387a5ffcfb81e6f0919683.git -c gc.auto=0 -c core.autocrlf=input -c core.useReplaceRefs=false -c commitGraph.generationVers
ion=1 -c core.fsyncObjectFiles=true cat-file --batch --buffer --end-of-options
```

```bash
/home/edith/gitlab-development-kit/tmp/gitaly-147787/git-exec-3738343929.d/git --git-dir /home/edith/gitlab-development-kit/repositories/@hashed/e7/f6/e7f6c0
11776e8db7cd330b54174fd76f7d0216b612387a5ffcfb81e6f0919683.git -c gc.auto=0 -c core.autocrlf=input -c core.useReplaceRefs=false -c commitGraph.generationVers
ion=1 -c core.fsyncObjectFiles=true cat-file --batch --buffer --end-of-options
```

```bash
/home/edith/gitlab-development-kit/tmp/gitaly-147787/git-exec-3738343929.d/git --git-dir /home/edith/gitlab-development-kit/repositories/@hashed/e7/f6/e7f6c0
11776e8db7cd330b54174fd76f7d0216b612387a5ffcfb81e6f0919683.git -c gc.auto=0 -c core.autocrlf=input -c core.useReplaceRefs=false -c commitGraph.generationVers
ion=1 -c core.fsyncObjectFiles=true log --format=%H --max-count=6000 --no-merges --end-of-options master
```

So, the above `git-log` command indicates that we use `--format=%H --max-count=6000 --no-merges` options, which only gives us the `SHA` of last `6000 no-merge commits`. Next, we pass the `SHA` to `git cat-file --batch` to stream the information from commit object. Later, GitLab receives this information, but only uses `author name`, `author email` and `date` to generate the contributors graph.

It would be possible to get all the info we need directly from `git log --format=...` using the right format… As suggested by my mentor, it would be easy if we break down this into 2 steps:

- Move from using `git log` + `git cat-file` to using just `git log`. As under the current implementation, Gitaly uses `git log --format=...` to get the SHA of command and then use `git cat-file` to stream the commit object.
- Using `git log` to using `git shortlog`.

## Benchmarking git-cat-file 〽️

Earlier, I run the benchmark in order to compare the performance of `git-cat-file` with and without `--use-mailmap` option in the `--batch` mode. This time also, I benchmarked `--batch` and wrote the following script,

- `cat-file.sh`
    
    ```bash
    #!/bin/bash
    git cat-file --batch <<-EOF
    `git log --format=%H --max-count=6000 --no-merges master`
    EOF
    ```
    
- `cat-file-mailmap.sh`
    
    ```bash
    #!/bin/bash
    git cat-file --use-mailmap --batch <<-EOF
    `git log --format=%H --max-count=6000 --no-merges master`
    EOF
    ```
    
    Then, I just passed these shell scripts to `hyperfine` for benchmarking using the following command
    
    ```bash
    hyperfine -N --runs=10000 './cat-file.sh' './cat-file-mailmap.sh'
    ```
    
    I just run, the following `hyperfine` up to 10000 runs just to be sure and got the following results
    
    ![Untitled](GSoC%20Week%208%20%E2%80%94%20Making%20progress%20one%20step%20at%20a%20time%20d9c7e7854ba24570b407ae03540bb174/Untitled.png)
    
    After running this large benchmark test of 10000 runs which probably ran for 2 hours basically shows no significant performance regression which is good and indicates that we can always use `--use-mailmap` flag.
    

## GitLab Hackathon 🦊

Some of you who don’t know, GitLab holds a quarterly hackathon every year. I have been taking part in these hackathons for quite some time. The [Q3 2022 Hackathon](https://about.gitlab.com/community/hackathon/) is a week-long hackathon from August 2 - 9, 2022! 

I asked my mentor if I can take part in the hackathon, and it was ok from him. Since from the start my priority was my **GSoC** project, So, I am working on hackathon issues only when I am not working on the GSoC project. Some of the issues I work on was around,

- RuboCop: Enable previously disabled cop
- Rollout some Feature Flag
- Wrote post deployment migration to drop index_vulnerabilities_on_author_id index
- and so on…

Up until now, I have raised 44 MRs and planning to work on more issues if I get time until the Hackathon is over.

PS: I have over 250 MRs in GitLab with more than 180 merged into main branch!

## The road ahead 🛣️

So, my next task will be to:

- Raise an MR for adding `--use-mailmap` in gitaly [https://gitlab.com/gitlab-org/gitaly/-/issues/4364](https://gitlab.com/gitlab-org/gitaly/-/issues/4364). Up until now, GitLab git repo doesn’t have my patch merged. So, my mentor suggested me the following,
    - Use Git Bundled method to get my patches into the git binary on my local setup so that I can test it locally
- Make progress in exploring `git log --format`.

So yeah, that was the week 8. Thanks a lot for reading 🙂

Will be back next week with another blog, Peace! ✌🏻