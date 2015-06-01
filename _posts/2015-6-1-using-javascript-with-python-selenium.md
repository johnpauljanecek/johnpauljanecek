---
layout: post
title: Injecting javascript into python selenium to increase scraping speed
---

When python selenium communicates with the webbrowser it sends its requests through a bridge. For locating a single element in a page and getting its data or clicking on it, this will not be much of a problem. But if you are scrapping a 100+ results from a page this can take a long delay. Sometimes upto several seconds.

The solution is to parse the data that is needed inside the browser using javascript, and then to pass the results back to python selenium using the selenium function [execute_script](http://selenium-python.readthedocs.org/en/latest/api.html?highlight=execute_script#selenium.webdriver.remote.webdriver.WebDriver.execute_script)

The function execute_script if passed arguments will attempt to convert them from python to javascript, when returning the arguments the reverse will happen.

Here is an example where two arguments are passed to the function from python. 

{% highlight python linenos=table %}
#driver is a webdriver instance

js_function = """
return (function(a,b) {
return a + b;
})(arguments[0],arguments[1]);
"""

driver.execute_script(js_function,1,2)
{% endhighlight %}

The example I have used is scrapping the results from duckduckgo. I created a simple example here in an ipython notebook here [Inject javascript into python selenium](http://nbviewer.ipython.org/gist/johnpauljanecek/39c1ab450f3d188af548).

The class DuckDuckGoResults takes a driver as an argument. Here is a brief description of the member functions.

* search(self,searchTerm) - takes a search term and inputs it into duckduckgo, should return when the search page results are there.
* scroll_botton(self) - utility function to scroll the page to the bottom. Used to load all the results from duckduckgo.
* load_all_results(self) - keeps on scrolling the page down until all of the results have been loaded.
* parse_resultElm(self,resultElm) - utility function for parsing a result div. Extracts the title,href and description from the div and returns it as dict.
* def get_results_python(self) - returns all of the results of the page as an array of dicts. This function is written using python. This function is extremely slow.
* def get_results_javascript(self): same as get_results_python except it uses javascript this function returns the results almost instantly.

The get_results_javascript function is an example of extracting results using javascript inside the browser and then passing them back to python selenium. Here is the javascript part of the function.

{% highlight javascript linenos=table %}
var resultElms = Array.prototype.slice.call(document.querySelectorAll("#links>div.results_links_deep"));
return resultElms.map(function(resultElm) {
    var result = [];
    var resultAElm = resultElm.querySelector("a.result__a");
    result.push(["title",resultAElm.textContent]);
    result.push(["href",resultAElm.getAttribute("href")]);
    result.push(["snippet",resultElm.querySelector("div.result__snippet").textContent]);
    return result;
    });
{% endhighlight %}

There is one problem when returning dicts from javascript. Javascript dicts and objects are the same data type. An object returning from javascript into python will not convert properly. Otherwise there is no problem returning all basing datatypes including arrays. The solution is to return the dictionary structure as an array of arrays like I have done. So each item in the array will be [["title","XXXXX"],["href","XXXXX"],[snippet,"XXXX]]. Once the structure has been returned to python convert it into a dict like this.

{% highlight python linenos=table %}
results = self.driver.execute_script(jsFunction)
return map(dict,results)
{% endhighlight %}

The map function applies dict to each member to convert it into an array of dicts so the output is the same as the python function.

For most of my web scrapping I use the firebug extension to assist me. Within firebug run the javascript code, and you can test it inside firebug.

This is an introduction to running javascript code from selenium. The big advantage is speed, and that javascript was designed to manipulate DOM structures.


