---
title: Week 2 & 3 — Taking first steps… Understanding and writing protobuf for the range diff RPC
date: 2023-06-22
tags: ["GSoC2023", "GitLab", "RangeDiff", "Diff", "Protobuf"]
---

In my previous [blog](https://siddharthasthana.dev/blog/week%201%20%E2%80%94%20exploring%20gitlab's%20diffs%2023405c63df6d4bdc8679945b03bfdf15/) post, I shared my plan to begin working on the `.proto` file for the `git range-diff` command. Over the course of these 2 week, I dedicated my time to gaining a comprehensive understanding of `protocol buffers` and the process of writing a `.proto` file. Furthermore, I explored the intriguing realm of `Gitaly` and how it leverages the power of protocol buffers.

## What are Protocol Buffer?

Protocol Buffers (or Protobuf) is a very amazing way of organising and sharing data between different programs and languages. *Think of them as a universal translator for data.* 

At it’s core, a Protocol Buffer is a schema that defines the structure of the data. This schema is defined in `.proto` file, where we can specify the data fields and their types. The magic happens when we use Protocol Buffer compiler, which takes the `.proto` file as input and generates code in your desired programming language. This generated code provides convenient APIs for working with the structured data. So, whether we’re using JavaScript, Go or any other supported language, we can generate the corresponding code and effortlessly work with the serialized data.

Let’s take an easy example of using Protocol Buffers (protobuf) with a small `.proto` file and generate the corresponding code in Golang

 

```protobuf
syntax = "proto3";

message Person {
  string name = 1;
  int32 age = 2;
}
```

- To start, I have created a file called `person.proto`. This file define the structure of the data we want to serialize.
- To generate code from the `person.proto` file we have to download Protobuf Compiler (`protoc`). We can download it from here accordingly https://github.com/protocolbuffers/protobuf.
- Since we want to generate code in Golang we should use this compiler https://github.com/protocolbuffers/protobuf-go and run the following commands
    
    ```bash
    protoc --go_out=. person.proto
    ```
    
    So this command tells `protoc` to generate Golang code (`- - go_out`) and output it in the current directory (`.`).
    
- After generating the code, we’ll find a `person.pb.go` file in the current directory. The file is a Golang source code file that contains the generated code for working with the Protocol Buffers data defined in the `person.proto` file.

## `Gitaly`'s back story

As GitLab's infrastructure scaled and the number of users accessing the **`.git`** directory grew, the company encountered several challenges. Initially, they implemented horizontal scaling by deploying multiple server instances behind a load balancer. However, this approach eventually became a bottleneck. Another problem arose from the shared access to the critical **`.git`** directory, which caused potential downtime if the hosting server failed and led to performance issues due to high read/write activity. To tackle these issues, GitLab introduced `Gitaly`. This innovative solution involved creating a fail-safe and optimized layer around the **`.git`** directory. Instead of direct interaction, Gitaly acted as a server-side proxy, managing all requests and performing operations on the server-side before transmitting the results over the network. By centralizing operations and optimizing server-side processes, Gitaly significantly improved GitLab's performance and efficiency.

## Current Progress 🏗️

- I have started working on the `rangediff.proto` file. So my initial set of tasks is to compare two versions of a merge request using `git range-diff`. When implementing this functionality as an RPC, we need to define the necessary parameters to perform the comparison. These parameters will include:
    - **`start_oid_old`**: Starting commit OID of the old version.
    - **`head_oid_old`**: Latest commit OID of the old version.
    - **`start_oid_new`**: Starting commit OID of the new version.
    - **`head_oid_new`**: Latest commit OID of the new version.
    
    By obtaining these commit OIDs for both versions, we can pass them as parameters to the RPC implementation, specifically to the **`git range-diff`** command. This command is responsible for executing the comparison between the two versions and generating a comprehensive report detailing the changes made. The report includes information on `added`, `removed`, `modified`, and `unmodified` lines, providing valuable insights into the differences between the two versions of the merge request.
    
    Here is the link to the MR: https://gitlab.com/gitlab-org/gitaly/-/merge_requests/5918
    

- **How Gitaly compiles the proto file to generate corresponding golang code.**
    
    Thanks to ZheNing for asking me to go through it! Gitaly compiles proto files to generate corresponding Golang code using the protoc compiler. The [Makefile](https://gitlab.com/gitlab-org/gitaly/-/blob/master/Makefile) in the Gitaly repository provides the necessary instructions and tools for this process.
    
    Let’s focus on the part related to compiling the proto file and generating the corresponding Go code.
    
    To generate Go code from a proto file, Gitaly uses the `protoc` compiler along with the `protoc-gen-go` and `protoc-gen-grpc` plugins. Here is a code snippet of Makefile:
    
    ```makefile
    # Directories
    SOURCE_DIR       := $(abspath $(dir $(lastword ${MAKEFILE_LIST})))
    PROTO_DEST_DIR   := ${SOURCE_DIR}/proto/go
    
    # Tools
    PROTOC            := ${TOOLS_DIR}/protoc
    PROTOC_GEN_GO     := ${TOOLS_DIR}/protoc-gen-go
    PROTOC_GEN_GO_GRPC := ${TOOLS_DIR}/protoc-gen-go-grpc
    
    # protoc target
    PROTOC_VERSION      ?= v23.1
    PROTOC_BUILD_DIR    ?= ${DEPENDENCY_DIR}/protobuf/build
    PROTOC_INSTALL_DIR  ?= ${DEPENDENCY_DIR}/protobuf/install
    
    # ...
    
    # protoc installation
    ${PROTOC_INSTALL_DIR}/.installed:
        # Download, build, and install protoc
        # ...
    
    # Generate Go code from proto files
    ${PROTO_DEST_DIR}/%.pb.go: ${SOURCE_DIR}/proto/%.proto ${PROTOC_INSTALL_DIR}/.installed
        @mkdir -p $(@D)
        ${PROTOC} \
            --proto_path=${SOURCE_DIR}/proto \
            --go_out=paths=source_relative:${PROTO_DEST_DIR} \
            --go-grpc_out=paths=source_relative:${PROTO_DEST_DIR} \
            $<
    ```
    
    Here’s how the process works:
    
    - The root directory of the Gitaly source code is identified as the source directory (`SOURCE_DIR`)
    - The Go code is generated and placed in the `proto/go` directory within the source directory.
    - The necessary tools and plugins for the compilation process are specified, which includes `protoc` (the protobuf compiler) as well as the `protoc-gen-go` and `protoc-gen-go-grpc` plugins.
    - The target for generating Go code from a proto file is defined using a pattern rule. It specifies that for any **`.proto`** file in the **`proto`** directory, a corresponding **`.pb.go`** file should be generated in the **`PROTO_DEST_DIR`**.
    - The **`PROTOC`** command is executed with the appropriate arguments to generate the Go code. The **`--proto_path`** flag specifies the directory where the **`.proto`** file is located, and the **`--go_out`** and **`--go-grpc_out`** flags specify the output directories and options for the generated Go code.
    
    To trigger the generation of Go code from proto files, we can run the following command in the Gitaly source directory:
    
    ```go
    make proto
    ```
    
    This will invoke the **`proto`** target defined in the Makefile, which will compile the proto files and generate the corresponding Go code. The generated Go files will be placed in the **`proto/go`** directory.
    
    Additionally, the `Makefile` includes other build and test configurations, such as Go package tests, test coverage reports, and build options for dependencies like `Git` and `libgit2`.
    

## Next step

- Work on the merge request for adding `rangediff.proto` in Gitaly.

Till next time,

Siddharth 🖖🏻