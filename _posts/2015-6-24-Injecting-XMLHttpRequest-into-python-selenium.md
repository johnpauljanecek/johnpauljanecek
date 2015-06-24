---
layout: post
title: Injecting XMLHttpRequests into python selenium.
---

When scraping websites using a headless browser, if it is possible to call the XMLHttpRequest call using [Selenium Requests](https://github.com/cryzed/Selenium-Requests) which is an extension of [Selenium-Requests](http://docs.python-requests.org/en/latest/). The Selenium Requests Library works by creating a small webserver, spawning another selenium window and copying all of the browser cookies. The solution is ingenious, and making calls with the requests library makes things a lot easier.

These are the possible pitfalls when using Selenium Requests

*  When selenium requests spawns the extra selenium windows to get the cookies this takes time.
* When doing repeated XMLHttpRequest calls, it does not seem to update the cookies, in certain cases it does not work.
* On rare occasions it can crash the headless browser.

Selenium requests is a great addon to to python selenium, and I use it frequently. It is in the cases when Selenium Requests does not work that I am talking about.

Recently a client asked me to scrape results from kickstarter. In this case he wanted all [kickstarter projects which are card games in the USA](http://nbviewer.ipython.org/gist/johnpauljanecek/3446d11bed47b3b12b27). I was able to locate the XMLHttpRequest, but is sets an anti-xss token on one of the headers. I located where the token is in the webpage and extracted it, but when I set the headers, the request would fail. But when I called the request from within firebug, it worked fine. Inclued in this post is a ipython notebook which explains the technique I used to solve the problem. [IPython notebook demonstrating the technique](http://nbviewer.ipython.org/gist/johnpauljanecek/3446d11bed47b3b12b27)

The important function in example is in SearchPage.doajaxresultsrequest with this piece of javascript code.

{% highlight javascript linenos=table %}
        var url = arguments[0];
        window._jsonResult = null;
        var token = document.querySelector('meta[name = "csrf-token"]').getAttribute("content");
        
        var xmlhttp = new XMLHttpRequest();
    
        xmlhttp.onreadystatechange = function() {
        if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
            window._jsonResult = jsonResult = JSON.parse(xmlhttp.responseText);
            }}

        xmlhttp.open("GET", url, true);
        xmlhttp.setRequestHeader("X-CSRF-Token",token);
        xmlhttp.setRequestHeader("X-Requested-With","XMLHttpRequest")
        xmlhttp.setRequestHeader("Accept","application/json, text/javascript, */*; q=0.01")
        xmlhttp.send();

        return true;;
{% endhighlight %}

The url is pulled off the arguments array, the xss token is extracted from the document,and then the XMLHTTPRequest call is setup and called. I decided not to wait for the XMLHTTPRequest call to complete, but instead store the result in window._jsonResult.

A second call is then made with  getajaxresult to return the result. In this case the XMLHttpRequest is parsed into json before it is stored, that means when the result is fetched by python there is no need to parse it with JSON.

The first example uses the browser class of my [docker_rpyc module](http://johnpauljanecek.github.io/controlling-docker-containers-with-python-rpyc/) . The second example uses the worker class of my rpyc_docker module which means the browser is running totally headless and isolated within a docker container. Since my library uses rpyc the code is almost identical, and the same SearchPage class in the same namespace can be used. It is also possible to run multiple isolated headless browsers.

#Possible improvements on the technique.

* Instead of making two calls to get the result, wait till the XMLHttpRequest finishes before returning, I am not really sure if this is an improvement.
* Have the XMLHttpRequest call save the status of the request instead of just saving the result.
* Have an array which stores XMLHttpRequests, and then repeatedly call the request function. Since javascript is able to do asycronous requests this would probably be a lot quicker for multiple XMLHttpRequests.
* A similar technique can be used to hook HMLHttpRequests, and then intercepting the results.

#Why this matters ?

In the past scraping web pages could be done with just curl and raw http requests. But websites have transformed from a series of static pages, into applications running inside a web browser. As a result new web scraping techniques need to be developed. From my experience the hybrid method of web scraping websites is highly effective. So a combination of raw http calls, running headless browsers and injecting javacripts.
