---
layout: post
title:  "Using webpack-dev-server with (iOS) devices on the local network"
date:   2017-03-14
tags: ruby-on-rails webpacker webpack-dev-server development
---

I was having an issue when using `webpack-dev-server` (via
the [webpacker](https://github.com/rails/webpacker/) gem). It worked great on my
development machine, but I couldn't get the webpack bundles to load on other
devices, namely my iphone and ipad, that were on the same local network. The
steps I took to fix this are as follows, in the context of a rails
application. Note that this tutorial is very specific to iOS and
the
[WebBlock app](https://itunes.apple.com/us/app/weblock-adblock-for-apps-websites/id558818638),
but the general idea of using a proxy is applicable to other devices.


## 1. Specify the devServer.host option

There are a couple ways to do this. You can specify the `--host 0.0.0.0` flag
when running the `webpack-dev-server` command. I did this by adding 
`--host 0.0.0.0` to the string passed to ruby's `exec` in
`bin/webpack-dev-server`. 
     
The other way, which is probably the more 'webpack-y' way of doing things, is
to edit the webpack configuration file. Add `devServer: { host: '0.0.0.0' }`
to the exported config object: 
  {% highlight js %}
     module.exports = merge(sharedConfig.config, {
       devtool: 'sourcemap',
       devServer: {
         host: '0.0.0.0',
       },
     ...
     }
 {% endhighlight %}

## 2. Add the redirect rule to WebBlock app
In the 'Redirect' list in the [WebBlock app](https://itunes.apple.com/us/app/weblock-adblock-for-apps-websites/id558818638)
on your (iOS) device, add a 'URL' rule. The Redirect url should be
`0.0.0.0/bundles/*`, the Proxy IP should bethe local network IP of your
development machine, and the Proxy port should match the port that
`webpack-dev-server` is running on.

Restart your `webpack-dev-server` and your device should load the webpack
bundles! Note that you will probably see a bunch of `Origin <your-app-here>:8080
is not allowed by Access-Control-Allow-Origin` and other messages in the browser
console related to access control checks failing. These could probably be
eliminated by properly setting up [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS). However, these errors only affect the
auto-reloading feature of `webpack-dev-server`, and I'd spent enough time on
this already. I can live with reloading the page manually on my phone/ipad as
long as I could actually get the assets somehow.

