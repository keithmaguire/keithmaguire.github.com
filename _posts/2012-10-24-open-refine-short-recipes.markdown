---
layout: post
title: "Open Refine short recipes"
---

[Open Refine](http://openrefine.org/) is as useful for faceted browsing as it is for data cleaning. You can get to grips with the structure and character of a large and irregular data set without altering anything. 

- [Handling null values when faceting with multiple columns](#null)
- [Save your start/end point](#startend)
- [Faceting on a particular substring, regardless of blanks or case](#blankcase)
- [Look at the stuff after the first word](#secondword)
- [Isolating comments](#comments)
- [Capitalise the first letter of a string](#firstletter)
- [Faceting by is-it-blank-or-a-particular-word](#blankword)
- [Checking the number of 'words' in a field](#nowords)
- [Compare two columns](#compcol)
- [Compare two columns with different types of data](#compdifcol)

<a name="null"></a>
#Handling null values when faceting with multiple columns

When sticking the values in different columns together Refine can sometimes balk when one of the values is null and return a null for the joined-up string even if there are values in some of the columns.

This will replace any blank or null values with an empty string:
```
forNonBlank(value,v,v "")
```

If you're referring to another column replace `value` with `cells.col_1.value` or `cells["col 1"].value`

```
forNonBlank(cells.col_1.value,v,v "")
```

When joining columns you possibly like to put a space between them to make things clearer - rather than just `colA + colB`,  `colA + " " + colB`

If you use that you'll want to make sure you remove extra spaces from the string at the end. Do this by enclosing the GREL which joins the strings in brackets and at the end put `.trim().replace(/\s+/," ")`

This is the equivalent of doing the join and then running both of the Commonly Used Transformations - "Remove leading or following white spaces" and "remove multiple white spaces"

So if you were working with a dataset where some cells contained Genus information, some Subgenus and some Species - but not all of them in every row - you end up with something along the lines of

{% highlight javascript %}
(
cells.Genus.value
    +" "+
forNonBlank(cells.species.value,v,v "")
    +" "+
forNonBlank(cells.subspecies.value,v,v "")
)
.trim().replace(/\s+/," ")
{% endhighlight %}

That's formatted to hopefully make it a bit easier to read - spaces or new lines don't matter to Refine, so I use them a lot to make what I'm doing clearer to myself.

<a name="startend"></a>
#Save your start/end point

It's easy to miss the "permalink" link just to the right of your project name at the top left of the screen.  This is useful for when you're doing lots of different checks but always starting with the same few transformations.  Just get the facets set up to your starting point and save the permalink. When you next start work just use that rather than opening the project

I'm currently working with a bunch of datasets which I need to bring to a standardised format so they can be transferred into another management system.  Before cleaning or transforming any of the data using lots of faceted browsing is handy for getting a grip on just what you're working with.

There's a few different transformations and facets I need to be doing frequently so I'm recording them here to make an easy reference for myself.

<a name="blankcase"></a>
#Faceting on a particular substring, regardless of blanks or case

Facet on the following and then choose "true" (does contain the string) or "false" (doesn't contain the string or is blank) as appropriate.

```
(forNonBlank(toLowercase(value),v,v,"")).contains("SUBSTRING")
```

<a name="secondword"></a>
#Look at the stuff after the first word

To check to see if there are entries with multiple words, and what kind of entries they are. 

```
value.partition(" ")[2]
```

"value.split" doesn't do what I want here, because it returns an array with as many entries as there are bits separated by spaces. This way if the entry happens to be, say, a whole sentence then you'll see a bunch of words rather than just the first word after the first space.  Then you can facet by **Facet > Customized Facets > Text length facet** and see if there's anything which obviously doesn't belong.

<a name="comments"></a>
#Isolating comments

A common convention, particularly with library cataloging, is that comments added by a compiler or cataloger get added in square brackets to distinguish them from the original data. To isolate out these comments:

```
value.split(/[\[\]]/)[1]
```

<a name="firstletter"></a>
#Capitalise the first letter of a string

... But not convert to Title Case - just capitalise the very first word of the sentence or bunch of words that make up the entry.

```
toUppercase(substring(value,0,1 ))+toLowercase(substring(value,1))
```

<a name="blankword"></a>
#Faceting by is-it-blank-or-a-particular-word

To facet by whether the value is blank or another particular word -- the example I'm working with is where I've a load of values in a particular column which read "none". Equally you might want to ignore "n/a" or something like that.

```
isBlank(value.replace("none",""))
```
Then choose "false"

What it's doing is searching for blanks, but only after replacing any occurrences of "none" with a blank.

<a name="nowords"></a>
#Checking the number of 'words' in a field

When tidying up data it can be useful to be able to see how many lumps of text are in a field. I work with a lot of taxonomic data and I often want to see if the information is consistently structured like **genus-name species-name**.

To split up the value at any spaces and then count the number of "lumps" generated by that split

```
length(value.split(" "))
```

This gives results of 1,2 and 3, meaning the value either had 1, 2 or 3 "words" in it:

1. upon investigation turned out to be items only identified to Genus level
2. were items identified to Genus and Species level
3. upon investigation turned out to be entries in the format *genus-name sp. nov.* (meaning a new, but so far unnamed, species.)

So I can see that the second and third lumps of text should really be considered as a single unit. This helps avoid mistakes like assuming that if there's a third word then it'll be referring to a subspecies.


<a name="compcol"></a>
#Compare two columns

Sometimes when using Refine you're not sure if the column you're checking actually needs to be cleaned up -- it *looks* like it's the same as another one, but you want to be sure:

Take two columns COLUMN-ONE and COLUMN-TWO which look like they probably contain the same information, but it's hard to tell for sure as there's a few tens of thousands of entries in each of them.

In order to compare whether two columns which seem to be identical actually *are*, add a new column based on COLUMN_TWO:

`
value==cells["COLUMN_ONE"].value
`

Or (if you like to type everything out): `cells["COLUMN_ONE"].value==cells["COLUMN_TWO"].value`

and then facet and have a check to see if there are any 'false' results.  

This can give confusing answers if there are blank cells - "null" while typing out the expression or "(blank)" after you've clicked okay and the results are in the little box on the right-hand-side. If that happens use `forNotBlank()` as indicated above.

If you've named your column with a single word you can use the slightly clearer (I always give my columns single-word names because of this):

`
cells.COLUMN_ONE.value==cells.COLUMN_TWO.value
`




<a name="compdifcol"></a>
#Compare two columns with different types of data

If you're not getting useful results when comparing two columns make sure that the format of the data is the same - Refine doesn't think that a _date_ 1984 and a string "1984" are identical.  To compare them add a `toString` where appropriate so you'd end up with something like:

```
cells["COLUMN\_ONE"].value.toString==toString(cells["COLUMN\_TWO"].value)
```

I want to get one column of dates and compare it to the dates in another column called "Authority". 

The entries in Authority look like "Smith, 1985" or "(Smith, 1985)"; the entries in Year look like "1986".

1. Make a new column based on the column of only dates, so the right-hand side of the == will be `toString(value)`
2. Get the date bit of the Authority: `value.partition(" ")[2]`
3. Remove any parentheses that might be in the date bit of Authority: `value.replace(")","")`
which combined makes:

`
toString(cells.Authority.value.partition(" ")[2].replace(")",""))==toString(value)
`

