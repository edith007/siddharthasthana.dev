---
title: GSoC'22 | GitLab
---

# From Selecting an Org to Getting Selected in GSoC 2022

TLDR: This blog contains the proposal that got accepted in GSoC and [here](https://docs.google.com/document/d/16zWn9zSv5r-O_ICZoKX0alMA4tXQs67hoc_CQEd9cJY/edit?usp=sharing) it is!

**Selecting my GSoC Organization**

The GSoC 2022 organizations were announced on March 7, 2022, at 11:30PM IST. I started looking for the organization for submitting the proposal. I got intrigued by the projects announced by organizations like GitLab, Homebrew and The Linux Foundation. The reason for selecting these organizations are as follows:

- GitLab
    - I had been actively contributing to GitLab for quite some time. I even started contributing to GitLab because I wanted to experience what it feels like to work on such a huge product having so many independent components. In their ideas list, they mentioned a project which gave me an opportunity to work on Gitaly, which provides RPC access to the git repositories. I decided that project since it provided the opportunity to learn RPC (a concept which would be really new to me), explore go programming language and even give me an opportunity to contribute to git as well!!!
        
        The following points are just to brag my contributions in GitLab 🙈🙈
        
        - I have been working on lots of backend and migrations issues. (170+ Merged MR 🚀)
            
            ![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/From%20Selecting%20an%20Org%20to%20Getting%20Selected%20in%20GSoC%20%2025fce351279d4a7ca8737f963422c731/Untitled.png)
            
        - I am also a proud member of [GitLab Heroes program](https://about.gitlab.com/community/heroes/)
            
            ![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/From%20Selecting%20an%20Org%20to%20Getting%20Selected%20in%20GSoC%20%2025fce351279d4a7ca8737f963422c731/Untitled%201.png)
            
    
- Homebrew
    - It is very popular as the default package manager for mac! So, almost everyone knows or have even heard about it. Knowing that it is an open source product and is largely written in Ruby really made me want to contribute to it!

- The Linux Foundation
    - I follow Greg KH on Twitter, and came across this tweet [https://twitter.com/gregkh/status/1353318632420478979](https://twitter.com/gregkh/status/1353318632420478979)
        
        ![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/From%20Selecting%20an%20Org%20to%20Getting%20Selected%20in%20GSoC%20%2025fce351279d4a7ca8737f963422c731/Untitled%202.png)
        
        I was amazed to see so many open bugs in the Linux Kernel and really wanted to apply to this mentorship and close some syzkaller bugs. Plus contribute to the Kernel powering things from a small toy to the Mars rover!! So, in order to prepare for it, I learned a bit about the Linux Kernel, things like writing a loadable kernel module and how to debug a crash given it’s stacktrace.
        
    
    - Among so many subsystems listed, I chose to prepare the proposal for the projects under perf subsystem. The reason for that being sometimes back a senior of mine working as a performance engineer mentioned one of Brendan Gregg's blogs on [perf Linux profiler](https://www.brendangregg.com/perf.html). I opened that blog and left that tab open so that I could come back to it later (just like other 300 open tabs on my system… *sigh* 😮‍💨). I thought since performance engineers use this, so might be something important, so wanted to look into it!
        
        

Once I was done selecting the orgs, the time came to understand the projects and prepare the proposals for them. After giving a lot of thought and exploring the projects in the above organizations, I realized I don’t have enough time to understand and explore three projects and create proposal for them! So, I went with just the one! GitLab!! The reason for that was very simple, I was not only getting to contribute to GitLab but to Git as well!! 🤩🤩

**My Proposal**

I started early and spent a lot of trying debugging to understand how Gitaly works and where it fits into the GitLab’s architecture. While doing so, I asked a lot of silly questions to my mentors as well! And they were always very kind and pointed me to the right direction! 

Once I got some idea of how Gitaly works, I started exploring Git (specifically `git-cat-file`, since my project is around that command).

PS: I used my favorite debugging tool to understand these projects — **print statements!!**

My aim was to come up with the first draft of my proposal as soon as possible, so that I can ask for feedback from the mentors on that project. The mentors were really very helpful in giving feedback and helping me improve the proposal. I think after 3 round of reviews, it was in a shape which could be submitted! Thanks a ton, Christian!! 🙇

[Here](https://docs.google.com/document/d/16zWn9zSv5r-O_ICZoKX0alMA4tXQs67hoc_CQEd9cJY/edit?usp=sharing) is the proposal that I submitted and am very grateful that it even got selected!

**The Big Announcement**

The result of GSoC 2022 was announced on May 20, 2022, at 12:00PM (+0530 GMT). When the time comes to browse the list, the GSoC website froze, my friends and I were constantly refreshing the website to see my result and finally after 25 minutes, I could finally see that dreams do come true when you religiously work for them.

![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/From%20Selecting%20an%20Org%20to%20Getting%20Selected%20in%20GSoC%20%2025fce351279d4a7ca8737f963422c731/Untitled%203.png)

I am looking forward to work with GitLab this summer with my amazing mentors [Christian Couder](https://gitlab.com/chriscool)
and [John Cai](https://gitlab.com/jcaigitlab) to implement the mailmap feature in GitLab. Thanks to my mentors and GitLab for believing in me to work on this project!
