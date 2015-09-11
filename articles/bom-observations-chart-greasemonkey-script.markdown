---
layout: article
title: "BOM observations chart Greasemonkey script"
article: true
categories: [web_tweaks]
date: 2013-02-06 20:26
---
Do you live in Australia?? 

Do you like to panic about the heat and/or humidity? 

Do you constantly refresh the BOMs pages to see if the cold change has kicked in?

Allow me to present you with an exciting Greasemonkey script which adds charts to the BOMs observations pages for your city and (in most cases) also sticks the weather forecast in there. You never need look at any other page for your weather. Well apart from the radar obviously, but that goes without saying.

<!--more-->

What this will do for you is instead of 

![observation page before script](/images/greasemonkey/beforescript.png)

You now get:

![observation page after script](/images/greasemonkey/afterscript.png)

* Red line = temperature
* Blue line = humidity
* Dashed Red Line = apparent temperature

It also puts the temperature and humidity into the page title so if you're on a different tab when it updates you can still see what's going on.

To install it you'll need to be using Firefox and have [Greasemonkey](https://addons.mozilla.org/en-US/firefox/addon/greasemonkey/) installed (Or be using Chrome and have Tampermonkey installed)

To install the script click [here](/downloads/code/bom-chart-json.user.js)

At this point Greasemonkey will kick up a warning - you can click Install to install the script or "Show Script Source" if you want to have a look through it ('though if you do want to look at the script it's possible easier to look [here](https://github.com/keithmaguire/bom-chart-userscript))

Now go to the BOM observations page for your location, such as for [Adelaide](http://www.bom.gov.au/products/IDS60901/IDS60901.94675.shtml). The script will always generate a chart but won't get the forecast for *every* location - it's to do with how predictable the URLS for the forecasts are.

If you hover over the temperature or humidity line it'll pop up the value for that point in the chart. I've not included that function for the apparent temperature because I personally don't really believe in the apparent temperature and I only added it to take away the sense of panic when the temperature gets over 40 degrees; and also - seriously, just scroll down.

I hope you find it useful!
