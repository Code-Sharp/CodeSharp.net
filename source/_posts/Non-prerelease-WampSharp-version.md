---
title: Non-prerelease WampSharp version
date: 2018-03-14 22:07:47
tags:
 - WampSharp
 - WAMP
---
Yesterday I released the first non-prerelease version of [WampSharp](http://wampsharp.net). This was requested and discussed by the community in this [issue](https://github.com/Code-Sharp/WampSharp/issues/237).

I thought to take this opportunity to write a bit about this open source project, and my journey in the WAMP world.

I began working on WampSharp in April 2013. I looked for a WebSocket subprotocol which supports both RPC and PubSub, and came across [this answer on StackOverflow](https://stackoverflow.com/a/10882808). The first version [was released](https://groups.google.com/d/msg/wampws/dR2j0U6E5m4/ijkrlHCgbF0J) around July 2013 and contained [WAMP v1](https://web.archive.org/web/20130511044655/http://wamp.ws:80/spec) support for both server and client sides. During its first year, the project did not receive almost any attention from the community.

Around December 2013, work on [WAMP v2 spec](https://groups.google.com/d/msg/wampws/7KykR53SP1w/kzoIWV5JOP8J) began. Around late September 2014, the first WampSharp supporting [WAMP v2](http://wamp-proto.org/spec/) was [released](https://groups.google.com/d/msg/wampws/lLxrrRYDozw/DVLx31-p7sAJ). Since then, I've released every year at least one major WampSharp version, and several hotfix versions, summing up to 53 versions. Usually the annual major version tries to catch up with [Crossbar.io](https://crossbar.io).

### Community

Nowadays, WampSharp has dozens of users, spread all over the world. A lot of WampSharp's development is due to the community. This involves some features (mainly authentication features), bug reporting and bug fixes. There were [several times](https://github.com/Code-Sharp/WampSharp/issues/92) where community members [helped me](https://gist.github.com/darkl/28380eced333ac8ff0fcc4df6cb2f21e#gistcomment-1975910) figure out how to reproduce a previously reported bug.

### Contribution to other open source projects

WampSharp's presence made me and others contribute to other non-WAMP related projects, as some of the issues filed for WampSharp turned out to be issues of other libraries. These include [feature requests](https://github.com/statianzo/Fleck/issues/70) and [issue filing](https://github.com/vtortola/WebSocketListener/issues/47) for WebSockets libraries, and sometimes even [pull requests](https://github.com/kerryjiang/WebSocket4Net/pull/41) to resolve these! Beside these, I submitted [various pull requests](https://github.com/damianh/LibLog/pulls?q=is%3Apr+author%3Adarkl+is%3Aclosed) for LibLog and also filed issues for the [.NET MessagePack implementation](https://github.com/msgpack/msgpack-cli/issues/39), and even [for Mono](https://bugzilla.xamarin.com/show_bug.cgi?id=13110) and [IIS Express](https://github.com/aspnet/AspNetKatana/issues/155)! These were all found via WampSharp related scenarios.

I also tried to contribute to the WAMP ecosystem in various ways. These include providing a [C# template](https://github.com/crossbario/crossbar/pull/135) to Crossbar.io, providing [type declarations](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/3956) for [AutobahnJS](https://github.com/crossbario/autobahn-js) (the most popular JavaScript implementation of WAMP), providing WAMP messages parsing code for a [Firefox developer extension](https://github.com/firebug/websocket-monitor/issues/32). I also created a project called [meta-wamp](https://github.com/darkl/meta-wamp) which provides WAMP messages declaration files, from which one can generate a WAMP related library.

### Bringing WampSharp features to other languages

In my opinion, the [reflection based roles](http://wampsharp.net/tags/reflection/) are among the nicest features WampSharp has to offer. I tried mimicking this in other languages. One attempt was {% post_link introducing-typedautobahn the TypedAutobahn project %}, which, unfortunately, was never completed. I couple months ago, I submitted an initial reflection based features [pull request](https://github.com/crossbario/autobahn-java/pull/356) to [autobahn-java](https://github.com/crossbario/autobahn-java), which was merged. Hopefully, this feature will be useful for WAMP Java developers.

### Future

It seems that WAMPv2 has kind of staled, and that the most critical period of its development was around 2014-2015. Hopefully, .NET Core has staled, and I won't have to make any more crazy changes in order to keep supporting this platform. I do plan to continue maintaining WampSharp, which includes answering questions and handling issues on GitHub. I hope I'll be able to release at least one major version per year.