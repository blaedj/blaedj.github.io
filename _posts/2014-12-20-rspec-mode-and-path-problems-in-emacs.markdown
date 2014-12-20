---
layout: post
title: "RSpec-mode, Magit and Path problems in Emacs"
categories: programming emacs
---

I have been using [rspec-mode](https://github.com/pezra/rspec-mode) to run my specs lately.
It's nice to get the spec feedback without having to tab out to a terminal (or
running a terminal in emacs). <code>C-c ,m</code> works great, it runs
all the specs for the current buffer. More and more, I've been seeing the value
in [Magit-mode](http://magit.github.io/). I always loved the ease in which magit lets
you choose hunks of files to commit, but I just learned that you can
stage *arbitrary sections* of a hunk! Simply mark (highlight) the section
that you want to stage for commit, and then press <code>s</code> in
magit-status-mode. So handy!

However, after a computer restart nothing seemed to work
correctly. Magit-mode was complaining that
it couldn't find the git executable.  I had recently moved Apple's
<code>/usr/bin/git</code> to <code>/usr/bin/git2</code> so that the system
would use Homebrew's version of git (I upgraded because of a
[vulnerability](http://article.gmane.org/gmane.linux.kernel/1853266) found in git clients).  Turns out that my PATH variable
for emacs was messed up. For future reference, the 'path' variable in
emacs is called 'exec-path'. PATH = exec-path. Took me **way** too
long to figure that out. The exec path wasn't including
<code>/usr/local/bin/</code>, the directory that homebrew puts all
it's executables (or at least symlinks them there).  The fix for this
was as simple as evaluating <code>(push "/usr/local/bin"
exec-path)</code> in an elisp scratch buffer. A more permanent fix was
to add this to my 'platform-specific.el' file in my emacs.d:

{% highlight common-lisp linenos %}
(if (eq system-type 'darwin)
	...
	;; add homebrew (/usr/local/bin) to the exec path
	(push "/opt/local/bin" exec-path))

{% endhighlight %}

The error that I was seeing for rspec-mode was something like
<code>your ruby version is 2.0.0 but your gemfile specified
 2.1.4.</code>
 I eventually realized that the command being run in the rspec-compilation
 buffer was <code>bundle exec rspec spec/path/to/specs/run</code>.
 Before I had been seeing <code>spring rspec spec/path/to/specs</code>.
 Simply running <code> spring rails server </code> or any other spring command [^1]
 in a terminal started up spring properly.After that rspec mode seemed to know
 to use spring to run the rspec commands, which solved the ruby
 version dependency.  This could probably be solved properly via some [RVM](https://rvm.io/)
 or [RVM.el](https://github.com/senny/rvm.el) configuration, but hey, my specs run nicely now!

[^1]: Spring is a rails application preloader that makes your specs and other rails commands run much faster, and is included in rails 4. [github.com/rails/spring](https://github.com/rails/spring)
