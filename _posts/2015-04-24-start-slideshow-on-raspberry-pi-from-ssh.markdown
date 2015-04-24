---
layout: post
title:  "Starting (and restarting) a slideshow on a remote raspberry pi"
date:   2015-04-24 
categories: raspberry-pi ssh openvt fbi
---

I recently got my first raspberry pi, its being used to run a simple display board at work. Newsletters
and announcements are converted to image form and then displayed in a slideshow. After some trial and error
I came upon the ```fbi``` command, which displays images on a screen directly, no need for an
X-session. This has worked really well, I have a script that displays all the .png files in a directory:

{% highlight bash linenos%}
#!/bin/bash
SLIDESHOWDIR="/home/me/slideshow"
INTERVAL=25 # slideshow interval in seconds
fbi -a -t $INTERVAL -u `find $PHOTODIR -iname "*.png"`
{% endhighlight %}

A little explanation:  

  *    ```-a``` enables autozoom, fbi picks a reasonable zoom factor when loading a new image
  *    ```-t $INTERVAL``` sets the slideshow interval
  *    ```-u``` randomizes the order of the filenames
  *    ```find $PHOTODIR -iname "*.png"``` is encased in backticks[^1], so it is run before ```fbi``` and the output is used by ```fbi```. In this case we are simply finding all the .png files in the slideshow directory.


The problem arises when I add or remove a file from the slideshow directory via ssh. I don't want to have to physically plug in a keyboard to my RPi everytime I want to restart the slideshow, but I couldn't seem to get it to work any other way. This is where the ```openvt``` command comes in
{% highlight bash %}
$ sudo openvt -f -c 1 ./slideshow.sh
{% endhighlight %}

```openvt``` runs a command on a new virtual terminal (VT):  

  * ```-f``` forces the opening of a virtual terminal without checking if it is already in use
  * ```-c``` specifies the virtual terminal to use (1 in this case)
  * ```./slideshow.sh``` is my ```fbi``` script from above.

Now changing the slideshow is as simple as ssh-ing in, changing the pictures in the slideshow directory, and running the above command (stashed in a ```restart_slideshow.sh``` file)




















[^1]: I've seen some interesting things about ```$(...)``` vs <code>`...`</code>. The ```$( )``` syntax seems to be nice in that is supports nesting of commands and escaping better, but for this simple case backticks work perfectly fine. Note that IANA bash/zsh/sh expert.
