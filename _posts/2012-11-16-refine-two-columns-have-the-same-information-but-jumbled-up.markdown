---
layout: post
title: "(Refine) Two columns have the same information but jumbled up"
date: 2012-11-16 21:05
categories: refine
---

This is to illustrate one way in which you can approach faceting data.  This exact example is possibly of very limited use for anything apart from the single time I needed it but it's useful for walking through the thought process where a number of simple steps get combined to make an apparently complicated expression.

#Setup

What happened is that I've already tidied up a massive list of names so that it all nice and tidy: LASTNAME FIRSTINITIAL. 

Now I've found another list of names which I am *almost* certain contains exactly the same names. If the values *are* the same then the second column can just be forgotten about. 

But this second list has the initials removed half the time and the names sometimes in different orders, so simply filtering by `value of column one == value of column two` won't work.

Look at these lines, they're small enough that just scanning it tells you that the surnames are the same - but you can see how if there were tens of thousands of entries it would be very time consuming.

![showing the columns](/images/foreachfingerprint/Selection_004.png )



None of the following are transformations applied to the values - these are all Custom Facets. At this stage we're not trying to change anything, just checking to see if they're the same. So we start by getting a `fingerprint()` of the surnames

#Fingerprint() of surnames

First I replaced anything that isn't a letter with a space `value.replace("."," ")`, for any occasions where there were two initials with a period between them. 

Then I got rid of any initials by using `split(" ")` to separate the values at each space:

![replace periods and then split at spaces](/images/foreachfingerprint/Selection_001.png )

I was using this *just* in order to make sure that double initials get separated out and don't get stuck together and end up mistaken for a very short surname. You can see that the ampersand `&` is still there. If you wanted to get rid of everything that wasn't a letter (or number) you could do `value.replace(/\W/," ")`

Then, each of these separated out values which was only a single character long was replaced with nothing `""`.

This is done using `forEach()` which works like: 

First of all describe an array: (The array constructed before `value.replace("."," ").split(" ")`)

Then allocate a variable: This can just be some random letter, let's say `v`

Then say what to do to each element of the array: `ForEach()` goes through the array and for each element of it does whatever you describe next. If you want to manipulate the value of the element you use the letter to represent it.

To get rid of any elements which are just a single character long:  `if(length(v)==1,"",v)` 

![replace each element that's only a single character long with nothing](/images/foreachfingerprint/Selection_002.png)

To reattach the remaining bits back together with a space, `join(" ")`,  then `fingerprint()` to get a simple fingerprint to compare the two values while ignoring punctuation. 

![stick back together and get fingerprint](/images/foreachfingerprint/Selection_003.png )

This gets a fingerprint of the surnames in _one_ of the columns, now we just need to do the same for the other and compare them.

#Compare the columns

So if the same is done to the second column then the result `true` will be given for entries where the same surnames occur in both columns.

The two columns are called Name1 and Name2. So creating a custom facet or a new column based on Name2 with the following will return "true" for when the surnames are the same in both columns.

{% highlight javascript%}
forEach(
	value.replace("."," ").split(" "),v,if(length(v)==1,"",v)
	).join(" ")
	.fingerprint()
==
forEach(
	cells.Name1.value.replace("."," ").split(" "),v,if(length(v)==1,"",v)
	).join(" ")
	.fingerprint()
{% endhighlight %}

(It doesn't have to be on different lines, but it doesn't mess it up and I find that easier to read)
Both of those lines are the mostly the same except that `value` in the first is replaced with `cells.Name2.value`

![new column with true or false](/images/foreachfingerprint/Selection_005.png )

To illustrate what happens when they're not the same:

![new column with true or false with false entry](/images/foreachfingerprint/Selection_006.png )



