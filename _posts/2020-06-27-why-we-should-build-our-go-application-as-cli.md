---
title: Why we should build our Go application as CLI
comments: true
categories:
- tech
- golang
layout: post
date: '2020-06-27 10:00:00 +0700'
---

![](/assets/2020-06-27-why-we-should-build-our-go-application-as-cli-0.jpeg)

So here we Go again, another opinion based on my experience create Go application. This time, i will discuss about CLI in Go and why it's matter. Atleast, it's matter for me because it's add operability to my application. I'll explain later in this article.

> A command-line interface (CLI) processes commands to a computer program in the form of lines of text. The program which handles the interface is called a command-line interpreter or command-line processor.
>

It's common for software engineer to use CLI, such as `cp`, `pwd`, `rm -rf /` (Please don't try the last one). It's also common for programming language to have CLI to make development easier, take an example for [Rails CLI](https://guides.rubyonrails.org/command_line.html), or [Laravel Artisan CLI](https://laravel.com/docs/7.x/artisan).

But in this case, we will not make our Go application into CLI so it's make our development easier. Instead we will make it as an interface to make our application more operable.

In other programming language, debugging in production environment is so easy. But in go, where we compiled our application into a single binary program it could be difficult to debug what happen in production environment.

Imagine this scenario, where we create a service that has a lot of dependency like:

- MYSQL
- Redis
- Service A
- Service B

![Example Scenario](/assets/2020-06-27-why-we-should-build-our-go-application-as-cli-1.png)

In php or rails it could be so easy to check wether we could establish connection into our databases. In rails we could use `rails console` and type

```ruby
ActiveRecord::Base.establish_connection
```

to do integration test to our databases. Also we could check how much pool in our database by using

```ruby
ActiveRecord::Base.connection_pool.stat
```

Or maybe doing some scripts that directly connect into our application business logic

```ruby
OurService::ServiceA.add_to_cart(user_id: 1, product_id: 1)
```

You might be thinking, what if we create GUI for something administrational like this? And just create monitoring to our databases so we could check it from monitoring dashboard instead of productions console.

My answer is, yes you could! Absolutely. Monitoring is essentials, i aggree with that. But this is the world of software engineering, where something everything could change. Imagine a scenario where you should change your database instance into a new one, you couldn't be deploy your application without knowing if it can connect into new database instance. You should deploy your application first, check if it's connect rightly into your new database instance, then change your traffic pointing to a new deployment or replace the current deployment with new config.

In rails you could do it so easily, because you have the ability to enter the console and just type what you need. But in Go? The simplest way to do it is run your server, curl into some API and see the response of that API. If it's return 500 then your config is wrong. There is also no guarantee if it's return 200 then your database config is okay.

Oh, and GUI for something administrational it's case by case. For big scale application it's a must, because there are administrational team which will be need that I assume. For team that consist of 2 engineer, no administrational team or small startup it's waste of time in my opinion. With GUI you just creating another layer of interface above of the CLI itself, when what you need is speed than CLI is your best friend.

So come the question, how you do it in Go? How to make your Go binary is more operable and adding some more function to it.

Here come the solution (atleast for me) [Cobra CLI](https://github.com/spf13/cobra). The same package which used by [kubectl](https://github.com/kubernetes/kubectl/blob/42bf054e195ad7984972b8d6939e3ce835cb8682/go.mod#L31).

With this package we could create our `./api` Go binary into Something like this:

```
Example CLI command in Go using cobra.

Usage:
  our_service [flags]
  our_service [command]

Available Commands:
  help               Help about any command
  ping:mysql         Check mysql availability
  ping:redis         Check redis availability
  ping:service_a     Ping service a.
  ping:service_b     Ping service b.
  run:api            Run API
  script:add_to_cart Add to cart based on user_id and product_id

Flags:
  -h, --help   help for our_service

Use "our_service [command] --help" for more information about a command.
```

Now, doing integration test with your Golang binary is so easy. If you want to test mysql connection, just type `./our_service ping:mysql` and it will run based on the connection logic of your application. Or if you want to run script in your golang binary just type `./our_service script:add_to_cart [user_id] [product_id]`. 

You can even install your binary into you `/usr/local/bin` by using `go install`, then next time you could run your service just with `our_service run:api`, no need to compiling it.

This is the sample respository that i create during creating this article to give you more visibility about the code. [Insomnius/Example-Go-Cli](https://github.com/insomnius/example-go-cli).

That's it, i hope you get some nice insight from this article that you can use later in your project. Thanks for reading.
