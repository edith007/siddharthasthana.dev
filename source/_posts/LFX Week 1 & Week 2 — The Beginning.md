---
title: The Beginning! Progress on Cortex's Importer Service
date: 2023-04-04
tags: ["LFX", "Cortex", "Prometheus"]
---

![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/LFX_week1%26week2_blogpost/LFX%20Week%201%20%26%20Week%202%20%E2%80%94%20The%20Beginning%20Progress%20on%20Co%20c1a39088f5a04e4cbb31eecb2df852e8/Untitled.png)

## Selecting my LFX Organization

On February 1, 2023, the LFX Spring 2023 organizations were announced, and I wasted no time in searching for organizations that would be a good fit for me to submit a proposal to. As a LFX Mentorship participant, I was allowed to submit up to three proposals in one coherent submission. After careful consideration of the various projects announced by different organizations, I was particularly drawn to the proposals put forth by [**Konveyor**](https://github.com/konveyor/move2kube), [**Kyverno**](https://github.com/kyverno/policies), and [**Cortex**](https://github.com/cortexproject/cortex). The reason for selecting these organizations are as follows:

- **Konveyor**
    - **Project** → https://github.com/konveyor/move2kube/issues/604
        - I find the project idea proposed by Konveyor particularly interesting, as it revolves heavily around **Git** – a technology I have gained some experience in while contributing to a previous project during my time as a **Google Summer of Code** participant. The familiarity I have with Git would allow me to hit the ground running with this project, and potentially enable me to make a valuable contribution to the team.
    
- **Kyverno**
    - Project → https://github.com/kyverno/policies/issues/491
        - The project aims to develop an organizational system, potentially including a custom application and process, to ensure that all Kyverno policies in the main branch of the `kyverno/policies` repository are reflected, searchable, and have complete metadata on [artifacthub.io](http://artifacthub.io/).
        - Kyverno is one of the most popular Kubernetes policy engines, and contributing to its policy library can help improve the security and efficiency of Kubernetes-based applications for many users around the world.

- **Cortex**
    - Project → https://github.com/cortexproject/cortex/issues/4956
        - There might be a lot of cases where the users might already be using Prometheus and later decides to switch to Cortex. Cortex will take care of the metrics which will be generated after the switch, but at present there is no way to backfill the old Prometheus TSDB blocks to Cortex. This feature adds the ability in cortex to fetch the Prometheus TSDB blocks and backfill the data.

After selecting the projects, I began to study and understand them before preparing my cover letter. I submitted my resume and cover letter to all three chosen projects, namely cortex, kyverno, and Konveyor. Shortly after, I received an assignment from Konveyor to write a project proposal, which included answering questions about technical and non-technical aspects of the Move2Kube project, as well as discussing any relevant approaches or PoCs I had previously worked on.

I submitted my proposal and received an email from Mehant, one of the mentors of Move2Kube, to discuss the project further. The next day, I received another email from **Cortex**, informing me that I had been selected to work on their project 🎉🚀

![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/LFX_week1%26week2_blogpost/LFX%20Week%201%20%26%20Week%202%20%E2%80%94%20The%20Beginning%20Progress%20on%20Co%20c1a39088f5a04e4cbb31eecb2df852e8/Untitled%201.png)

## About Cortex 📈

`Cortex` is an important project for the Kubernetes community because it provides a horizontally scalable, long-term storage solution for `Prometheus`. Prometheus is a popular monitoring system for Kubernetes-based applications, but it is designed to store data for a relatively short period of time. Cortex extends the capabilities of Prometheus by allowing users to store metrics data for much longer periods, without sacrificing the ability to query and visualize that data in real time.

With Cortex, users can create a highly available, horizontally scalable cluster of Prometheus servers, and store all of their metrics data in a centralized, long-term storage layer. This allows users to keep track of historical data, and to perform deep analysis on that data over time, which can help to identify trends, predict future issues, and optimize application performance.

Cortex also provides a number of other features that are useful for Kubernetes users, such as `multi-tenancy support`, `horizontal scaling`, and easy integration with popular Kubernetes tools and platforms. As Kubernetes continues to grow in popularity and becomes the de facto platform for container orchestration, the need for scalable, long-term storage solutions like Cortex will only increase.

## Feature to Work On! 💻

Prometheus is an open-source monitoring tool for collecting time-series data, while Cortex is a horizontally scalable and highly available version of Prometheus that can handle large-scale deployments. Cortex can store, query, and analyze metrics data in a distributed manner. There might be scenarios where users might already be using Prometheus and later decides to switch to Cortex. However, there is currently no way to backfill the old Prometheus TSDB blocks to Cortex. TSDB stands for Time-Series Database, which is a database optimized for storing and querying time-series data.

The proposed solution is to add the ability to Cortex to fetch the old Prometheus TSDB blocks and backfill the data into the Cortex system. This feature will enable users to migrate to Cortex without losing their historical data. This will also allow them to take advantage of Cortex's scalability, fault tolerance, and multi-tenancy features.

## Proposal 📑

To achieve this, a new `/import` endpoint will be added to the Cortex API, which allows users to upload the old Prometheus TSDB blocks to Cortex. The import process will be initiated using the `cortex-cli` tool, which will take care of uploading the blocks to the Cortex cluster. The blocks can be stored in either local storage or remote storage, depending on the user's choice. During the import process, Cortex will upload the old TSDB blocks in one request. This will help to avoid timeouts and other issues that may occur during the upload process. The chunks will be uploaded in parallel, which will help to improve the overall performance of the import process.

Once the import process is complete, the user can verify that the blocks have been successfully imported by querying the Cortex API. The imported data will be available for querying, visualization, and analysis in Cortex, just like any other metric data ingested by Cortex.

## The Road Ahead 🛣️

Since this project is a feature request, As a first step, we're preparing a proposal to outline our plan and get feedback from the community. Once the proposal is approved, we'll proceed to the coding phase of the mentorship. Here is the link to the proposal https://github.com/cortexproject/cortex/pull/5229

### References

- [https://cortexmetrics.io/docs/](https://cortexmetrics.io/docs/) – Cortex Documentation
- [https://www.cncf.io/blog/2018/12/18/cortex-a-multi-tenant-horizontally-scalable-prometheus-as-a-service/](https://www.cncf.io/blog/2018/12/18/cortex-a-multi-tenant-horizontally-scalable-prometheus-as-a-service/) – Introduction of Cortex
- [https://grafana.com/blog/2020/08/12/scaling-prometheus-how-were-pushing-cortex-blocks-storage-to-its-limit-and-beyond/](https://grafana.com/blog/2020/08/12/scaling-prometheus-how-were-pushing-cortex-blocks-storage-to-its-limit-and-beyond/) – how Grafana is pushing the limits of Cortex blocks storage to scale Prometheus for larger deployments.
- [https://grafana.com/blog/2020/07/16/how-the-cortex-and-thanos-projects-collaborate-to-make-scaling-prometheus-better-for-all/](https://grafana.com/blog/2020/07/16/how-the-cortex-and-thanos-projects-collaborate-to-make-scaling-prometheus-better-for-all/) – about scaling Prometheus using Cortex blocks storage
- [https://opstrace.com/blog/scaling-200m-series](https://opstrace.com/blog/scaling-200m-series)
- [https://www.youtube.com/watch?v=f8GmbH0U_kI&t](https://www.youtube.com/watch?v=f8GmbH0U_kI&t=1230s) – Highly recommended!!!
- [https://www.youtube.com/watch?v=1OVpogSKxyI](https://www.youtube.com/watch?v=1OVpogSKxyI) – Better Scalability & More Isolation? The Cortex “Shuffle Sharding” Story.
- [https://www.youtube.com/watch?v=u1SfBAGWHgQ](https://www.youtube.com/watch?v=u1SfBAGWHgQ) ****– Current State and the Future of Cortex

So yeah, that was the week 1 & 2. Thanks a lot for reading 🙂