---
title: Spring使用Junit测试异常
date: 2017-08-30 16:26:04
tags:
categories: 总结
comments: true
---

### 异常现象 

近日一个老项目中,使用spring junit测试出现了如下异常,比较少见,如此记录一下

```
java.lang.Exception: No tests found matching [{ExactMatcher:fDisplayName=testSpring], {ExactMatcher:fDisplayName=testSpring(com.xxx.xxx.TestSpring)], {LeadingIdentifierMatcher:fClassName=com.xxx.xxx.TestSpring,fLeadingIdentifier=testSpring]] from org.junit.internal.requests.ClassRequest@46fbb2c1
	at org.junit.internal.requests.FilterRequest.getRunner(FilterRequest.java:37)
	at org.eclipse.jdt.internal.junit4.runner.JUnit4TestLoader.createFilteredTest(JUnit4TestLoader.java:77)
	at org.eclipse.jdt.internal.junit4.runner.JUnit4TestLoader.createTest(JUnit4TestLoader.java:68)
	at org.eclipse.jdt.internal.junit4.runner.JUnit4TestLoader.loadTests(JUnit4TestLoader.java:43)
	at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:444)
	at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:678)
	at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.run(RemoteTestRunner.java:382)
	at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.main(RemoteTestRunner.java:192)

```

### 原因

发现是junit版本过低导致,spring版本为4.3.2,junit版本为4.9

### 解决方案

升级junit版本为4.12

[参考这里](https://stackoverflow.com/questions/34057379/java-lang-exception-no-tests-found-matching)


