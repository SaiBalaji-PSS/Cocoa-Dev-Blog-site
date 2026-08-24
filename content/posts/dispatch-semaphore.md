+++
date = '2026-08-24T20:32:32+05:30'
draft = false
title = 'Dispatch Semaphore'
tags = ['swift', 'ios']
categories = ['general']
summary = 'Dispatch Semaphore explained'
+++

# Dispatch Semaphore

* It is an object which is used to control the access shared resource. When two or more process atleast with one write operation access a shared piece of data then it cause irregular updates to the data. It is called as race condition. 

* It can be prevented by using Semaphores. It is used to provide synchoronization between the processes when accessing a shared piece of data. 

* It consist of an integer value which represents number of processes which can access the shared resources in given time.

* When a process gains access to the resource it will decrement the integer value. When the integer value becomes 0 no other process can access the shared resource

* When a process completes accessing the shared resource it will increment the counter value indicating that next process can access the shared resource.

## Creating a Dispatch Semaphore 

We can create a sempahore by creating an object for DispatchSempahore class. Its constructor will look like this 

```swift
init(value: Int)
```

* ```value``` The starting value for the semaphore. Do not pass a value less than zero.

The DispatchSemaphore object will have two methods 

```wait()``` - Decrements the integer value. Called when a process is about to access the shared resource.
```signal()``` - Increments the integer value. Called when a process finishes accessing the shared resource.

##Common example 

```swift
class BankAccount {
    private var balance: Int
    init(balance: Int) {
        self.balance = balance
    }

    // NO semaphore — read, then write, with a gap in between
    func deposit(amount: Int) {
        let current = balance      // READ
        balance = current + amount // WRITE
    }

    func getBalance() -> Int {
        return balance
    }
}

let account = BankAccount(balance: 0)
let iterations = 1000

// Runs 1000 iterations across multiple threads concurrently,
// and blocks the current thread until ALL of them finish
DispatchQueue.concurrentPerform(iterations: iterations) { _ in
    account.deposit(amount: 1)
}

print("Expected balance: \(iterations)")
print("Actual balance:   \(account.getBalance())")

```

Here 1000 concurrent process access the shared data balance. The output varies each time you run it and the Actual balance will never be same as expected balance. 
Because all the 1000 processes are executed concurrently by multi-core hardware on multiple threads they are not executed one after the another. 
Consider thread A reads the balance as 50 before it can write back other threads like thread B, C, D etc also reads 50. So all of them write back 51 which causes data inconsistency. This is called race condition.

```
Expected balance: 1000
Actual balance:   349
```


This can be prevented by using DispatchSemaphore which provides synchronisation when accessing a shared resource.

```swift
class BankAccount {
    private var balance: Int
    var sempahore = DispatchSemaphore(value: 1) //only one process can access the shared resource in given time
    init(balance: Int) {
        self.balance = balance
    }

    // NO semaphore — read, then write, with a gap in between
    func deposit(amount: Int) {
        defer{
            sempahore.signal()
        }
        sempahore.wait()
        let current = balance      // READ
        balance = current + amount // WRITE
    }

    func getBalance() -> Int {
        return balance
    }
}

let account = BankAccount(balance: 0)
let iterations = 1000

// Runs 1000 iterations across multiple threads concurrently,
// and blocks the current thread until ALL of them finish
DispatchQueue.concurrentPerform(iterations: iterations) { _ in
    account.deposit(amount: 1)
}

print("Expected balance: \(iterations)")
print("Actual balance:   \(account.getBalance())")

```


