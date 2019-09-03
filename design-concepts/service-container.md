---
description: '@fusion.io/container'
---

# Service Container

_Fusion Framework was design with Dependency Injection \(DI\) in mind. To archive a seamlessly DI, the heart of Fusion Framework is a service container._

## Dependency Injection Introduction

In short, Dependency Injection \(DI\) is a technique whereby passing or _injecting_ objects \(_the dependencies/services_\) to another object to use via constructor or getter methods. Using DI properly, your application will be easier to test and better code reuse. \([more about DI](https://martinfowler.com/articles/injection.html)\). 

For example, we are to developing booking service:

{% code-tabs %}
{% code-tabs-item title="di-example.js" %}
```javascript
class BookingService {

    constructor(ticketStore, emailService) {
        this.ticketStore  = ticketStore;
        this.emailService = emailService;
    }
    
    async book(user) {
        const foundTicket = await this.ticketStore.find();
        
        foundTicket.markAsBooked(user);
        
        await this.ticketStore.update(foundTicket);
        await this.emailService.send(ticket, user);
    }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

