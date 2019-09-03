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
const ticketStore    = new TicketStore();
const emailService   = new EmailService();

const bookingService = new BookingService(ticketStore, emailService);

// ...

await bookingSerivce.book(user);

```
{% endcode-tabs-item %}
{% endcode-tabs %}





