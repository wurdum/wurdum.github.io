---
layout: page
title: "About"
date: 2015-07-12 13:34
comments: false
sharing: false
footer: true
---
By default, ELMAH is configured to disallow remote access to the error logs - only a local user can get to elmah.axd. If you take care of the security implications it can be very useful to enable remote access and monitor the logs on your production servers.

We chose to use an XML file for each error but ELMAH is entirely pluggable. There are dozens of alternatives for persisting the error log including Sql Server, an RSS feeds, to Twitter, even to an iPhone app. There are even third party sites such as elmah.io who will host your error logs for you.

One of the advantages of using XML files is that the files can be copied to another machine. If you look in `MainDemo.Web\App_Data\Elmah.Errors`, you will find the resulting xml files.By default, ELMAH is configured to disallow remote access to the error logs - only a local user can get to elmah.axd. If you take care of the security implications it can be very useful to enable remote access and monitor the logs on your production [servers](http://http://en.wikipedia.org/wiki/File:Great_Pyramid_Diagram.svg).

We chose to use an XML file for each error but ELMAH is entirely pluggable. There are dozens of alternatives for persisting the error log including Sql Server, an RSS feeds, to Twitter, even to an iPhone app. There are even third party sites such as elmah.io who will host your error logs for you.