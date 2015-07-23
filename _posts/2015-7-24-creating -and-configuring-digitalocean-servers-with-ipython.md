---
layout: post
title: Creating and configuring digitalocean droplets with ipython notebooks.
---

I recently had an extremely large web scraping gig where I required 4 VPS at once. The client wanted to use [digitalocean](https://www.digitalocean.com/?refcode=da3d5d4ab00c) since you pay for the VPS by the hour, instead of by the month. To make the job more difficult, as the job was progressing the client would specify changes. To make these changes I would need to ssh into each VPS and make a change. Also on digitalocean you pay for the VPS whether it is poweredup or off, but you do not pay for the storage of snapshots. Since the client wanted to scrape for only a few hours a day, it was in my best interest to be able to bring the VPS up, run my scrapers and then when finished power it off, and take a snapshot as quickly as possible. Time is money. 

Once each VPS was up and running, controling each VPS was a breeze with [rpyc](https://rpyc.readthedocs.org/en/latest/), it was how to run shell commands on each VPS to set things up. In the rpyc documentation it mentions another module called [plumbum](http://plumbum.readthedocs.org/en/latest/) which uses a python syntax to do shell commands. More importantly it will ssh into another machine and run them there. You can do ssh commands as shown here [](http://plumbum.readthedocs.org/en/latest/remote.html#remote-machines)

{% highlight python linenos=table %}
from plumbum import SshMachine
rem = SshMachine("hostname", user = "john", keyfile = "/path/to/idrsa")
{% endhighlight %}

Creating the actual droplet is simple since digitalocean has python binding to their api [digitalocean-api](https://github.com/valerylisay/digitalocean-api).But unfortunately when I attempted to ssh into one of my freshly created droplets it would not connect. The problem is I needed to add the knownhosts file. While I was researching this I found this webpage [Managing Multiple SSH Keys](http://www.robotgoblin.co.uk/blog/2012/07/24/managing-multiple-ssh-keys/). So I decided to make a python script which would generate the required files as described on the webpage. While I was using the script in ipython notebook, kept on getting confused which VPS I was logging into, what images where on the server etc. I happened to have some code kicking around which will dump a python dictionary into an HTML table and displays it in the ipython notebook. So I decided to use that to dump which droplets are active, what images are available etc. To convert the python object into a dictionary I just fed my function the __dict__ property.

Basically I have 3 files:

* ipython_helpers.py - contains functions to display python dictionaries as tables. I just happened to have this kicking around. The file is on my gist here. [ipython_helpers.py](https://gist.github.com/johnpauljanecek/a1c9f61f0609c330c213)
* secret.py - a file which needs to contain three variables.digitalOceanToken,IdRsaPub and idRsa. Set these as appropriate for your digitalocean account.
* common.py - contains common functions, I need to refactor more. If I have to work on other IAAS frameworks besides digitalocean, I will refactor more. [common.py](https://gist.github.com/johnpauljanecek/a91d45cf1ba6a03f5993).
* digitalocean_config.py - the file that I does everything I talked about. [digitalocean_config.py](https://gist.github.com/johnpauljanecek/132bdb2ad4379be38b01)
* and ofcourse last but not least an ipython notebook which gives an example of it in operation.[Ipython demo notebook](https://gist.github.com/johnpauljanecek/1252a4b0c626bbc62410)

Summary
-------

This is just a short quick post on how to make tools which save you time. Before running the code, make sure you backup your .ssh directory. Also it is good to keep old code lying around, the ipython_helpers.py was from another project.

Digitalocean also is great for testing new ideas. [digitalocean](https://www.digitalocean.com/?refcode=da3d5d4ab00c). So I wanted to start using mongodb, since it seemed to be similar to zodb, but I didn't want the hassle of installing it on my system. So I just created a mongodb on digitalocean and within minutes I was able to test and try out mongodb. 
