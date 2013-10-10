---
layout: post
title: (Refine) a worked example
date: "2012-08-13 20:08"
comments: true
categories: refine
published: true
---

[Google Refine](http://code.google.com/p/google-refine/) is a great tool for working with awkward lumps of irregularly formatted data. I'll try and do the occasional post where I'll show how I've used it to address a particular issue with a dataset.

The first example I'll go through is getting a date out of a particular dataset when I couldn't be certain which field it was going to be in.

The difficulty arose from an inevitable result of working with datasets compiled over a long time. I was working with a data extraction from a library catalogue which contained entries created over the course of more than a century. As a result there were inconsistencies in the ways in the ways in which information was contained in the entries. 

##The problem

The difficulty this presented for me was like this - the date that an item was published would not necessarily be in the expected place. In the example I was using the columns of interest were those derived from the MARC code field DATE where the publication date would be expected to occur, IMPRINT, where the imprint information would be expected, and there was the off-chance that the TITLE could contain the date of publication. 

So, instead of 

![text arranged in the ideal way](/images/workexample/ideal_data.PNG)


<p>There were many entries where the entries would look like</p>

![messy or normal data](/images/workexample/messy_data.PNG)


It's worth mentioning that every positive result was going to be hand-checked as part of the larger process, so I was more concerned that I might miss items than that I might return too many items. In the event there were only maybe four false results out of about three and a half thousand positives.

##Make a new column 

...which contains the text of all the cells which might have the date in them stuck together in order of reliability"

Starting off with a dump from a library catalogue of all the items look for the columns most likely to have had the date in them.  Over years items may have ended up with the date in the publication date "PUB\_DATES_COMBINED", or the imprint information "IMPRINT" or at the last guess maybe the title "TITLE" mentions the year.

The thinking behind this step is to look at the publishing date and record that.  If it's not there then check in the imprint date and record the last date (If a serial this catches the last year in the range).  If it's not there then check just in case it's mentioned in the title.  We'll do this by just sticking the text in each field together and then working from right to left and recording the first occurrence of something that looks like a date.

To stick the text in the various columns together click on the drop-down menu for the left-most column of interest and choose 
**Edit column >> Add column** based on this column

```
cells["TITLE"].value + " " + cells["IMPRINT"].value + " " + cells["PUB_DATES_COMBINED"].value
```

##Tidy up the text 

...in this new column so that all dates are reliably in the same format:

Over the years various ways have been used to indicate that a portion of a date is unknown.  An unknown year in the 1940s could appear as 194?, 194-, 194U, or 194u.  We want to change this so that they are consistent.  What we are going to change them to is 194\_ because having an underscore there is useful for one of the later steps - and, as we're just looking for items published before 1950 we don't care in which year of the 1940s an individual item was published in.

We do this through repeated transformations on the "Combined columns" column.  Remember that all we care about is the dates so it doesn't matter how much this messes up the rest of the text in the column.

Firstly change everything to upper case:
**Edit cells >> Common transforms >> To uppercase**



Now change all question marks to an underscore:

**Edit cells >> Transform**

and then enter
```
value.replace("?","_")
```

Now do that again to change all occurrance of the letter "U" to an underscore:

```
value.replace("U","_")
```

Finally, change all occurrance of "-" to an underscore:

```
value.replace("-","_")
```

(You could probable do all that in one lump with ANDs between them)

##Make a new column

...which goes through the previously created column and takes out everything apart from the last occurance of a date:

Add a new column based on the "Combined columns" column.  In the "Expression" box for creating the new column enter:
```
value.rpartition(/\d\d\w\w/)[1]
```
and give the Column an appropriate name and click OK

What this is doing is:

- 'value' - take the contents of the cell
- 'rpartition' - split the contents of the cell into three (based on what is in the brackets).  Numbered 0,1 and 2: 0, the bits before the last occurance of a date; 1, the date; 2, the bits after the date
- '/\d\d\w\w/' - this is a regular expression meaning "find a bit of the text in the form digit,digit,element of a word,element of a word" (an 'element of a word' is any letter, digit or an underscore - this is why the previous steps changed )
- '[1]' - this specifies that we are only interested in the _second_ bit, i.e. the date

Again as you type you'll see the bits that are going to be the contents of the new cells appearing - this is good for making sure that you're getting it right.

##Check for absent or unusual dates.

Have a look at an remaining columns which still don't have a date in the new "Dates" column by for that column:
**Facet >> Customized facets >> Facet by blank**

And have a look to see if there are any entries where you can spot the date

Have a look at all the different dates which now occur in the document.  

**Facet >> Text facets**

In the sidebar will now appear a small window with all the different dates in the column, if you scroll down the bottom you'll see some ones which don't fit

Click on the ones you want to check.  If it's only a small number that need fixing, like "991_" in the example above you can just edit the cell.  If it's a larger number you can do a transform.

The entries marked 9999 will be serials, look at them as for some of them the date will be evident from the title or imprint fields.  Again you can just edit them as necessary.

If you are preparing the entries because you want to get all the entries before a certain date you might want to change all the unknown digits "_" to a "9" as that's the highest it could be and will make things easier to sort in a spreadsheet.

Then export to whatever format you prefer