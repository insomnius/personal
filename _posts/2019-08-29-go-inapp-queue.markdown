---
title: Golang In-app Queue
---

This is my second post in this site, as i mentioned before. I want to use this site as a platform to share my knowledge. This time i want to share a topic about in-app queue, maybe some of you already know what is queue system. But for the sake of others who dont know, i will discuss it a little.

In the context of web service, **queue system** is an asynchronous process that run in the background and there are worker for every queue which run differently from the web service itself.

So let's say, you have an API for user registration and you want to send email in that API. If that process is synchronous, it can be toooo looooong. Because send an email is a heavy process, in order to fasten that API then you had to do an asynchronous process which run differently from the main service. Then came a solution to create a **queue system**, it's basically a tube or a place where you put something from one service to be worked by other service. Let's put it in to an image:

![Queue System](/assets/queue-system.png)

By using queue system we are not blocking the main process of user registration and send an email to registrated user with faster response time. There are several open source technology you can use for queue system, such as:

* [RabbitMQ](https://www.rabbitmq.com/)
* [Apache Kafka](https://kafka.apache.org/)
* [Beanstalkd](https://beanstalkd.github.io/)

Every technology comes with their own scenario to solve a problem, it will cost you more but it's essential for medium to large scale application. We are not gonna talk about that technology here, instead we will make our own in-app queue system using Go with far more cheaper cost than those technology above.

How can we make that? Go, is invented to solve modern programming language problem such as concurrency and garbage collecting, that makes Go a lightweight programming language.

There are feature in Go called Channel, we will use this feature to create our in-app queue system. This image bellow are gonna tell what will we create later:

![Queue System](/assets/in-app-queue.jpg)

So there are asynchronous process inside of web service without blocking user registration process, it's called **Channel**.  Maybe some of you already know, **channels** are the pipes that connect concurrent goroutines. You can send values into channels from one goroutine and receive those values into another goroutine.


Prepare VS Code (or else) we will do some practical stuff!

### 1. Initialize your go app with go module

We will use go module as our package manager, make sure you are using go version 1.11 or above.

```bash
	go mod init github.com/{your-cool-github-username}/inapp-queue
```

Create main package:

```go
// main.go
package main

import "fmt"

var server *http.Server

func main() {
	fmt.Println("In app queue system")
}
```

### 2. Create Web Service Using GIN

I prefer gin to create my web service, if you want to use another http router then go for it. I prefer using master branch when add gin as my dependencies, you can use latest table branch if you want.

```bash
	go get github.com/gin-gonic/gin@master
```

Add this code to **main.go**.

```go
appEngine := gin.Default()
	appEngine.POST("/users", func(c *gin.Context) {

		// Send an email represented by this time sleep
		func() {
			time.Sleep(time.Second*2)
			fmt.Println("Complete send an email")
		}()

		c.JSON(http.StatusCreated, gin.H{
			"data": gin.H{
				"username": "user1",
				"email":    "user1@gmail.com",
			},
			"message": "success create new users",
			"status":  http.StatusCreated,
		})
	})
	server = &http.Server{
		Addr:    ":8080",
		Handler: appEngine,
	}

	if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
		fmt.Printf("Unexpected server error because of: %v\n", err)
	}
```

Run the web service.

```bash
	go run main.go
```

Test the designated URL with curl

```bash
	curl --request POST localhost:8080/users
```

It should return a response like this.

```json
{"data":{"email":"user1@gmail.com","username":"user1"},"message":"success create new users","status":201}
```

And print this in your web service terminal log.

```
	Complete send email
```

The response time of that API should be two second. Actually you can make you process concurent by adding `go` before send email function. Lets try that, change your send email function:

```go
    		go func() {
			time.Sleep(time.Second * 2)
			fmt.Println("Complete send an email")
		}()
```

Yay, we already make it fasten in such an easy way like that. Yes! We could do that, but it isn't safe, why? This is the question, how do we know if there is some concurrency doing their job when we want to restart the server? Say, that there will be a deployment tomorrow, what would we do in the deployment process? Just shutdown the server? Then how we can ensure that every email sent to the registering users, there are probability that some users not receiving an email when in that deployment process.

We should use more reliable method to do this, we have to use go channel and make sure the channel is drained if there is a terminate signal or a kill signal from the OS. Then shutdown the app when there is no left over process in the app. Using that way we can ensure all email sent to registrated users without having so much work or procedure in our deployment process.

## 3. Create Queue Package

Create new directory in your workspace named `queue`:

```go
// queue/email_queue.go

package queue

import (
	"fmt"
	"time"
)

type emailQueue struct {
	emailChannel   chan string
	workingChannel chan bool
}

func NewEmailQueue() *emailQueue {
	emailChannel := make(chan string, 10000)
	workingChannel := make(chan bool, 10000)
	return &emailQueue{
		emailChannel:   emailChannel,
		workingChannel: workingChannel,
	}
}

func (e *emailQueue) Work() {
	for {
		select {
		case eChan := <-e.emailChannel:
			e.workingChannel <- true

			// Let's assume this time sleep is send email process
			time.Sleep(time.Second * 2)
			fmt.Println(eChan)

			<-e.workingChannel
		}
	}
}

func (e *emailQueue) Size() int {
	return len(e.emailChannel) + len(e.workingChannel)
}

func (e *emailQueue) Enqueue(emailString string) {
	e.emailChannel <- emailString
}

```

We have this struct called email queue, there are two channel inside of it. Email and working channel. Email channel is carrying all enqueued data from others package, and working channel is to notify that this emailQueue is working on something.

```go
type emailQueue struct {
	emailChannel   chan string
	workingChannel chan bool
}
```

We have this method called `Size()`, it's actually return that how much work that this emailQueue left to do.

```go
func (e *emailQueue) Size() int {
	return len(e.emailChannel) + len(e.workingChannel)
}
```

And this work method, to fetch data from emailChannel and do the send email job.

```go
func (e *emailQueue) Work() {
	for {
		select {
		case eChan := <-e.emailChannel:
			e.workingChannel <- true

			// Let's assume this time sleep is send email process
			time.Sleep(time.Second * 2)
			fmt.Println(eChan)

			<-e.workingChannel
		}
	}
}
```

## 4. use OS Signal

In order to drain all leftover job in the channel before shutdown the app, then you must listen to OS Signal. When you receive the signal, make sure to drain all leftover job in the channel.

Create osSignal variable in **main.go**:

```go
// inside main.go
var (
	server   *http.Server
	osSignal chan os.Signal
)
```

Listen into interrupt and terminated signal:

```go
// inside main.go
// Initialize channel with the 10K length
osSignal = make(chan os.Signal, 10000)
signal.Notify(osSignal, os.Interrupt, syscall.SIGINT, syscall.SIGTERM)
```

Implement queue logic in your main function, so it would look like this.

```go
// main.go
package main

import (
	"context"
	"fmt"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"

	"github.com/gin-gonic/gin"
	"github.com/insomnius/inapp-queue/queue"
)

var (
	server   *http.Server
	osSignal chan os.Signal
)

func main() {
	// Initialize channel with the 10K length
	osSignal = make(chan os.Signal, 10000)
	signal.Notify(osSignal, os.Interrupt, syscall.SIGINT, syscall.SIGTERM)

	fmt.Println("In app queue system")

	emailQueue := queue.NewEmailQueue()

	appEngine := gin.Default()
	appEngine.POST("/users", func(c *gin.Context) {

		// Enqueue into go channel
		emailQueue.Enqueue("Send email to the user")

		c.JSON(http.StatusCreated, gin.H{
			"data": gin.H{
				"username": "user1",
				"email":    "user1@gmail.com",
			},
			"message": "success create new users",
			"status":  http.StatusCreated,
		})
	})
	server = &http.Server{
		Addr:    ":8080",
		Handler: appEngine,
	}

	go func() {
		if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			fmt.Printf("Unexpected server error because of: %v\n", err)
		}
	}()

	// Spawn 10 worker in that channel
	for i := 0; i < 10; i++ {
		go emailQueue.Work()
	}

	<-osSignal

	fmt.Println("Terminating server")
	server.Shutdown(context.Background())

	fmt.Println("Terminating email queue")
	// Wait untuk there is no active job in the queue
	for emailQueue.Size() > 0 {
		time.Sleep(time.Millisecond * 500)
	}

	fmt.Println("Complete terminating application")
}
```

## 5. Let's test out!

Run the web service:

```bash
go run main.go
```

Spam it!

```bash
curl --request POST localhost:8080/users
```

![Spam it!](/assets/Screenshot%20from%202019-09-29%2017-35-57.png)

Now, shut down the service, with Ctrl + C.

![Server terminating](/assets/Screenshot%20from%202019-09-29%2017-38-29.png)

You will see that after `^C` the service will drain out all leftover jobs before it's shutdown.


## Conclusion

In-app queue is very easy to implement with Go. If you want to see a source code, kindly check [this repo](https://github.com/insomnius/inapp-queue).

This is my first time to write tech article like this, please tell me if you have any suggestion with my write. Thanks, i hope you learn something new today!