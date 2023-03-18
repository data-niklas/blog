+++
title = "Your own blog"
author = ["Niklas Loeser"]
date = 2022-07-25
lastmod = 2022-07-30T23:00:48+02:00
tags = ["hugo", "org", "caddy"]
categories = ["emacs"]
draft = false
weight = 1001
+++

Have you ever wanted to write a blog?
Are you using Emacs and know the power of org-mode?
You can setup your very own blog in just a few minutes.


## Hugo {#hugo}

Hugo has many themes, which change the layout and design of your blog. I will use the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme.

```sh
ssh yourserver

# Create your new blog
hugo new site blogdirectory
cd blogdirectory
ls
# public will contain your static website

# add the tania theme
git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod --depth=1
rm config.toml
# retrieve tania's sample config
wget https://raw.githubusercontent.com/adityatelange/hugo-PaperMod/exampleSite/config.yml

# now change all of the values to your preferred values
# e.g.: change the domain and blog name
# for it to work with Emacs, change the /articles/ url to /posts/

# add your page icon to /static/favicon.ico

# adds monokai as an available theme for syntax highlighting
hugo gen chromastyles --style monokai > themes/PaperMod/assets/css/extended/monokai.css
```
<div class="src-block-caption">
  <span class="src-block-number">Code Snippet 1:</span>
  Hugo installation script
</div>

Add the following to your config to use the monokai syntax highlighting

```yaml
params:
  assets:
    disableHLJS: true

markup:
  highlight:
    # anchorLineNos: true
    codeFences: true
    guessSyntax: true
    lineNos: true
    # noClasses: false
    style: monokai
```


## Emacs {#emacs}

To be able to convert org-mode files to Hugo, install `ox-hugo`. Then add the following snippet to your Emacs config:

```emacs-lisp
(defun hugo-export ()
  "Export current file via hugo"
  (interactive)
  (org-hugo-export-to-md)
  (let* ((keyword-map #'(lambda (keyword)
                          (,(org-element-property :key keyword)
                            ,(org-element-property :value keyword))))
         (org-keywords (org-element-map (org-element-parse-buffer) keyword keyword-map))
         (keyword-filter #'(lambda (l) (equal "HUGO_BASE_DIR" (car l))))
         (filtered-keywords (seq-filter keyword-filter org-keywords))
         (dir (cadr (car filtered-keywords))))
    (cd dir)
    (shell-command "hugo")
    )
  )
```

The snippet will export the current file to your Hugo blog and rebuild the website. You can also bind the function to any hotkey.

Now create a new org-mode file, e.g.: `my_sample_post.org` and create a simple structure:

```org
#+title: Your very first post
#+hugo_base_dir: emacs_path_to_your_hugo_blog_directory

#+date: currentdate

* Title
```

`hugo_base_dir` needs to point to your `blogdirectory` dir. To point to your server, use: `/ssh:serverdnsorip:blogdirectory`. This will work, if `blogdirectory` is in your home folder. Else adjust the path. Then edit your blog and run `hugo-export` when you are done.


## Caddy {#caddy}

Add the following snippet to your `Caddyfile`.

```caddy
blog.data-niklas.de {
	root * /path/to/blogdirectory/public
	file_server
}
```


## Syntax {#syntax}

You will be able to use existing org-mode elements. For code blocks just use

```org
#+begin_src java
foo.bar();
#+end_src
```

```org
| org-mode | table
| fine!    | even really long elements |


#+begin_src org
#+begin_quote
This is a nice quote
#+end_quote
```

> This is a nice quote[^fn:1]

```org
Footnotes[fn:n]

[fn:n]Footnote text
```

```java
int i = 3;
Car car = new Car();
car.bark();
```

![](/ox-hugo/UnixMagic.jpg)[^fn:2]

[^fn:1]: Though how do I add the source of the quote?
[^fn:2]: <https://archive.org/details/unix-magic-poster-gary-overcare-1>
