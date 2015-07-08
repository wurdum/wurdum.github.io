---
layout: post
title: "Some kind of new blog post"
date: 2015-07-07 15:34:44 +0300
comments: true
categories: test-post
---

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