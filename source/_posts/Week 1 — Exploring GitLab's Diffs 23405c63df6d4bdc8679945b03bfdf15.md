---
title: Week 1 — Exploring GitLab's Diffs
date: 2023-06-4
tags: ["GSoC2023", "GitLab", "RangeDiff", "Diff"]
---

I am finally back home 🏡 and it's May 29th, marking the official start of GSoC coding period! Since I was about to graduate from college, I reached out to Christian and ZheNing to confirm my commitment to start working on the project from the day I will be back 🏡. In my pervious [blog](https://siddharthasthana.dev/blog/kickstarting%20gsoc%202023%20with%20gitlab%20%F0%9F%9A%80/), I shortly discussed my GSoC project. Now, let's delve into the details and explain it more comprehensively.

## What is `git range-diff`?

`git range-diff` allows us to compare two versions of a patch series or commit ranges (eg. two versions of a branch).

When we use the git range-diff, it looks for different sets and identifies pairs of commits that share some similarities like

- Author’s name
- compares the commit messages to see if they are similar
- actual code changes (diffs) between the commits
    
    After identifying the corresponding pairs of commits, git range-diff gives us a detailed summary of the differences between them. This summary includes information that I mentioned above in bullet points ☝🏻. It helps us to understand the modifications made in each commit and how they are connected to each other.
    

Let’s imagine we are working on a project and have several commits on a branch called `our`. And, the main branch, called `master`, has also progressed with it’s own commits.

To keep our branch up to date with the latest changes in the master branch, we decided to `rebase` it with `master`. During the `rebase`, we encounter some conflicts on one of our commits, let’s call it `o1`. We resolved these conflicts by making the necessary changes to the code.

Once the `rebase` is complete, we end up with a new version of our branch `our` with modified commits `o1'`, `o2'`, and `o3'`. While we still have the original version of our branch, `backup_our` with commits `o1`, `o2` and `o3`. It's crucial to ensure a seamless rebase process, minimizing unintended modifications caused by merge conflicts and code changes. Merge conflicts happen when conflicting changes are made to the same code in different branches. Resolving these conflicts requires careful attention to detail to avoid unintended changes and mistakes. By giving importance to a smooth rebase process, we ensure that the codebase remains reliable and reduce the chances of unwanted changes that could affect how the system works and how stable it is.

This is where `git range-diff` comes in handy. By comparing the commits from the original branch version `backup_our` to the new version `our`, we can easily identify unexpected changes.

We can simply compare the sets of commits by:

```ruby
git range-diff backup_our~3..backup_our our~3..our
```

It will help us verify that the rebase process didn’t introduce any unintended changes.

A typical output of `git range-diff` would look like this:

![Screenshot 2023-06-05 at 1.19.44 AM.png](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/Week%201%20%E2%80%94%20Exploring%20GitLab's%20Diffs%2023405c63df6d4bdc8679945b03bfdf15/Screenshot_2023-06-05_at_1.19.44_AM.png)

## GitLab's `range-diff` advancement

In GitLab, the current functionality only allows us to see a basic difference between the old and new versions of a branch using `git diff`. However, incorporating `git range-diff` can greatly enhance the effectiveness of code reviews. It will enable us to precisely identify the changes across different versions of a merge request.

By using `git range-diff`, developers and reviewers can ensure that no unintended modifications were introduced during the process of rebasing the merge request with the master branch.

The objective of this project is to introduce an RPC (Remote Procedure Call) in `Gitaly`, a Git server component, that will facilitate the execution of git range-diff between branches and retrieve the comparison results. Once this RPC call is successfully implemented, the subsequent goal is to integrate this functionality into GitLab, allowing the display of detailed range-diff information directly on merge request pages.

## Current Progress 🏗️

Since GSoC has just started, I've been exploring GitLab to understand how it uses `git diff` to compare different versions on the merge request (MR) page. Here are some key findings I've discovered while working with diffs in GitLab:

- **What Git command is been executed during the Diff process?**
    - After going through Gitaly codebase and implementing some debugging statements, we find out the specific git command being executed during the diff process
        
        ```c
        $ git diff --raw --abbrev=40 --full-index --find-renames <left_commit_id> <right_commit_id>
        ```
        

- **What `CommitDiffRequest` RPC is returning during diff process?**
    - `eachDiff` function in `commit.go` process the `diff` and send response through a stream. Inside the callback function  provided to `eachDiff`, a `CommitDiffResponse` struct is created based on the information extracted from the parsed diff.
        
        
        Here is a code snippet of `commit.go` file 
        
        ```go
        return s.eachDiff(stream.Context(), in.Repository, cmd, limits, func(diff *diff.Diff) error {
        		response := &gitalypb.CommitDiffResponse{
        			FromPath:       diff.FromPath,
        			ToPath:         diff.ToPath,
        			FromId:         diff.FromID,
        			ToId:           diff.ToID,
        			OldMode:        diff.OldMode,
        			NewMode:        diff.NewMode,
        			Binary:         diff.Binary,
        			OverflowMarker: diff.OverflowMarker,
        			Collapsed:      diff.Collapsed,
        			TooLarge:       diff.TooLarge,
        		}
        ```
        
        The **`response`** variable contains various fields such as **`FromPath`**, **`ToPath`**, **`FromId`**, **`ToId`**, **`OldMode`**, **`NewMode`**, **`RawPatchData`**, and **`EndOfPatch`**.
        
        I wrote some debug statements to get the response data to a file. The debug output gives us insights into the changes between the two commit versions.
        
        ```go
        from_path:"CONTRIBUTING.md"  to_path:"CONTRIBUTING.md"  from_id:"58ac8b45efb332b82f0f014c7021548b20f0d75d"  to_id:"245c7d5c017935015684ad2bd40dafcd851210e1"  old_mode:33188  new_mode:33188  raw_patch_data:"@@ -116,4 +116,6 @@ https://github.com/twitter/flight/blob/master/LICENSE
         
         ## GitLab
         
        -World's First Remote Company. GitLab Inc. is an open-core company that operates GitLab, a DevOps software package which can develop, secure, and operate software.
        \ No newline at end of file
        +World's First Remote Company. GitLab Inc. is an open-core company that operates GitLab, a DevOps software package which can develop, secure, and operate software.
        +
        +The open source software project was created by Ukrainian developer Dmytro Zaporozhets and Dutch developer Sytse Sijbrandij.
        \ No newline at end of file
        "  end_of_patch:true
        ```
        
        The raw patch data represents the changes made between two versions of a file. It uses the diff format, where additions are marked with a **`+`** sign and deletions are marked with a **`-`** sign.
        

- **What is `cache context` and what are its uses?**
    - Initially, the system was fetching the diff information from Gitaly every time it was needed. This became inefficient and problematic, especially when dealing with merge requests that had a large number of comments or revisions. For example, if there were a hundred comments on a merge request, it would require fetching the diff from Gitaly a hundred times, resulting in significant overhead.
    - To address this issue, the cache layer was introduced. The diff information is now persisted in the PostgreSQL database, allowing the system to retrieve it quickly without making repeated requests to Gitaly. By caching the diffs, the performance of fetching and rendering the merge request diff, as well as the discussion tabs and highlighting in the HTML, is greatly improved.
    - The cache context is defined within the `diffs_batch` action of the `projects::MergeRequests::DiffsController` class in `app/controllers/projects/merge_requests/diffs_controller.rb` file in `GitLab` codebase. Here is the following code snippet:
        
        ```ruby
        cache_context = [
              current_user&.cache_key,
              unfoldable_positions.map(&:to_h),
              diff_view,
              params[:w],
              params[:expanded],
              params[:page],
              params[:per_page],
              options[:merge_ref_head_diff]
            ]
        ```
        
        The cache context is used to check the freshness of the cache using the `stale?` method and I think it helps avoid serving stale cache when any of the variables in the context change.
        

- **Unfolding Files in the Diff: UI Layer and JSON Rendering**
    - So unfolding file in a diff are the action of expanding collapsed or we can say hidden section within the diff view to reveal additional content.
        
        The unfolding of files in the diff can be initiated by the UI layer. I think when a user interacts with the UI to expand a collapsed section, such as clicking on a toggle button or expanding a collapsed file, it triggers a requests to the server to fetch the unfolded content.
        
        Here is the following code snippet, the unfolding of files in done within `diffs_batch` action of the `Projects::MergeRequests::DiffController`
        
        ```ruby
        Gitlab::Metrics.measure(:diffs_unfold) do
          diffs.unfold_diff_files(unfoldable_positions)
        end
        ```
        
        The unfolding of files in the diff is initiated by the UI layer, and performed in the server-side controller action, and the unfolded data is serialised into JSON for rendering in the UI.
        
    

## Next steps

- Start writing the proto for `range-diff`.
- Implement a caching mechanism for RangeDiff (for later part of project).
- Here is the link to issue : [https://gitlab.com/gitlab-org/gitaly/-/issues/5123](https://gitlab.com/gitlab-org/gitaly/-/issues/5123)

### Links to relevant blogs and Video

- [Working with diffs](https://docs.gitlab.com/ee/development/merge_request_concepts/diffs/)
- [Commits are snapshots, not diffs](https://github.blog/2020-12-17-commits-are-snapshots-not-diffs/)
- [Highlights from Git 2.19](https://github.blog/2018-09-10-highlights-from-git-2-19/#compare-histories-with-git-range-diff)
- [Diffs and Commenting on Diffs](https://www.youtube.com/watch?v=K6G3gMcFyek)

Till next time,

Siddharth 🖖🏻