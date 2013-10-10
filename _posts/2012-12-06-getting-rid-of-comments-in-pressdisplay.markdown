---
layout: post
title: "Getting rid of comments in Pressdisplay"
---

Pressdisplay is a fantastic website which your library may provide access to.  Using it you can read newspapers from around the world in newspaper form as opposed to website form.  It's great.

But -- like many things on the internet nowadays they feel compelled to make it social or something by allowing people to add comments to articles. But this ignores the perfectly functional letters page that's been doing a good job for a century or so.

To get rid of them 

First navigate to Pressdisplay using the route through your library.  

Now copy the URL of the homepage minus the `http://` bit

Add the following rules to [Adblock Plus](http://adblockplus.org), replacing PRESSDISPLAYURL with whatever the URL is for the pressdisplay homepage via your library

```
PRESSDISPLAYURL##span.lbl-comment
PRESSDISPLAYURL##div.comments-content
PRESSDISPLAYURL##div.marker-comment
```
Now the comments - and any indication that they were ever there - should be gone! Hooray!
