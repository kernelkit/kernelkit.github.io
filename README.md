## Checkout

    cd path/to/your/repos
    git clone git@github.com:kernelkit/kernelkit.github.io.git blog
    cd blog/
    git submodule update --init

Make changes/additions on a separate branch:

    git checkout -b my-changes

When you push it to GitHub you will get a question by GitHub.com if you
want to create a pull request.  Do that and follow the instructions.

## Setup

This blog use [Jekyll][0] with the [Chirpy theme][1].  Jekyll is written
in Ruby, so you need a fairly modern system to write and preview posts.
Verified to work on Linux Mint 21.3, based on Ubuntu 22.04 LTS:

 - [Install Jekyll](https://jekyllrb.com/docs/installation/)
 - Run `bundle` from the blog directory to install all deps

## Blog

 1. New blog posts go in `_posts/yyyy-mm-dd-brief-title.md`
 2. Add front matter at the top of your post

        ---
        title: Longer title of post
        date: 2022-11-20 09:03:20 +0100
        categories: [examples]
        tags: [cli]
        pin: false
        ---

 3. [Add content ...](https://chirpy.cotes.page/posts/write-a-new-post/)
 4. Use relevant tags and categories, check first!
 5. Preview

        jekyll serve -lIw

[0]: https://jekyllrb.com/
[1]: https://chirpy.cotes.page/
