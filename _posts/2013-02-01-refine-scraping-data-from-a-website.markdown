---
layout: post
title: (Refine) Scraping data from a website
date: "2013-02-01 00:00"
published: true
---

One of the things that Refine can do for you is add information to a data set by looking up information online. Here's a demonstration of one way to do that.

I have a file containing scientific names of animals.

I would like to add the details of the journal article where the species was first described. 

## Sources

The sites I will consult to get the publication information are 

the [Australian Faunal Directory](http://www.environment.gov.au/biodiversity/abrs/online-resources/fauna/afd/home) and 
[biodiversity.org.au](http://biodiversity.org.au).

The AFD provides in-depth taxonomic and bibliographic information about animals from Australia.  [biodiversity.org.au](http://biodiversity.org.au) provides pretty much the same information but will return data in JSON form. It will also return the information for the name you search for even if that name is no longer valid, whereas the AFD will automatically redirect you to the current name.

To demonstrate the process I'm using an extract of dangerous Australian fish species from [Fishbase](http://www.fishbase.org/Country/CountryChecklist.php?showAll=yes&c_code=036&vhabitat=dangerous) with just the Genus and Species listed.

## Get the data

To get the required information out of the site you need to work out how queries should be formed. The Atlas of Living Australia has a page which describes [webservices](http://www.ala.org.au/about-the-atlas/downloadable-tools/web-services/) available for looking up Australian biodiversity data. There you will see a link to the Taxon name service technical information at [Biodiversity.org.au](http://biodiversity.org.au/confluence/display/bdv/NSL+Services). 

Here it instructs you that queries should be constructed in the form `http://biodiversity.org.au/name/The name`

So the URL will need to be constructed from `http://biodiversity.org.au/name/` + Genus + a space + species

So to create a new column which will contain all the appropriate URLs pointing to biodiversity.org.au:

In the Species column choose *Edit column*, then *Add column based on this column...* and enter:

```
"http://biodiversity.org.au/name/"+cells.Genus.value+"%20"+value+".json"
```

`%20` represents a space - it didn't work when the space was there as `" "`

![creating URL column](/images/scrapingafd/Selection_001.png)

Before clicking OK check that you're on the right track by copy and pasting one of the URLs into a new tab and just make sure that it finds the appropriate page.

Then create a new column based on *this* column by adding a column by fetching URLs - just leave the `value` there for the URL.

Change the speed of requests to another value as 5000 milliseconds between requests means completing all the requests will take a very long time. 

Get refine to record the error rather than a blank when it encounters an error, at least at first.

![scraping biodiversity.org.au](/images/scrapingafd/Selection_002.png)

*This may take quite some time - depending on the amount of items you're checking*

WHen it's finished retrieving the data all the retrieved results will be there filling a cell.


## Parse the JSON

![JSON results](/images/scrapingafd/Selection_003.png)

I've found the easiest way to work out the following bit is by copy-and-pasting one of the cells into a text editor and squinting at the JSON to work out the location of the bit of interest.

For example the full scientific name is there under dcterms_title

![full scientific name highlighted](/images/scrapingafd/Selection_004.png)

So to get the scientific name out of each entry:

```
value.parseJson()[0].dcterms_title
```
*with the just-created columns hidden so as to show the results better:*

![creating URL column](/images/scrapingafd/Selection_005.png)

One bit of information you can get out of what you've gotten so far is information about the publication which named the taxon, *if that information has been recorded* So for example to get the publication code out  - (this will be easier to work with as it contains the long versions of journal names)

```
value.parseJson()[0].PublicationRef.jsonHref
```
This only returns results for 83 of the 397 entries but that's still a lot quicker than looking them up by hand!

So if we now make a new column by fetching those URLS...

And then, from that new column we can get all sorts of info by adding a new column based on that column:

* Title: `value.parseJson()[0].Title`
* Journal name: `value.parseJson()[0].ParentPublicationRef.dcterms_title`
* Serial number - (Volume number etc.): `value.parseJson()[0].SerialNumber`
* Pages: `value.parseJson()[0].Pages`

To end up with (for entries where there is bibliographic information):

![bibliographic examples](/images/scrapingafd/Selection_006.png)

And continue in a similar manner for other values.

## Parse HTML instead

*OR* - because it seems to depend on the day as to which is faster - you could alternatively check the afd. This involves parsing HTML rather than JSON:

In this example I'll try and get the list of synonymies from the AFD. If you look at a species' entry on the AFD you'll see there's a link to a "Names List" - that's the page we're after. Working from a random example URL:

'http://www.environment.gov.au/biodiversity/abrs/online-resources/fauna/afd/taxa/'+ Genus + a space + species +'/names'

So to create the URL from the species column:

```
"http://www.environment.gov.au/biodiversity/abrs/online-resources/fauna/afd/taxa/"+cells.Genus.value+"%20"+value+"/names"
```

Again, make sure to use `"%20"` rather than `" "` or you may end up spending a frustrating few hours trying to work out why this isn't working...

This fills each cell that contains a valid name with the souce HTML from all the different individual pages. It looks horrible, but from this you can extract the information you're after. 

![scrape from AFD names](/images/scrapingafd/Selection_007.png)

To do this you need to use `parseHtml()` to specify the particular piece of the web page that contains the information you are after. This can be done by opening one of the target URLs in a browser and using Firefox's "Inspect Element" menu to identify just which bit of the page contains the information which is of interest.

## scientific name

###

However in this case if we want to get the scientific name we just need to examine the structure of the page to see where the scientific name is:

![working out location of scientific name](/images/scrapingafd/Selection_008.png)

You can see that the scientific name is in a div with class `.afdbody`, and it's the `<h1>` element. Then if we get rid of "Names List for" and anything between `<`and`>` then we'll have the scientific name:

So we add a new column based on the messy HTML-filled column:

```
value.parseHtml().select(".afdbody")[0].select("h1")[0].replace("Names List for","").replace(/\<.\w+\>/,"")
```

(I find it makes expressions like that much easier to understand if you type them out slowly and watch the results change.)

## synonymies

Then to get all the different names which have been given to that species, again add a column based on the HTML-filled column:

```
value.parseHtml().select(".afdbody")[0].select(".noindent")[0].select("li")[0].replace(/\<[\w\s\/]+\>/,"").partition(">")[2].unescape("html")
```

(As you can see the biodiversity/JSON option is *far* more convenient!). 

And you end up with

![synonymies added](/images/scrapingafd/Selection_009.png)