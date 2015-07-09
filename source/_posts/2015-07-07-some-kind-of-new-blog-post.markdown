---
layout: post
title: "Some kind of new blog post"
date: 2015-07-07 15:34:44 +0300
comments: true
categories: test-post
---

By default, ELMAH is configured to disallow remote access to the error logs - only a local user can get to elmah.axd. If you take care of the security implications it can be very useful to enable remote access and monitor the logs on your production servers.

We chose to use an XML file for each error but ELMAH is entirely pluggable. There are dozens of alternatives for persisting the error log including Sql Server, an RSS feeds, to Twitter, even to an iPhone app. There are even third party sites such as elmah.io who will host your error logs for you.

One of the advantages of using XML files is that the files can be copied to another machine. If you look in `MainDemo.Web\App_Data\Elmah.Errors`, you will find the resulting xml files.By default, ELMAH is configured to disallow remote access to the error logs - only a local user can get to elmah.axd. If you take care of the security implications it can be very useful to enable remote access and monitor the logs on your production [servers](http://http://en.wikipedia.org/wiki/File:Great_Pyramid_Diagram.svg).

We chose to use an XML file for each error but ELMAH is entirely pluggable. There are dozens of alternatives for persisting the error log including Sql Server, an RSS feeds, to Twitter, even to an iPhone app. There are even third party sites such as elmah.io who will host your error logs for you.

{% img https://raw.githubusercontent.com/ZeroSharp/zerosharp.github.com/master/images/blog/an-afternoon-with-two-fields-medallists-002.jpg %}

One of the advantages of using XML files is that the files can be copied to another machine. If you look in MainDemo.Web\App_Data\Elmah.Errors, you will find the resulting xml files.

``` c#
[TestFixtureSetUp]
public void CreateTestEnvironment() {
    _surveyDir = "TestSurvey" + TestUtils.GetPostfix();
    _conf = CreateConfForPackage(_surveyDir);

    TestUtils.PreparePackageFor(DeployMode.Install, _conf.PackagePaths.Survey);
    Directory.CreateDirectory(_conf.CommonPaths.Surveys);
    Directory.CreateDirectory(_conf.CommonPaths.Backups);
}
```

By default, ELMAH is configured to disallow remote access to the error logs - only a local user can get to elmah.axd. If you take care of the security implications it can be very useful to enable remote access and monitor the logs on your production servers.

We chose to use an XML file for each error but ELMAH is entirely pluggable. There are dozens of alternatives for persisting the error log including Sql Server, an RSS feeds, to Twitter, even to an iPhone app. There are even third party sites such as elmah.io who will host your error logs for you.

One of the advantages of using XML files is that the files can be copied to another machine. If you look in MainDemo.Web\App_Data\Elmah.Errors, you will find the resulting xml files.By default, ELMAH is configured to disallow remote access to the error logs - only a local user can get to elmah.axd. If you take care of the security implications it can be very useful to enable remote access and monitor the logs on your production servers.

We chose to use an XML file for each error but ELMAH is entirely pluggable. There are dozens of alternatives for persisting the error log including Sql Server, an RSS feeds, to Twitter, even to an iPhone app. There are even third party sites such as elmah.io who will host your error logs for you.

One of the advantages of using XML files is that the files can be copied to another machine. If you look in MainDemo.Web\App_Data\Elmah.Errors, you will find the resulting xml files.