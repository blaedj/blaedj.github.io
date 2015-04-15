---
layout: post
title: "Base64 decoding email attachments with Ruby"
date: 2015-04-15 14:35:30
categories: ruby quick_hacks
---

This may be a fairly uncommon situation, but have you ever needed to recover
an attachment from a deleted email? I had accidentally deleted an email
containing an excel spreadsheet that I still needed. For reasons I won't go
into here, the only copy of the email I had was a 'raw' text file of the
email. rather than mess around with trying to get Mail.app to recognize it as
an email, I decided to try and recover the base64-encoded excel file from the
email. 

First off, copy the base64-encoded version of the spreadsheet to a seperate
  file. It will be very long and look something like this:  
  {% highlight bash %}
  UEsDBBQABgAIAAAAIQCTW7VrkgEAABEHAAATAAgCW0NvbnRlbnRfVHlwZXNdLnhtbCCiBAIooAAC
  AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
  AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
  AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
  AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
  AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
  AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
  AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
  AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
  AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADM
  Vd1KwzAUvhd8h5JbWTMVRGTdLvy51MH0AWJytoalScg50+3tPc10iMyOsYHetLTJ+X5Oer4ORsvG
  FW+Q0AZfifOyLwrwOhjrZ5V4eX7oXYsCSXmjXPBQiRWgGA1PTwbPqwhYcLXHStRE8UZK1DU0CssQ
  wfPKNKRGET+mmYxKz9UM5EW/fyV18ASeetRiiOHgDqZq4ai4X/LrtRIuF8Xtel9LVQkVo7NaEQuV
  7arcWpfAYUfhmzc/1PU+lZVcmcGxthHPfmd4tX4/gjCdWg0m6EXDpkuMCZTBGoAaV8ZkuRdpAkTc
  cmxdPfFxJGugGKtEj6ph73Lp5HtI89cQ5mV3a7Y47BSQhZSNsv7Lcwd/3owy386PLKT1l4H31HHx
  T3Rc/pEO4pkDma+HH0mG2XEASCsHeOzPMIPuYq5VAjMhnpnZ0QV8x96hQyunb2semSM3YYPbxc9B
  ...
  ...
  {% endhighlight %}

Next open up a pry (or irb) console in the directory holding the file you
just created. 

We'll be using Ruby's Base64 utility, so 
{% highlight ruby %}
pry> require 'base_64'
=> true
{% endhighlight %}

Then we want to open the file we created: 
{% highlight ruby %}
  pry> encodedcontent = File.read('encoded_file.xlsx', 'wb') 
{% endhighlight %}

And write the unencoded version of it to a new file (the 'w' is for **w**rite mode, 'b' is for **b**inary):

{% highlight ruby %}
File.open('new_file.xlsx', 'wb') { |file| file << Base64.decode64(encodedcontent) }
{% endhighlight %}


and voila! you know have an open-able excel file! [^1]


[^1]: I had to open the file with OpenOffice, Excel complained that the file was un-openable.
