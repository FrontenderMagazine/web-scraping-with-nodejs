<article class="post-216707 post type-post status-publish format-standard has-
post-thumbnail hentry category-coding tag-javascript tag-node-js
">
Web scraping is the process of programmatically retrieving information from the
Internet. As the volume of data on the web has increased, this practice has 
become increasingly widespread, and a number of powerful services have emerged 
to simplify it. Unfortunately, the majority of them are costly, limited or have 
other disadvantages. Instead of turning to one of these third-party resources,
**you can use Node.js to create a powerful web scraper** that is both extremely
versatile and completely free.

In this article, I’ll be covering the following:

*   two Node.js modules, Request and Cheerio, that simplify web scraping;
*   an introductory application that fetches and displays some sample data;
*   a more advanced application that finds keywords related to Google searches
    .

Also, a few things worth noting before we go on: **A basic understanding of
Node.js is recommended for this article**; so, if you haven’t already, 
[check it out][1][1][2] before continuing. Also, web scraping may violate the
terms of service for some websites, so just make sure you’re in the clear there 
before doing any heavy scraping.

### Modules

To bring in the Node.js modules I mentioned earlier, we’ll be using [NPM][3]
[2][4], the Node Package Manager (if you’ve heard of Bower, it’s like that
— except, you use NPM to install Bower). NPM is a package management utility 
that is automatically installed alongside Node.js to make the process of using 
modules as painless as possible. By default, NPM installs the modules in a 
folder named`node_modules` in the directory where you invoke it, so make sure
to call it in your project folder.

And without further ado, here are the modules we’ll be using.

#### Request

While Node.js does provide simple methods of downloading data from the Internet
via HTTP and HTTPS interfaces, you have to handle them separately, to say 
nothing of redirects and other issues that appear when you start working with 
web scraping. The[Request module][5][3][6] merges these methods, abstracts away
the difficulties and presents you with a single unified interface for making 
requests. We’ll use this module to download web pages directly into memory. To 
install it, run`npm install request` from your terminal in the directory where
your main Node.js file will be located.

#### Cheerio

[Cheerio][7][4][8] enables you to work with downloaded web data using the same
syntax that jQuery employs. To quote the copy on its home page, “Cheerio is a 
fast, flexible and lean implementation of jQuery designed specifically for the 
server.” Bringing in Cheerio enables us to focus on the data we download 
directly, rather than on parsing it. To install it, run`npm install cheerio`
from your terminal in the directory where your main Node.js file will be located.

### Implementation

The code below is a quick little application to nab the temperature from a
weather website. I popped in my area code at the end of the URL we’re 
downloading, but if you want to try it out, you can put yours in there (just 
make sure to install the two modules we’re attempting to require first; you can 
learn how to do that via the links given for them above
).

    var request = require("request"),
    	cheerio = require("cheerio"),
    	url = "http://www.wunderground.com/cgi-bin/findweather/getForecast?&query=" + 02888;
    	
    request(url, function (error, response, body) {
    	if (!error) {
    		var $ = cheerio.load(body),
    			temperature = $("[data-variable='temperature'] .wx-value").html();
    			
    		console.log("It’s " + temperature + " degrees Fahrenheit.");
    	} else {
    		console.log("We’ve encountered an error: " + error);
    	}
    });
    

So, what are we doing here? First, we’re requiring our modules so that we can
access them later on. Then, we’re defining the URL we want to download in a 
variable.

Then, we use the Request module to download the page at the URL specified above
via the`request` function. We pass in the URL that we want to download and a
callback that will handle the results of our request. When that data is returned,
that callback is invoked and passed three variables:`error`, `response` and 
`body`. If Request encounters a problem downloading the web page and can’t
retrieve the data, it will pass a valid error object to the function, and the 
body variable will be null. Before we begin working with our data, we’ll check 
that there aren’t any errors; if there are, we’ll just log them so we can see 
what went wrong.

If all is well, we pass our data off to Cheerio. Then, we’ll be able to
handle the data like we would any other web page, using standard jQuery syntax. 
To find the data we want, we’ll have to build a selector that grabs the element(
s) we’re interested in from the page. If you navigate to the URL I’ve used for 
this example in your browser and start exploring the page with developer tools, 
you’ll notice that the big green temperature element is the one I’ve constructed
a selector for. Finally, now that we’ve got ahold of our element, it’s a simple 
matter of grabbing that data and logging it to the console.

We can take it plenty of places from here. I encourage you to play around, and
I’ve summarized the key steps for you below. They are as follows.

#### In Your Browser

1.  Visit the page you want to scrape in your browser, being sure to record its
    URL.
   
2.  Find the element(s) you want data from, and figure out a jQuery selector
    for them.
   

#### In Your Code

1.  Use request to download the page at your URL.
2.  Pass the returned data into Cheerio so you can get your jQuery-like
    interface.
   
3.  Use the selector you wrote earlier to scrape your data from the page.

### Going Further: Data Mining

More advanced uses of web scraping can often be categorized as [data mining][9]
[5][10], the process of downloading a lot of web pages and generating reports
based on the data extracted from them. Node.js scales well for applications of 
this nature.

I’ve written a small data-mining app in Node.js, less than a hundred lines,
to show how we’d use the two libraries that I mentioned above in a more 
complicated implementation. The app finds the most popular terms associated with
a specific Google search by analyzing the text of each of the pages linked to on
the first page of Google results.

There are three main phases in this app:

1.  Examine the Google search.
2.  Download all of the pages and parse out all the text on each page.
3.  Analyze the text and present the most popular words.

We’ll take a quick look at the code that’s required to make each of these
things happen — as you might guess, not a lot.

#### Downloading the Google Search

The first thing we’ll need to do is find out which pages we’re going to
analyze. Because we’re going to be looking at pages pulled from a Google search,
we simply find the URL for the search we want, download it and parse the results
to find the URLs we need.

To download the page we use Request, like in the example above, and to parse it
we’ll use Cheerio again. Here’s what the code looks like:

    request(url, function (error, response, body) {
    	if (error) {
    		console.log(“Couldn’t get page because of error: “ + error);
    		return;
    	}
    	
    	// load the body of the page into Cheerio so we can traverse the DOM
    	var $ = cheerio.load(body),
    		links = $(".r a");
    		
    	links.each(function (i, link) {
    		// get the href attribute of each link
    		var url = $(link).attr("href");
    		
    		// strip out unnecessary junk
    		url = url.replace("/url?q=", "").split("&")[0];
    		
    		if (url.charAt(0) === "/") {
    			return;
    		}
    		
    		// this link counts as a result, so increment results
    		totalResults++;
    

In this case, the URL variable we’re passing in is a Google search for the
term “data mining.
”

As you can see, we first make a request to get the contents of the page. Then,
we load the contents of the page into Cheerio so that we can query the DOM for 
the elements that hold the links to the pertinent results. Then, we loop through
the links and strip out some extra URL parameters that Google inserts for its 
own usage — when we’re downloading the pages with the Request module, we don’t 
want any of those extra parameters.

Finally, once we’ve done all that, we make sure the URL doesn’t start with a 
`/` — if so, it’s an internal link to something else of Google’s, and we
don’t want to try to download it, because either the URL is malformed for our 
purposes or, even if it isn’t malformed, it wouldn’t be relevant.

#### Pulling the Words From Each Page

Now that we have the URLs of our pages, we need to pull the words from each
page. This step consists of doing much the same thing we did just above — only, 
in this case, the URL variable refers to the URL of the page that we found and 
processed in the loop above.

    request(url, function (error, response, body) {
    	// load the page into Cheerio
    	var $page = cheerio.load(body),
    		text = $page("body").text();
    

Again, we use Request and Cheerio to download the page and get access to its
DOM. Here, we use that access to get just the text from the page.

Next, we’ll need to clean up the text from the page — it’ll have all
sorts of garbage that we don’t want on it, like a lot of extra white space, 
styling, occasionally even the odd bit of JSON data. This is what we’ll need to 
do:

1.  Compress all white space to single spaces.
2.  Throw away any characters that aren’t letters or spaces.
3.  Convert everything to lowercase.

Once we’ve done that, we can simply split our text on the spaces, and we’re
left with an array that contains all of the rendered words on the page. We can 
then loop through them and add them to our corpus.

The code to do all that looks like this:

    // Throw away extra white space and non-alphanumeric characters.
    text = text.replace(/\s+/g, " ")
    	     .replace(/[^a-zA-Z ]/g, "")
    	     .toLowerCase();
    
    // Split on spaces for a list of all the words on that page and 
    // loop through that list.
    text.split(" ").forEach(function (word) {
    	// We don't want to include very short or long words because they're 
    	// probably bad data.
    	if (word.length  20) {
    		return;
    	}
    				
    	if (corpus[word]) {
    		// If this word is already in our corpus, our collection
    		// of terms, increase the count for appearances of that 
    		// word by one.
    		corpus[word]++;
    	} else {
    		// Otherwise, say that we've found one of that word so far.
    		corpus[word] = 1;
    	}
    });
    

#### Analyzing Our Words

Once we’ve got all of our words in our corpus, we can loop through that and
sort them by popularity. First, we’ll need to stick them in an array, though, 
because the corpus is an object.

    // stick all words in an array
    for (prop in corpus) {
    	words.push({
    		word: prop,
    		count: corpus[prop]
    	});
    }
    	
    // sort array based on how often they occur
    words.sort(function (a, b) {
    	return b.count - a.count;
    });
    

The result will be a sorted array representing exactly how often each word in
it has been used on all of the websites from the first page of results of the 
Google search. Below is a sample set of results for the term “data mining.” (
Coincidentally, I used this list to generate the word cloud at the top of this 
article.
)

    [ { word: 'data', count: 981 },
      { word: 'mining', count: 531 },
      { word: 'that', count: 187 },
      { word: 'analysis', count: 120 },
      { word: 'information', count: 113 },
      { word: 'from', count: 102 },
      { word: 'this', count: 97 },
      { word: 'with', count: 92 },
      { word: 'software', count: 81 },
      { word: 'knowledge', count: 79 },
      { word: 'used', count: 78 },
      { word: 'patterns', count: 72 },
      { word: 'learning', count: 70 },
      { word: 'example', count: 70 },
      { word: 'which', count: 69 },
      { word: 'more', count: 68 },
      { word: 'discovery', count: 67 },
      { word: 'such', count: 67 },
      { word: 'techniques', count: 66 },
      { word: 'process', count: 59 } ]
    

If you’re interested in seeing the rest of the code, check out the 
[fully commented source][11][6][12].

A good exercise going forward would be to take this application to the next
level. You could optimize the text parsing, extend the search to multiple pages 
of Google results, even strip out common words that aren’t really key terms (
like “that” and “from”). More bug handling could also be added to make the app 
even more robust — when you’re mining data, you want as many layers of 
redundancy as you can reasonably afford. The variety of content that you’ll be 
pulling in is such that inevitably you’ll come across an unexpected piece of 
text that, if unhandled, would throw an error and promptly crash your 
application.

### In Conclusion

As always, if you find anything related to web scraping with Node.js that you
think is helpful or just have questions or thoughts you want to share, be sure 
to let us know via the comments below. Also, follow me[on Twitter][13][7][14]
and check out[my blog][15][8][16] for more on Node.js, web scraping and
JavaScript in general.

*(il, rb, al)*

#### Footnotes

[↑ Back to top][17] [Tweet it][18][Share on Facebook][19]</article>

 [1]: http://nodejs.org/
 [2]: http://www.smashingmagazine.com/2015/04/08/web-scraping-with-nodejs/#1
 [3]: https://www.npmjs.com/
 [4]: http://www.smashingmagazine.com/2015/04/08/web-scraping-with-nodejs/#2
 [5]: https://github.com/request/request
 [6]: http://www.smashingmagazine.com/2015/04/08/web-scraping-with-nodejs/#3
 [7]: https://github.com/cheeriojs/cheerio
 [8]: http://www.smashingmagazine.com/2015/04/08/web-scraping-with-nodejs/#4
 [9]: http://en.wikipedia.org/wiki/Data_mining
 [10]: http://www.smashingmagazine.com/2015/04/08/web-scraping-with-nodejs/#5
 [11]: https://gist.github.com/elliotbonneville/1bf694b8c83f358e0404
 [12]: http://www.smashingmagazine.com/2015/04/08/web-scraping-with-nodejs/#6
 [13]: https://twitter.com/bovenille
 [14]: http://www.smashingmagazine.com/2015/04/08/web-scraping-with-nodejs/#7
 [15]: http://heyjavascript.com
 [16]: http://www.smashingmagazine.com/2015/04/08/web-scraping-with-nodejs/#8
 [17]: http://www.smashingmagazine.com/2015/04/08/web-scraping-with-nodejs/#top

 [18]: https://twitter.com/intent/tweet?original_referer=http%3A%2F%2Fwww.smashingmagazine.com%2F2015%2F04%2F08%2Fweb-scraping-with-nodejs%2F&source=tweetbutton&text=Web%20Scraping%20With%20Node.js&url=http%3A%2F%2Fwww.smashingmagazine.com%2F2015%2F04%2F08%2Fweb-scraping-with-nodejs%2F&via=smashingmag

 [19]: http://www.facebook.com/sharer/sharer.php?u=http%3A%2F%2Fwww.smashingmagazine.com%2F2015%2F04%2F08%2Fweb-scraping-with-nodejs%2F