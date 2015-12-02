Purchases. Reservations. Reviews. Forgotten passwords. There are lots of events
in a web application that generate transactional emails.

Often, we want these emails decoupled from the events that generate them. When a
user purchases an item on a website, we want to respond immediately, but sending
a confirmation email can happen outside of the usual request/response flow.

I decoupled business logic from transactional emails recently in an app I'm
working on. I used [RabbitMQ][rabbitmq] and was pleased with how it worked out.

I created two separate services. The first is the API that handles all of the
business logic. The second is a dedicated email sender. And I hooked them
together using RabbitMQ.

When a purchase is made in the API, the API does its usual thing. It then
connects to the RabbitMQ exchange and posts a message with the routing key
`product.purchased`. Like an in-process event, this is basically fire and
forget. "Yeah, a product was purchased. Anybody interested in this event can
deal with it."

And who is interested? The email service. It tells the RabbitMQ server, "I'm
interested in `product.purchased` events. When they come in, let me know." And
when they come in, it sends an email to the purchaser.

And then… requirements changed. The client now wants to get an email when the
purchase is made, too. Because of the decoupling, the API doesn't need to know
this because the API doesn't care about emails. So my API didn't change at all.
The email service deals with emails, so I just had it send an extra email on the
`product.purchased` event.

[rabbitmq]: https://www.rabbitmq.com/
