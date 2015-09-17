---
layout: post
title: "Arbitrary Mode Scratch buffers for Emacs"
tags: programming
---

  In my current job, I do a fair amount of teaching about web development. Usually these are quick 10 minute lessons, and often times it is easiest to explain a concept with a code snippet. Being an emacs user, I typically open up a new temp buffer, set the major-mode js2-mode/web-mode/css-mode (for javascript, html/velocity/jsp and css respectively) and start typing.
  After doing this several times, the several steps became irritating. It seemed to me that emacs is all about customization, and I should be able to open an arbitrary buffer with one command. Seeing this as a perfect excuse to further enhance my elisp skills, I came up with the function below. Now I can just call this function[^1] and type in the mode-type and a buffer with all the correct settings is loaded.


  {% highlight common-lisp linenos %}
(defun create-scratch-buffer ()
  "creates a scratch buffer of specified type"
  (interactive)
  (let (newbuffer mode-type)
    (setq mode-type (read-from-minibuffer "scratch mode: "))
    (setq newbuffer (get-buffer-create (concat mode-type "-scratch")))
    (switch-to-buffer newbuffer)
    (funcall (intern (concat mode-type "-mode")))))

{% endhighlight %}

  There are probably many ways this function could be improved, but it sure is satisfying when I am able to improve my workflow using a tool that I wrote myself.

[^1]: I bound this to ```C-c C-s b```, for Create Scratch Buffer, it may or may not be appropriate for your keymap setup.
