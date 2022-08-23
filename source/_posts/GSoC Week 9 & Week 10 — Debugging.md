---
title: GSoC Week 9 & Week 10 — Debugging
date: 2022-08-21
---

Well Hello friend 👋🏻

A lot has happened since I published by previous blog. I had to travel a lot last week as my college vacations were over. So, I traveled back to Lucknow from Hyderabad, and then to college. But, the journey doesn’t end here as I am planning to attend the [Rubyconf](https://rubyconf.in/) and [Gophercon](https://gopherconindia.org/) in Pune next week 🤩.

Now, let's talk about project.

## Struggling with Gitaly to make it support mailmap feature

My changes in the Git project to add mailmap support to `git-cat-file` will be release in `v2.38.0`. So, my changes in Gitaly will be made available in production after that release. But, I must test that my changes are indeed working.

So, the first task is to make Gitaly use a git version that has my patches in it. I tried 2 approaches to do so:

1. `The bundled Git approach`
    
    Gitaly uses bundled method, usually for accessing bundled Git binaries. This is not a complete Git installation, but it only contains a subset of binaries that we required at runtime. Bundled Git binaries allows Gitaly to install multiple different versions of Git at the same time. Generally, we use Bundled git method by,
    
    - Adding the Git patches in `_support/git-patches/v2.37.1.gl1` directory to make sure we have those changes in Gitaly new version.
    - Run the command `make WITH_BUNDLED_GIT=YesPlease` or `make` to build Gitaly and test the changes.
    
    But this method is only used when Gitaly team have to solve an important bug/problem we experience that cannot be solved via a different way, and we generally don’t back port patches to Gitaly Git version. 
    
2. `GIT_VERSION method`
    - Comment out `use_bundled_binaries` in `gitaly/gitaly-0.praefect.toml`.
        
        ```bash
        # # Git settings
        [git]
        # use_bundled_binaries = true
        catfile_cache_size = 10
        ignore_gitconfig = true
        ```
        
    - Add `GITALY_TESTING_GIT_BINARY` in `gitlab-development-kit/Procfile`
        
        ```bash
        praefect-gitaly-0: exec /usr/bin/env GITALY_TESTING_GIT_BINARY=/usr/local/bin/git GITALY_LOG_REQUEST_METHOD_DENY_PATTERN="^/grpc.health.v1.Health/Check$" /home/edith/Desktop/gitlab-development-kit/gitaly/_build/bin/gitaly /home/edith/Desktop/gitlab-development-kit/gitaly/gitaly-0.praefect.toml
        ```
        
    - Run the command `make GIT_VERSION=<2.37.2 or Merged Git SHA> git` to compile the Gitaly on local Git version, which already has mailmap feature enabled in git-cat-file.
    - Added `.mailmap` file in `Flightjs/flight` project to check that contributor's graph is using the mailmap feature.
    

Once Gitaly starts using the required git version, the next task is to make changes in gitaly to use `--use-mailmap` flag. For which I have raised this [MR](https://gitlab.com/gitlab-org/gitaly/-/merge_requests/4822#note_1072458262), which adds the `-- use-mailmap` flag to `git cat-file --batch` processes in gitaly.

The problem that I am facing is whenever we try to test these methods locally, we're getting an unusual error of `streaming commit` whenever we try to access the commit history or contributors graph with the `--use-mailmap` flag enabled. I am still working on debugging it!

## Contributors Graph 📊

As mentioned in my previous blog, I shared my finding about which Git commands are executed when a request comes to generate a contribution graph. And we can also possibly get these data directly from `git log --format=...` using the right format. I did some research around `git log --format=...` and as discussed in previous blogs contributors graph only uses `author name`, `author email` and `date` to generate the graph. I think we can use `git log --format="%aN%n%aE%n%as"` instead. 

So yeah, that was the week 9 & week 10. Thanks a lot for reading 🙂

Will be back next week with another blog, Peace! ✌🏻