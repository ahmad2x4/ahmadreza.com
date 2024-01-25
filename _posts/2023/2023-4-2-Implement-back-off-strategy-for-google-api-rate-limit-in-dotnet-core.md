---
layout: post
title: Implement back off stragety for Google API rate limit in .net core
published: true
author: ahmadreza
comments: false
categories: [blog]
tags: [Google API,Google Workspaces,Rate Limit,Polly]
shareimg: /img/Implement-back-off-strategy-for-google-api-rate-limit-in-dotnet-core.png
excerpt_separator: <!--more-->
---

Google APIs enforces rate limits for API calls. Google APIs are shared service, and google applies rate limitation to make sure it's used fairly by all user to protect overall perfomance of the Google Workspace system. If you use Google API you could hit this rate limit and this would disrupt your service.  

## Introduction
Implementing back-off strategy is simple in .net core. Thanks to `IHttpClientFactory` and libraries like [Polly](https://github.com/App-vNext/Polly) you can implement [this strategy](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly) for any HTTP Client calls if the HttpClient is used as recommended in .net core. However, if you use [Google API .net client](https://github.com/googleapis/google-api-dotnet-client) it is not as easy as you'd think.

The reason is, google client library is not using the [recommended implementation](https://learn.microsoft.com/en-us/dotnet/core/extensions/httpclient-factory) for creating HttpClient. I am not exactly sure about the history and the reason Google team did not implement this and took their own approach of having their onw [IHttpClientFactory](https://github.com/googleapis/google-api-dotnet-client/blob/main/Src/Support/Google.Apis.Core/Http/IHttpClientFactory.cs). I can only guess it is because of the original implementation back in 2013 and reusing the same code base for .net core.


## Google API Code sample

I this example we are going to use Google Workspace API (Google Drive in my example). To test this simple application you need to [create a Google cloud project](https://developers.google.com/workspace/guides/create-project) and [enable google workspace APIs](https://developers.google.com/workspace/guides/enable-apis).





## Is it possible to use IHttpClientFactory in MicrosoftExtensions.Http?
Although the original design does not let developer use Microsoft IHttpClientFactory but the Google team did their best to make this possible. For more details you can read [issue 1756](https://github.com/googleapis/google-api-dotnet-client/issues/1756) 
