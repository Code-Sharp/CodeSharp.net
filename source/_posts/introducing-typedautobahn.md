title: Introducing TypedAutobahn
date: 2016/06/19 17:58:00
tags:
- TypeScript
- TypeAutobahn
- WAMP
- WampSharp
---
In this post I want to introduce a project I've been working on lately.

The project is named TypedAutobahn. It is a project that aims to bring strongly typed programming style for [WAMP](http://wamp-proto.org/) in [TypeScript](http://www.typescriptlang.org).

I don't intend to write here much about WAMP. [WAMP](http://wamp-proto.org/) (not to be confused with [Windows Apache MySQL PHP](http://www.wampserver.com/en/)) is a protocol that provides patterns of Remote Procedure Call and Publish/Subscribe. WAMP usually runs over WebSocket + JSON, but other options exist as well (such as RawSocket and MsgPack). WAMP has various implementations in various languages.

I have written a C# implementation for WAMP, which provides both client and router support, it is called [WampSharp](http://github.com/Code-Sharp/WampSharp).

#### WampSharp reflection based-api

[MSDN]: https://msdn.microsoft.com/en-us/library/bb412178(v=vs.110).aspx

One of the features WampSharp provides, is some reflection-based api for the WAMP roles. The api reminds a bit [WCF's ServiceContracts][MSDN]. For instance, in order in to call a procedure, we define the following contract:

```csharp
public interface ICalculatorProxy
{
    [WampProcedure("com.arguments.add2")]
    Task<int> Add2Async(int a, int b);
}
```

and then request a proxy for it and call the procedure:
```csharp
ICalculatorProxy proxy =
    channel.RealmProxy.Services.GetCalleeProxy<ICalculatorProxy>();

int five = await proxy.Add2Async(2, 3);

Console.WriteLine($"Result: {five}");
```

(This code isn't complete, I am omitting for clarity the code that setups the WampChannel object denoted as the channel variable).

Similarly, registering a procedure is done in the following way:

Define a contract and implement it:

```csharp
public interface ICalculator
{
    [WampProcedure("com.arguments.add2")]
    int Add2(int a, int b);
}

public interface Calculator : ICalculator
{
    public int Add2(int a, int b)
    {
        return (a + b);
    }
}
```

then register an instance of it:

```csharp
ICalculator instance = new Calculator();

IWampRealmProxy realm = channel.RealmProxy;

IAsyncDisposable disposable =
    await channel.RealmProxy.Services.RegisterCallee(instance).ConfigureAwait(false);
```

Similar features are available also for [publisher](http://wampsharp.net/wamp2/roles/publisher/reflection-based-publisher/) and [subscriber](http://wampsharp.net/wamp2/roles/subscriber/reflection-based-subscriber/) roles. ([see also documentation](http://wampsharp.net/))

#### AutobahnJS

[AutobahnJS](http://github.com/crossbario/AutobahnJS) is a WAMP client library written in JavaScript. It runs both on [Node.js](https://nodejs.org/) and browsers. It has been written and maintained by the [Crossbar.io team](http://crossbar.io/) which consists of the people who have created WAMP. AutobahnJS is very popular among WAMP implementations.

Lets review how the preceding samples look using AutobahnJS:

Procedure call:
```javascript
session.call('com.arguments.add2', [2, 3]).then(
   function (res) {
      console.log("Result:", res);
   }
);
```

Procedure registration:
```javascript
function add2(args) {
   return args[0] + args[1];
}

session.register('com.arguments.add2', add2);
```

As before, similar api exists for Publish/Subscribe roles.

#### The problem
Although AutobahnJS api is very simple, it might be challenging to consume services which have a large domain.

Suppose you have written a service that takes a use of lot of contracts in C#. Suppose that now you want to consume that service from JavaScript. It might be challenging to be familiar with all service aspects from JavaScript, since JavaScript is a dynamic-typed language, i.e. has no "hard-coded types", and therefore you don't have intellisense.

So you might consider using [TypeScript](http://www.typescriptlang.org), which allows you to define types and interfaces and enjoy advantages of static-typed languages (such as intellisense) for JavaScript programming.

Luckily, there exist [type declarations for AutobahnJS](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/autobahn/index.d.ts) that give intellisense for AutobahnJS (These declarations were mostly written by me). These give intellisense for AutobahnJS, but don't help you consume your services.

It would be nice if we could just "export" our domain written C# to TypeScript, so we could immediately start consuming our services from TypeScript. This is what TypedAutobahn attempts to achieve.

#### TypedAutobahn

TypedAutobahn currently consists of two parts: a (relatively small) TypeScript library which consists of types and methods that are been used by generated code, and a code generator written in C# which generates code based on given contracts.

For instance, suppose we have the following contract in C#:
```csharp
public interface IArgumentsService
{
    [WampProcedure("com.arguments.ping")]
    void Ping();

    [WampProcedure("com.arguments.add2")]
    int Add2(int a, int b);

    [WampProcedure("com.arguments.stars")]
    string Stars(string nick = "somebody", int stars = 0);

    [WampProcedure("com.arguments.orders")]
    string[] Orders(string product, int limit = 5);
}
```

Running the code generator for this contract yields some code which contains the following contracts:

```typescript
interface IArgumentsService {
    ping(): When.Promise<void> | void;

    add2(a : number, b : number): When.Promise<number> | number;

    stars(nick? : string, stars? : number): When.Promise<string> | string;

    orders(product : string, limit? : number): When.Promise<string[]> | string[];
}

interface IArgumentsServiceProxy {
    ping(): When.Promise<void>;

    add2(a : number, b : number): When.Promise<number>;

    stars(nick? : string, stars? : number): When.Promise<string>;

    orders(product : string, limit? : number): When.Promise<string[]>;
}
```

A class named IArgumentsServiceProvider which allows us to consume the services is also generated.

Consuming caller services with TypedAutobahn has the following form:
```typescript
let serviceProvider = new IArgumentsServiceProvider(session);

let proxy = serviceProvider.getCalleeProxy();

proxy.ping().then(() => console.log("Pinged!"));
proxy.add2(2, 3).then(res => console.log("Add2:", res));

proxy.stars().then(res => console.log("Starred 1:", res));
proxy.stars("Homer").then(res => console.log("Starred 2:", res));
proxy.stars(null, 5).then(res => console.log("Starred 3:", res));
proxy.stars("Homer", 5).then(res => console.log("Starred 4:", res));

proxy.orders("coffee").then(res => "Orders 1:", res);
proxy.orders("coffee", 10).then(res => "Orders 2:", res);
```

The nice thing is that we have intellisense - the editor gives us auto-completion for proxy methods and for their return values. Also, we don't need to be worried about the WAMP procedure uris - these are specified in the generated code. The sample here is rather simple, but the generator also generates contracts for composite types your methods receive/return.

This is even more incredible than it looks: if we're targeting Node.js, we can use [async/await](https://blogs.msdn.microsoft.com/typescript/2015/11/03/what-about-asyncawait/):
```typescript
var five = await proxy.add2(2, 3);
console.log("Add2: ", five);
```

Similarly, we can register WAMP procedures using the IArgumentsServiceProvider. Consider this implementation for instance:

```typescript
class ArgumentsServiceCallee implements IArgumentsService {
    private _orders: number[];

    constructor() {
        this._orders = [];

        for (let i = 0; i < 50; i++) {
            this._orders.push(i);
        }
    }

    ping(): void {
    }

    add2(a: number, b: number): number {
        return (a + b);
    }

    stars(nick: string = "somebody", stars: number = 0): string {
        return (nick + " starred " + stars + "x");
    }

    orders(product: string, limit: number = 5): string[] {
        return this._orders.slice(0, limit);
    }
}

```

We simply register it using

```typescript
let serviceProvider = new IArgumentsServiceProvider(session);
let instance = new ArgumentsServiceCallee();
let promise = serviceProvider.registerCallee(instance);
promise.then(() => console.log("all registered!"));
```

This registers all methods of the IArgumentsService contract, using the procedure uris specified in the C# code. Note that the methods no longer receive an arguments array that needs to be manually unpacked, but TypedAutobahn unpacks it automatically: first it searches for positional arguments, and if those aren't present, it searches for keyword arguments having the names of the method arguments. If none of these are found, the default value specified is used.

A similar mechanism which mimics [WampSharp's reflection-based subscriber](http://wampsharp.net/wamp2/roles/subscriber/reflection-based-subscriber/) exists for subscribers.

#### Current status

I made the TypedAutobahn repository public not long ago. You can find it [on GitHub](http://github.com/darkl/TypedAutobahn).
There are a few things that are missing: regarding features, there is currently no support for publisher role.
Regarding other stuff: some work on deployment needs to be done: the project currently targets Node (since it uses export/import syntax). A build that creates a version that is browser-friendly should be set. Also, a package for npm/bower/NuGet should be created and published automatically on builds.
I should probably open GitHub issues regarding these. Any help on these or on anything else would be appreciated!

#### Future

TypedAutobahn is cool, but it is very limiting, since it requires you to have C# code representing your WAMP service contracts. It would be awesome if we could be able to generate code for WAMP service contracts, regardless the language they are written in. This should be possible in the future, since WAMP has [reflection](https://github.com/tavendo/WAMP/issues/61) on its roadmap (See also these links: [#1](https://groups.google.com/forum/#!searchin/wampws/swagger/wampws/5Z2o25vx9nQ/1CR1LaSmKawJ) [#2](https://groups.google.com/forum/#!msg/wampws/jW_6UZYBhpQ/u8MQ70NMzrYJ) [#3](https://github.com/crossbario/crossbar/blob/master/DEVELOPERS.md) [#4](https://github.com/crossbario/crossbar/issues/301)).
This feature should allow to generate code representing a WAMP service contract of a living contract. Once it would be present, the current code generator could be replaced with a new code generator (probably written in JavaScript/TypeScript) which allows to generate code to many target languages regardless of the language the original service was written in.