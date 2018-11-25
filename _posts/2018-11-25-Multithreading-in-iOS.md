---
layout: post
title: 📱 Conquer the multithread fear in iOS - Deadlocks
date: '2018-11-25T11:40:00.005-07:00'
author: Anu Mittal
categories: 'anu'
subclass: post
tags:
- iOS
- tech
- multithreading
---

<br/>
With the emergence of multi-core processing chips in all our mobile devices, it has given rise to the possibility of performing multiple tasks simultaneously in the application. In iOS, **in order to keep the application’s UI responsive** 📲, all the heavy task are added in a new thread e.g. networking calls. These threads usually run in the background and send a call back to update the UI if necessary.

Need of multithreading has been perfectly stated in the apple documentation, 
it says:

> Each process (application) in OS X or iOS is made up of one or more threads, each of which represents a single path of execution through the application's code. Every application starts with a single thread, which runs the application's main function. Applications can spawn additional threads, each of which executes the code of a specific function.

  
In order to facilitate and safely manage these additional threads we need a framework like [dispatch][https://developer.apple.com/documentation/DISPATCH]


✍🏼 **Ways to achieve multithreading:**  
- Grand Central Dispatch  
- [NSThread][https://developer.apple.com/documentation/foundation/thread]  
- [NSOperationQueue][https://developer.apple.com/documentation/foundation/operationqueue]  

This blog assumes you have basic understanding of these types.

When it comes to comparing these methods: 

`Code/Structural Origins:`  

* GCD is a low-level C-based API.  
* NSOperation and NSOperationQueue are Objective-C classes (a wrapper on GCD).  
* NSThread is an NSObject (uses pthreads).

`Complexity comparison:`  

* For GCD implementation is very light-weight  
* NSOperationQueue is complex and heavy-weight  
* NSThread is just wrapper to pthreads therefore making it lightweight too.

`NSOperation advantages over GCD and NSThread:`  
You can:

* Set up a dependency between two NSOperations hence enables developers to execute tasks in a specific order.
* Pause, cancel, resume an NSOperation once the task as started its execution therefore give it control over the operation's life cycle.
* Monitor the state of an operation like: ready, executing, or finished.
* Specify the maximum number of queued operations that can run simultaneously.

Clearly, NSOperationQueue provides you more control over the operations.

Once you have decided which methodology you want to use, you need to decide the order and priority of it.

`Order of Operation:`  
Queues can be of two type: Serial Queue or Concurrent Queue which runs the tasks synchronously and asynchronously respectively

<!--[Queues][queuesType]-->
<img src="/assets/images/multithreadingiOS/queuesType.png" alt="Queues" width="60%"/>
`Priority of operations:`  
Priority is defined as Quality of Service known as QoS and is of following four types:


<!--[QoS][QoS]-->
<img src="/assets/images/multithreadingiOS/QoS.jpg" alt="QoS" width="80%"/>

Let’s now try to understand these operation by predicting the output of couple of examples.  
Before trying to answer the output of code below, let’s look at the documentation of [dispatch_async][https://developer.apple.com/documentation/dispatch/1453057-dispatch_async] and [dispatch_sync][https://developer.apple.com/documentation/dispatch/dispatchqueue/1452870-sync]:

**Dispatch_async:**  

>  🤞🏼 Declaration  
	void dispatch_async(dispatch_queue_t queue, dispatch_block_t block)  
	👥 Discussion  
	This function is the fundamental mechanism for submitting blocks to a dispatch queue. Calls to this function always return immediately after the block has been submitted and never wait for the block to be invoked. The target queue determines whether the block is invoked serially or concurrently with respect to other blocks submitted to that
	 same queue. Independent serial queues are processed concurrently with respect to each other.  
	💁🏻‍♀️ Summary  
	Submits a block for asynchronous execution on a dispatch queue and returns immediately.

**Dispatch_sync:**  

>  🤞🏼 Declaration  
	void dispatch_sync(dispatch_queue_t queue, dispatch_block_t block);  
	👥 Discussion  
	Submits a block to a dispatch queue for synchronous execution. Unlike dispatch_async, this function does not return until the block has finished. Calling this function and targeting the current queue results in deadlock.
	Unlike with dispatch_async, no retain is performed on the target queue. Because calls to this function are synchronous, it "borrows" the reference of the caller. Moreover, no Block_copy is performed on the block.
	As an optimization, this function invokes the block on the current thread when possible.  
	💁🏻‍♀️ Summary  
	Submits a block object for execution on a dispatch queue and waits until that block completes.


Now try these out:  
In which order would these logs get printed? Suggest you to give it a try before looking at the solution.

<!--[async Sync Quest][asyncSyncQuest]-->
<img src="/assets/images/multithreadingiOS/async_sync_quest.jpg" alt="asyncSyncQuest" width="100%"/>

What is given to us?  
A new “queue” is created from global queue (which means it’s a concurrent queue).
We are given 2 dispatch task and few NSLog to be printed to understand the flow.

So the output will be:  

<!--[Async Sync Output][asyncSync]-->
<img src="/assets/images/multithreadingiOS/async_sync.png" alt="asyncSync" width="100%"/>

As we were expecting the string after the async block is printed first, followed by the tasks under the block.
The reason is quite clearly stated in the documentation, the async task return immediately after the submission. 

Agree to me? BUT…  


There is a twist: 🔀   
As we have created a concurrent queue from the global queue and added it to an async dispatch, sure it returns immediately after submission, but the whole point of creating a async dispatch is to allow multiple threads to execute concurrently. So here the main queue (the task of printing “anumittal %num”) and the task in the async dispatch queue can run simultaneously. So you can also expect output in this order:

<!--[Async Sync Output Random][asyncSyncRandom]-->
<img src="/assets/images/multithreadingiOS/async_sync_random.png" alt="asyncSyncRandom" width="100%"/>

Making sense?

Similarly, now predict the output if instead of dispatch_async in the first example we had dispatch sync.
Once again before looking at solution, try to answer it yourself.

<!--[sync sync Output][syncSync]-->
<img src="/assets/images/multithreadingiOS/async_async.png" alt="syncSync" width="100%"/>


As the documentation says: “this function does not return until the block has finished” 
In the sync_sync block, everything happens in the most calmest way. :blush: it prints “1 01 001” then comes another dispatch_sync and the outer one will not continue to line 46 until line 41 to 44 are executed.

But suppose the inner queue was async then it would immediately return and line after 45 and after 40 will execute simultaneously.
<!--[sync async Output][syncAsync]-->
<img src="/assets/images/multithreadingiOS/async_async.png" alt="syncAsync" width="100%"/>

Last variant would be when both the queues are dispatch_async. 😃  

I am sure you must have guessed the output.
<!--[Async async Output][asyncAsync]-->
<img src="/assets/images/multithreadingiOS/async_async.png" alt="asyncAsync" width="100%"/>


It will be all concurrent.

Congratulations 👏🏼 you have understood the essentials of multithreading. If you get confident with these examples, I am sure you can now play around with threads in your applications.

Also, you must try creating your own new queues. 

Syntax:  
**Serial queue:**  
```dispatch_queue_t serial_queue = dispatch_queue_create("your.queue.label", DISPATCH_QUEUE_SERIAL);```

**Concurrent queue:**  
```dispatch_queue_t concurrent_queue = dispatch_queue_create("your.queue.label", DISPATCH_QUEUE_CONCURRENT);```


**Deadlocks**  
You should always resist passing same queue (especially a serial queue) to multiple dispatch blocks, as it might lead to deadlock.


For example.
<!--[Serial Queue Deadlock][serialQueueDeadlock]-->
<img src="/assets/images/multithreadingiOS/async_async.png" alt="serialQueueDeadlock" width="100%"/>

The code leads to deadlock condition as, the outer async operation waits for the inner block to start and complete, whereas the inner block will not start until the task in “queue” is completed. 

Where as works fine with concurrent queue :
<!--[new concurrent queue][newConcurrentQueue]-->
<img src="/assets/images/multithreadingiOS/async_async.png" alt="newConcurrentQueue" width="100%"/>

*Just to emphasise, this will work but it is not preferred to use the same queue*

Lastly a good to know example of deadlock:
<!--[smallestDeadlock][smallestDeadlock]-->
<img src="/assets/images/multithreadingiOS/async_async.png" alt="smallestDeadlock" width="100%"/>
This is claimed to be the shortest code to create a deadlock. I hope by now you must be able to think in the direction to answer as to why would this create a deadlock.

`Hint:` main queue is a serial queue, this code is trying to dispatch the new code block synchronously on the same queue which is running & waiting on the dispatch_sync.

If this all feels little overwhelming then it’s ok. The more you try out examples the easier it is to understand. Please let me know if you would like to learn any of these topics more in detail.
You can find the project on my [github][https://github.com/anumittal/multithreading-iOS].

In the next blog, we shall look into locks and the most famous readers/writers problem 😊 
Thanks for reading. 👓


[smallestDeadlock]: /assets/images/multithreadingiOS/smallestDeadlock.png
[newConcurrentQueue]: /assets/images/multithreadingiOS/newConcurrentQueue.png
[serialQueueDeadlock]: /assets/images/multithreadingiOS/serialQueueDeadlock.png
[asyncAsync]: /assets/images/multithreadingiOS/async_async.png
[queuesType]: /assets/images/multithreadingiOS/queuesType.png
[QoS]: /assets/images/multithreadingiOS/QoS.jpg
[asyncSyncQuest]: /assets/images/multithreadingiOS/async_sync_quest.jpg
[asyncSync]: /assets/images/multithreadingiOS/async_sync.png
[asyncSyncRandom]: /assets/images/multithreadingiOS/async_sync_random.png
[syncAsync]: /assets/images/multithreadingiOS/sync_async.png
[syncSync]: /assets/images/multithreadingiOS/sync_sync.png