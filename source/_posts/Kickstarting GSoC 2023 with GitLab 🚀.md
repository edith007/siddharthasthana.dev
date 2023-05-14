---
title: Kickstarting GSoC 2023 with GitLab 🚀
date: 2023-05-15
---

TLDR: This blog contains the proposal that got me accepted in GSoC and [here](https://docs.google.com/document/d/16d9nUts_uflhpYrm7bFnp1no0Aym8wxLpmJCKxvchHY/edit?usp=sharing) it is!

## Selecting GSoC Organization

The GSoC 2023 organizations were announced on February 22, 2023, at 11:30 PM IST. I started looking for the organizations for submitting the proposal. Having actively engaged with GitLab community for over two years, I had developed a deep connection and strong admiration for the organizationa and its mission. Therefore, it was natural for me to choose [GitLab](https://about.gitlab.com/) as my primary option for GSoC 2023.

![Screenshot 2023-05-14 at 8.55.35 PM.png](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/Kickstarting%20GSoC%202023%20with%20GitLab%20%F0%9F%9A%80/Screenshot_2023-05-14_at_8.55.35_PM.png)

The incredible experience I had during **GSoC 2022**, collaborating with [Christian Couder](https://gitlab.com/chriscool) and [John Cai](https://gitlab.com/jcaigitlab) to [add support for mailmap in GitLab](https://summerofcode.withgoogle.com/archive/2022/projects/yaKP2iJK), was truly remarkable. The project aimed to enhance GitLab by incorporating support for Git’s mailmap feature. This feature enables the display of updated contributor information, ensuring that accurate details are presented  instead of potential outdated information associated with past commits. By implementing this capability, GitLab ensures that contributors who have modified their email address or name are correctly recognized as the same individual, resulting in a more comprehensive and accurate representation.

The following points are just to brag my contributions in GitLab 🙈🙈

- I successfully completed **GSoC 2022** with GitLab, working on adding mailmap support. You can check out my work in this repository: [https://gitlab.com/groups/gitlab-org/-/epics/8765](https://gitlab.com/groups/gitlab-org/-/epics/8765)
- I have been working on lots of backend and migrations issues. (230+ Merged MR 🚀)
    
    ![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/Kickstarting%20GSoC%202023%20with%20GitLab%20%F0%9F%9A%80/Untitled.png)
    
- Recently joined **[GitLab Core Team](https://about.gitlab.com/community/core-team/)** and also **[Heroes Team Member](https://about.gitlab.com/community/heroes/)**.

![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/Kickstarting%20GSoC%202023%20with%20GitLab%20%F0%9F%9A%80/Untitled%201.png)

- One of the top 10 contributors at GitLab in 2021 & 2022. [Here](https://about.gitlab.com/community/top-annual-contributors/) is the link for Top Annual contributors at GitLab.
- Recently selected in GitLab [Leading Organization Program](https://about.gitlab.com/handbook/marketing/community-relations/leading-organizations/).

## My proposal

When it came to choosing a project, I had a clear focus on my areas of interest. Considering my passion for projects involving C, Ruby, and/or Golang, along with my aspiration to gain research exposure, it was evident that applying to GitLab was my primary and only choice. I dedicated several days to thoroughly exploring the range of project opportunities they presented, and I was particularly drawn to a specific project that encompassed both **Git** and **Gitaly**.

After dedicating a significant amount of time to understanding the project requirements, I was able to prepare a well-rounded proposal for GSoC 2023. I believe sharing my [selected proposal](https://docs.google.com/document/d/16d9nUts_uflhpYrm7bFnp1no0Aym8wxLpmJCKxvchHY/edit?usp=sharing) can be beneficial to others who are considering participating in the program as well.

## My GSoC 2023 Project

My project aims to enhance the functionality of GitLab by enabling comprehensive differences to be displayed between two development branches. The focus is on addressing issues that arise when one branch is rebased on top of another, leading to potential conflicts and inconsistencies. To overcome this limitation, an RPC call will be implemented in Gitaly, a crucial component of GitLab, which will facilitate the use of the **`git range-diff`** command for a more comprehensive branch comparison. The result of the range-diff command will be sent back to GitLab for display. By integrating this new RPC call into Gitaly, GitLab will be equipped to showcase the range-diff on Merge Request (MR) pages, providing users with a clearer understanding of the changes made to the branch since its initial push. This project is valuable as it improves code reviewing effectiveness, allows for verification of unwanted changes during rebasing, and enhances the overall user experience in GitLab.

Here is the link to issue : https://gitlab.com/gitlab-org/gitaly/-/issues/5123

![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/Kickstarting%20GSoC%202023%20with%20GitLab%20%F0%9F%9A%80/Untitled%202.png)

I would be looking forward to work as good as possible this summer for GitLab under my mentors [Christian Couder](https://gitlab.com/chriscool) and [ZheNing He](https://gitlab.com/adlternative). Thanks to my mentors and the org to consider me worthy for their selection list. 😀

![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/Kickstarting%20GSoC%202023%20with%20GitLab%20%F0%9F%9A%80/Untitled%203.png)

For those who are interested in following the progress of our project and engaging in discussions, join the GitLab Discord Channel: [discord.gg/gitlab](http://discord.gg/gitlab)