## Checkout

```bash
cd path/to/your/repos
git clone git@github.com:kernelkit/kernelkit.github.io.git blog
cd blog/
git submodule update --init
```

Make changes/additions on a separate branch:

```bash
git checkout -b my-changes
```

When you push it to GitHub you will get a question by GitHub.com if you
want to create a pull request.  Do that and follow the instructions.

## Setup

This blog use [Jekyll][0] with the [Chirpy theme][1].  Jekyll is written
in Ruby, so you need a fairly modern system to write and preview posts.
Verified to work on Linux Mint 21.3, based on Ubuntu 22.04 LTS:

 - [Install Jekyll](https://jekyllrb.com/docs/installation/)
 - Run `bundle` from the blog directory to install all deps


## Running

With everyting installed, starting the previewer is as simple as:

```bash
$ jekyll serve
Configuration file: /home/jocke/src/kernelkit.github.io/_config.yml
            Source: /home/jocke/src/kernelkit.github.io
       Destination: /home/jocke/src/kernelkit.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
                    done in 0.801 seconds.
 Auto-regeneration: enabled for '/home/jocke/src/kernelkit.github.io'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```

## Updating

Occasionally we update the Chirpy theme or its dependencies.  To be able
to continue using the previewer, you need to update your deps:

```bash
$ bundle update
Fetching gem metadata from https://rubygems.org/...........
Resolving dependencies...
Fetching concurrent-ruby 1.3.4 (was 1.3.3)
Fetching google-protobuf 4.27.4 (x86_64-linux) (was 4.27.2)
Fetching rexml 3.3.6 (was 3.3.2)
Fetching parallel 1.26.3 (was 1.25.1)
Installing concurrent-ruby 1.3.4 (was 1.3.3)
Installing google-protobuf 4.27.4 (x86_64-linux) (was 4.27.2)
Installing rexml 3.3.6 (was 3.3.2)
Fetching jekyll-theme-chirpy 7.1.0 (was 7.0.1)
Installing parallel 1.26.3 (was 1.25.1)
Installing jekyll-theme-chirpy 7.1.0 (was 7.0.1)
Bundle updated!
```

## Identity

All blog posts have one or more authors.  Make sure your nick is added
to the file `_data/authors.yml` on the format:

```yaml
jacky:
  name: Jacky Switch
  url: https://infix-os.net
```

## Blog

 1. New blog posts go in `_posts/yyyy-mm-dd-brief-title.md`
 2. Add front matter at the top of your post

        ---
        title: Longer title of post
        author: jacky
        date: 2022-11-20 09:03:20 +0100
        categories: [examples]
        tags: [cli]
        pin: false
        ---

 3. [Add content ...](https://chirpy.cotes.page/posts/write-a-new-post/)
 4. Use relevant tags and categories, check first!
 5. Preview

        jekyll serve -lw

> **Tip:** for work in progress, use the top-level directory `_drafts/`
> and add the `-D` option to `jekyll serve` to preview your post.

[0]: https://jekyllrb.com/
[1]: https://chirpy.cotes.page/
