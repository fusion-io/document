---
description: '@fusion.io/container'
---

# Service Container

_Fusion Framework was designed with Dependency Injection \(DI\) in mind. To archive a seamlessly DI, the heart of Fusion Framework is a service container._

## Dependency Injection Introduction

In short, **Dependency Injection** \(DI\) is a technique whereby passing or _injecting_ objects \(_the dependencies/services_\) to another object via constructor or getter methods. Using DI properly, your application will be easier to test and better code reuse [more about DI](https://martinfowler.com/articles/injection.html). 

For example, _**a booking service will find a ticket for an user, mark it as booked and send an email to the user about the ticket**_.

{% code-tabs %}
{% code-tabs-item title="BookingService.js" %}
```javascript
class BookingService {

    constructor(ticketStore, emailService) {
        this.ticketStore  = ticketStore;
        this.emailService = emailService;
    }
    
    async book(user) {
        const foundTicket = await this.ticketStore.find();
                
        await this.ticketStore.booked(foundTicket, user);
        
        await this.emailService.send(ticket, user.getEmail());
    }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

In the above example, we have injected 2 services into the `BookService`. `ticketStore` for finding an available ticket, and `emailService` for sending the ticket to the user.

To use this `BookingService`, we have to initialize the `ticketStore` and `emailService` first:

{% code-tabs %}
{% code-tabs-item title="main.js" %}
```javascript
const bookingService = new BookingService(
    new TicketStore(), 
    new EmailService()
);

// ...

await bookingService.book(user);

```
{% endcode-tabs-item %}
{% endcode-tabs %}

_**The trade-off of DI is the complexity of initialization steps**_. Especially when the services are using in a nested form.   
For example, the `ticketStore` might need a `DatabaseConnection` to perform SQL query and `emailService` might need an `SMPT` protocol for sending the email:

{% code-tabs %}
{% code-tabs-item title="main.js" %}
```javascript
const bookingService = new BookingService(
    new TicketStore(new DatabaseConnection()), 
    new EmailService(new SMTP())
);

// ...

await bookingService.book(user);

```
{% endcode-tabs-item %}
{% endcode-tabs %}

To create an instance of `BookingService`, we first need to know how to create a `TicketStore` service, which in turns need to know how to create a `DatabaseConnection` and so on... In real world application, the source code is even more complicated. 

The Service Container of Fusion Framework was created to address this problem.

{% hint style="info" %}
**Service Container** has another name called **Inversion of Control Container** or **IoC Container**.

Fusion Service Container was inspired by [**Laravel's Service Container**](https://laravel.com/docs/6.0/container#introduction).
{% endhint %}

## Service Container

_A service container is a simple map object for holding and getting dependencies/services.   
To use the Service Container, we need to install it from NPM_

```bash
npm install @fusion.io/container
```

### Binding  basics

To bind a service into Service Container, we'll use the `.bind()` method:

{% code-tabs %}
{% code-tabs-item title="bind-example.js" %}
```javascript
import { container } from '@fusion.io/container';
import HelloService from './HelloService';

container.bind('helloService', () => new HelloService());
```
{% endcode-tabs-item %}

{% code-tabs-item title="HelloService.js" %}
```javascript
export default class HelloService {
    
    sayHello() {
        return 'Hello World';
    }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

The `.bind()` method requires 2 parameters: the dependency key `helloService` and a function returning an instance of the service - we call it as a `Factory Function`.

After we bind a service, we can retrieve it from the container by calling the `.make()` method:

```javascript
const helloService = container.make('helloService');

console.log(helloService.sayHello()); // Hello World
```

We can also call the `.make()` method inside the a `Factory Function`. By doing so we can  instruct the container how to make a service with nested dependencies easily:

```javascript
container.bind('fooService', () => new FooService());
container.bind('barService', () => new BarService(container.make('fooService')));
container.bind('fooBarService', () => new FooBarService(container.make('barService')));

const fooBarService = container.make('fooBarService');
```

In the above example, we have a `fooBarService` is using the `barService` as a dependency. And the `barService` is using `fooService` as a dependency for itself.

### Singleton binding

If your service was bounded to the to the container via `.bind()` . Every time we call the `.make()` method, the container will invoke the `Factory Function` again to get your service instance. 

```javascript
container.bind('fooService', () => new FooService());

const instance1 = container.make('fooService');
const instance2 = container.make('fooService');

console.log(instance1 === instance2); // false
```

Sometimes, we only need one instance of service or a _singleton_, and every time we calling `make()`, we can get back the same instance. We can archive this by using the `singleton()` method instead of `bind()`:

```javascript
container.singleton('fooService', () => new FooService());

const instance1 = container.make('fooService');
const instance2 = container.make('fooService');

console.log(instance1 === instance2); // true
```

### Auto bindings

{% hint style="info" %}
Thanks to ES6 Map, the dependency key is not limited to string value, but any value can be used. Even a class

```javascript
class FooService {

}

container.bind(FooService, () => new FooService());
```
{% endhint %}

Most of the time, you may find your factory function is doing nothing but 2 steps: making dependencies and creating a new service instance with those dependencies:

```javascript
class Dependency1 {

}

class Dependency2 {

}

class MyService {
    constructor(dependency1, dependency2) {
        this.dependency1 = dependency1;
        this.dependency2 = dependency2;
    }
    
    // ...
}

container.bind(Dependency1, () => new Dependency1());
container.bind(Dependency2, () => new Dependency2());

// We'll do this most of the time
container.bind(MyService, () => new MyService(
    container.make(Dependency1),
    container.make(Dependency2)
));
```

So instead of doing this again and again, we support an `autoBind()` method. The above example can be equivalent to:

```javascript
class Dependency1 {
    static get dependencies() {
        return [];
    }
}

class Dependency2 {
    static get dependencies() {
        return [];
    }
}

class MyService {
    constructor(dependency1, dependency2) {
        this.dependency1 = dependency1;
        this.dependency2 = dependency2;
    }
    
    static get dependencies() {
        return [Dependency1, Dependency2];
    }
    
    // ...
}

container.autoBind(Dependency1);
container.autoBind(Dependency2);
container.autoBind(Dependency1);
```

Similar to `autoBind()`, we also support `autoSingleton()`.

We also support `bind()`, `singleton()` functions as ES7 decorators, so you can shorten the above code to this:

```javascript
import { bind } from '@fusion.io/container';

@bind()
class Dependency1 {

}

@bind()
class Dependency2 {

}

@bind(Dependency1, Dependency2)
class MyService {
    constructor(dependency1, dependency2) {
        this.dependency1 = dependency1;
        this.dependency2 = dependency2;
    }
    
    // ...
}
```

