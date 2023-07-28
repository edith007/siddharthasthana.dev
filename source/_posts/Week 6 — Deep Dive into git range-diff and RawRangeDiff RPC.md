---
title: Week 6 — Deep Dive into git range-diff and RawRangeDiff RPC
date: 2023-07-28
tags: ["GSoC2023", "GitLab", "RangeDiff", "git-range-diff", "RawRangeDiffRPC"]
---

The past couple of weeks were quite an adventure for me. In the midst of the week 4 & 5, I found myself immersed in a series of back-to-back interviews. These interviews, while exciting, did pull my focus away from the GSoC project for a short while. I am grateful to Christian and ZheNing, my mentors, who understood the situation and extended the project timeline by two weeks. This shift was a lifesaver, giving me the necessary time to fully commit to both my interviews and the GSoC project without compromise.
With the interviews now behind me, I was finally able to dive back into my GSoC project in earnest. I’ve been diving deeper into the nitty-gritty of the range-diff Merge Request. I’m excited to share with you what I’ve learned and the decision we’ve made towards implementing the feature!

## Exploring the Different Modes of git range-diff

At the beginning of the week, I was in discussion with Christian, and he mentioned three possible ways to use the git range-diff command.

 1. `<range1> <range2>`: Here, either commit range can be of the form `<base>..<rev>`, `<rev>^!`, or `<rev>^-<n>`.

1. `<rev1>...<rev2>`: This is equivalent to `<rev2>..<rev1> <rev1>..<rev2>`.
2. `<base> <rev1> <rev2>`: This is equivalent to `<base>..<rev1> <base>..<reb2>`.

He suggested to make an enum to select any of these modes. The three modes would be **`2_RANGES`**, **`2_REVS`**, and **`BASE_AND_REVS`**, corresponding to the three ways to pass commit ranges listed above. We agreed to not support the **`2_RANGES`** mode initially due to its complexity and potential lack of immediate need for GitLab. 

```protobuf
message RangeDiffRequest {
  // Enum to specify the type of commit range notation.
  enum RangeNotation {
    // Specifies the <range1> <range2> notation.
    TWO_RANGES = 0;
    // Specifies the <rev1>...<rev2> notation.
    TWO_REVS = 1;
    // Specifies the <base> <rev1> <rev2> notation.
    BASE_AND_REVS = 2;
  }
  // Other fields...
}
```

In the middle of the week, we had another productive discussion on `RangeDiffRequest` structure. Christian pointed out that our current design was missing the fields for `<range1>`, `<range2>`, `<rev1>` and `<rev2>`. He suggested that we only needed three fields:

- `base_rev` for `<base>` when using `BASE_AND_REVS` mode,
- `r_1` for `<range1>` in `TWO_RANGES` mode or for `<rev1>` in other modes, and
- `r_2` for `<range2>` in `TWO_RANGES` mode or for `<rev2>` in other cases.

This suggestion was indeed insightful and well-thought. Therefore, I agreed with his proposition and updated our `RangeDiffRequest` message to have `base_rev`, `r_1` and `r_2` fields. However, Christian raised another concern about the types of **`r_1`**, **`r_2`**, and **`base_rev`** fields. Originally, I intended to use **`GitCommit`** for these fields, but Christian pointed out that a **`GitCommit`** might not contain all the information that a **`<range>`** can contain. He advised to use strings for these fields to ensure more flexibility. We all agreed that parsing these strings depending on the **`range_notation`** value would give us more flexibility and might be simpler in the end.

```protobuf
message RangeDiffRequest {
  // Other fields...
  // The base commit. Used only in case of BASE_AND_REVS.
  string base_rev = 3;
  // The first commit range or revision.
  string r_1 = 4;
  // The second commit range or revision.
  string r_2 = 5;
}
```

After finalizing the structure for **`RangeDiffRequest`**, I started to implement the **`RawRangeDiff`** RPC function. This function uses the **`RangeDiffRequest`** to perform a git range-diff and returns the raw output stream of the range-diff command. I added a new **`RawRangeDiffResponse`** message field in **`rangediff.proto`** that holds the data from the raw range-diff output.

```protobuf

rpc RawRangeDiff(RangeDiffRequest) returns (stream RawRangeDiffResponse) {
    option (op_type) = {
      op: ACCESSOR
    };
}

message RawRangeDiffResponse {
  bytes data = 1;
}

```

## Introducing RawRangeDiff RPC

Now, let’s dive into understanding the `RawRangeDiff` RPC that I’ve been working on. This function lies at the core of the improvements we are making.

First and foremost, **`RawRangeDiff`** RPC validates the incoming request and assigns values to **`rev1`** and **`rev2`** based on the **`range_notation`** specified. Although the **`range-diff`** command supports **`TWO_RANGES`** and **`BASE_AND_REVS`** modes, for now, we've chosen to prioritize the **`TWO_REVS`** mode, leaving the others for potential future exploration.

Below is the initial structure of our **`RawRangeDiff`** RPC:

```go
func (s *server) RawRangeDiff(in *gitalypb.RangeDiffRequest, stream gitalypb.RangeDiffService_RawRangeDiffServer) error {
  // Validate request and assign rev1 and rev2
  // Create the range-diff command
  cmd := git.Command{
    Name:  "range-diff",
    Flags: []git.Option{},
    Args:  []string{rev1 + "..." + rev2},
  }
  // Rest of the function...
}
```

Once the **`range-diff`** command is formed, **`RawRangeDiff`** RPC executes it. The output stream we get is untouched, offering a snapshot of the raw output generated by **`git range-diff`**. This raw output is intended for use by other services or user interfaces, allowing tailored processing for various unique requirements.

So, in the **`RawRangeDiff`** RPC, the assignment of **`rev1`** and **`rev2`** depends on the **`range_notation`** provided in the **`RangeDiffRequest`**. If the **`range_notation`** is **`TWO_REVS`**, **`rev1`** and **`rev2`** are interpreted as two revision points, and the range-diff is calculated between these two points.

## Making Our Terms Clear

Another important thing we did was to take a closer look at how we were defining the relationships between old and new commits. The terms we were using, `NO_CHANGE` and `CHANGES`, were a bit too vague and could be confusing.

So, we decided to use terms from the [Git documentation](https://git-scm.com/docs/git-range-diff) itself to clear up any confusion. Now, we use `unmodified` to represent `=`, meaning nothing has changed, and `modified` for `!`, which means there are changes. Hopefully, this will make things much easier to understand.

## Next step!

- Enhance the existing **`.proto`** file
- Implement the RPC on the Gitaly Ruby side
- Begin work on the client-side of the **`RawRangeDiff`** function

Till next time,

Siddharth 🖖🏻