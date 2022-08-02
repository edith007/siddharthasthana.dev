---
title: GSoC Week 6 & week 7 — Diving deep into GitLab and Gitaly!
date: 2022-08-02
---

Well Hello friend 👋🏻

 

I was not in capacity to publish by week 6 blogs as I was not able to focus much on work due to health issues. But, now I am feeling great 🦾. Now, let's talk about project. So, far in Git, my patch for adding support to mailmap in `git-cat-file` is merged in git `next` branch 🎉 and will soon be promoted to `master` 🚀. Thanks a ton Junio, Phillip, Đoàn, Ævar, Johannes, Christian and John for helping me with the reviews and making the patch better. Here is the link to the patch [https://public-inbox.org/git/20220718195102.66321-1-siddharthasthana31@gmail.com/](https://public-inbox.org/git/20220718195102.66321-1-siddharthasthana31@gmail.com/).

### The Mid-Term Evaluation 🖖

This first month of GSoC were totally exciting! I am also very happy that I have passed my GSoC midterm evaluation and got my first stipend 💵. Probably will buy myself a Green lightsaber ⭐🧔⚔️.

So, Now let's talk about the things in GitLab and Gitaly that have been working on this week!

### Contributors Graph 📊

As mentioned in my previous blog, I share some of my finding related to contributors graph, where GitLab is using `FindCommit` RPC when contributor's graph is loaded. I tried to dig deep into GitLab side of project to find out how GitLab and gitaly are interacting. I approached the search systematically,

- My approach was to first find out the routes which the contribution graph page is on, I visited the GitLab routes page [http://localhost:3000/rails/info/routes](http://localhost:3000/rails/info/routes) where I find out contribution graph is using `/*namespace_id/:project_id/-/graphs/:id` path and `projects/graphs#show` controller#Action. Now, I know when we visit the contribution graph page the controller that is called is `show()` and is defined in `graph_controller.rb`.
- Following is the snippet of the `show` function in `graph_controller.rb`. The most interesting happening here is the call to `fetch_graph()`.
    
    ```ruby
    def show
        respond_to do |format|
          format.html
          format.json do
            fetch_graph
          end
        end
      end
    ```
    
- Following is the snippet of the `fetch_graph` function.
    
    ```ruby
    def fetch_graph
        @commits = @project.repository.commits(@ref, limit: 6000, skip_merges: true)
        @log = []
    
        @commits.each do |commit|
          @log << {
            author_name: commit.author_name,
            author_email: commit.author_email,
            date: commit.committed_date.strftime("%Y-%m-%d")
          }
        end
    
        render json: @log.to_json
      end
    ```
    
    The first line of the function, makes a call to a function called `commits` and passes `ref` (which is master), `limit:6000` and `skip_merges:true`.  The `commits` function is defined in `app/models/repository.rb`.
    
- Following is the snippet of the `commits` function
    
    ```ruby
      def commits(ref = nil, opts = {})
        options = {
          repo: raw_repository,
          ref: ref,
          path: opts[:path],
          author: opts[:author],
          follow: Array(opts[:path]).length == 1,
          limit: opts[:limit],
          offset: opts[:offset],
          skip_merges: !!opts[:skip_merges],
          after: opts[:after],
          before: opts[:before],
          all: !!opts[:all],
          first_parent: !!opts[:first_parent],
          order: opts[:order],
          literal_pathspec: opts.fetch(:literal_pathspec, true),
          trailers: opts[:trailers]
        }
    
        commits = Gitlab::Git::Commit.where(options)
        commits = Commit.decorate(commits, container) if commits.present?
    
        CommitCollection.new(container, commits, ref)
      end
    ```
    
    so, the first thing we are doing here is to create the options to be issued in the git command. The arguments that we passed are used to set the corresponding options, and the options in our case will look like the following:
    
    ```json
    repo ==> <Gitlab::Git::Repository: flightjs/Flight>
    ref ==> master
    path ==>
    author ==>
    follow ==> true
    limit ==> 6000
    offset ==> 40
    skip_merges ==> true
    after ==>
    before ==>
    all ==> false
    first_parent ==> false
    order ==>
    literal_pathspec ==> true
    trailers ==>
    ```
    
    Then we make a call to `Gitlab::Git::Commit::where (options)`, which is defined in `lib/gitlab/git/commit.rb`. In the `where` function, we make a call to `log` function defined in `lib/gitlab/git/repository.rb` 
    
- Following is a snippet of the `log` function
    
    ```ruby
    def log(options)
        default_options = {
            limit: 10,
            offset: 0,
            path: nil,
            author: nil,
            follow: false,
            skip_merges: false,
            after: nil,
            before: nil,
            all: false        
        }
    
        options = default_options.merge(options)
        options[:offset] ||= 0
        limit = options[:limit]
        if limit == 0 || !limit.is_a?(Integer)
            raise ArgumentError, "invalid Repository#log limit: #{limit.inspect}"
        end        
        wrapped_gitaly_errors do
            gitaly_commit_client.find_commits(options)
        end 
    end
    ```
    
    as we can see, we are again updating the options and making a very interesting function call, `gitaly_commit_client.find_commits(options)`. This is the call to a function called `find_commits` defined in the gitaly client.
    
- Following is the snippet of `find_commits` function:
    
    ```ruby
          def find_commits(options)
            request = Gitaly::FindCommitsRequest.new(
              repository:   @gitaly_repo,
              limit:        options[:limit],
              offset:       options[:offset],
              follow:       options[:follow],
              skip_merges:  options[:skip_merges],
              all:          !!options[:all],
              first_parent: !!options[:first_parent],
              global_options: parse_global_options!(options),
              disable_walk: true, # This option is deprecated. The 'walk' implementation is being removed.          trailers: options[:trailers]
            )
            request.after    = GitalyClient.timestamp(options[:after]) if options[:after]
            request.before   = GitalyClient.timestamp(options[:before]) if options[:before]
            request.revision = encode_binary(options[:ref]) if options[:ref]
            request.author   = encode_binary(options[:author]) if options[:author]
            request.order    = options[:order].upcase.sub('DEFAULT', 'NONE') if options[:order].present?
    
            request.paths = encode_repeated(Array(options[:path])) if options[:path].present?
    
            response = GitalyClient.call(@repository.storage, :commit_service, :find_commits, request, timeout: GitalyClient.medium_timeout)
            consume_commits_response(response)
          end
    ```
    
    The `call` function here sends the request to Gitaly and invokes the `FindCommits` RPC there. This is how the RPC is called from GitLab. Gitaly will respond with the information from the commit objects which is further processed by GitLab and the contributors graph is generated!
    

Now, how does Gitaly extract information from the bare git repositories it interacts with?

To understand that, let's talk about a very important git plumbing command, `cat-file`. The command provides content or type and size information for repository objects. For example, we can execute `git cat-file -p HEAD`, and we will be getting all the information about the `HEAD` commit object. GitLab makes extensive use of this command across its features. We have an option for this command called`--batch`. This enables us to print object information and contents for each object provided on stdin. So, Gitaly keeps a `git cat-file --batch` process running. So, all we have to do is give this process the revisions, and it will provide content as per the type of the revision. So, in the `FindCommits` RPC, we first get all the revisions by issuing a `git-log` command along with all the options that we received from GitLab. Now that we have the revisions, we pass them to the stdin of the `git cat-file --batch` process and stream the information to GitLab.

Now that we know that we use `git-cat-file` to get information for constructing the contributors graph, we can just make use of my patches adding mailmap support to `git-cat-file`. But, first we must benchmark and  understand if my patches will incur any performance issues. My mentors suggested me to use `hyperfine` tool to benchmark the performance of `git-cat-file` with and without `--use-mailmap` option.

### Benchmarking git-cat-file

In order to benchmark and compare the performance of `git-cat-file` with and without `--use-mailmap` option in the `--batch` mode, I created two shell scripts.

The first one was just to invoke `git cat-file --batch`, it was named `[cat-file.sh](http://cat-file.sh)` and looks like following:

```bash
#!/bin/bash
git cat-file --batch <<-EOF
HEAD
HEAD~1
HEAD~2
HEAD~3
HEAD~4
v2.37.1
EOF
```

The second one was to invoke `git cat-file --use-mailmap --batch`, it was named `[cat-file-mailmap.sh](http://cat-file-mailmap.sh)`, and it looked like following:

```bash
#!/bin/bash
git cat-file --use-mailmap --batch <<-EOF
HEAD
HEAD~1
HEAD~2
HEAD~3
HEAD~4
v2.37.1
EOF
```

Then, I just passed these shell scripts to `hyperfine` for benchmarking using the following command:

```bash
hyperfine -N './cat-file.sh' './cat-file-mailmap.sh'
```

and got the following result

![Untitled](https://raw.githubusercontent.com/edith007/siddharthasthana.dev/main/source/_posts/week6%267/GSoC%20Week%206%20%26%20week%207%20%E2%80%94%20Diving%20deep%20into%20GitLab%20and%207b5668f3d1f84441b2776ca419d8d031/Untitled.png)

Comparing both the run for checking any performance implication. The benchmark test that shows using `cat-file` with in `--batch` is 1.02 times faster than without `--use-mailmap`, which I don’t think is much performance difference. But, I am waiting for my mentors suggestions about this analysis.

So, my next task will be to:

- Work on Gitaly side of project to make Gitaly use `--use-mailmap` implemented in `git cat-file` [https://gitlab.com/gitlab-org/gitaly/-/issues/4364](https://gitlab.com/gitlab-org/gitaly/-/issues/4364).

So yeah, that was the week 6 & 7. Thanks a lot for reading 🙂

Will be back next week with another blog, Peace! ✌🏻