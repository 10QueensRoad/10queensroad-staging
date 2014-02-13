---
layout: post
title: "Builders and Providers"
category: design-patterns
tags: [ddd, java, builder, provider]
author: "Andrew Hamilton"
---

In this blog post we will implement a flexible helper object that can:

  1. Keep track of which user groups we want to email.
  2. Query objects in the user domain to retrieve the users’ email addresses.
  3. Provide an email builder with the users’ email addresses using a standard interface.

Our solution implements the [Provider interface model](http://en.wikipedia.org/wiki/Provider_model) to limit dependencies on the email builder logic, mixed with traits of the [Builder pattern](http://en.wikipedia.org/wiki/Builder_pattern) to sequentially keep track of the recipients that we intend to email.

<!--end excerpt-->

Here is a class diagram of the solution we should end up with:

![class diagram](/assets/images/2013-12-07-builders-and-providers/class-diagram.png "Class diagram of EmailBuilder, EmailRecipientProvider, UserEmailRecipientBuilder and User.")

### The code

Let’s start with our User object. Each user keeps track of their email address, their supervisor and their coworkers.

```java
public class User {

    private List<User> coworkers;
    private User supervisor;
    private InternetAddress emailAddress;

    /* ... */

    public List<User> getCoworkers() {
        return coworkers;
    }

    public User getSupervisor() {
        return supervisor;
    }

    public InternetAddress getEmailAddress() {
        return emailAddress;
    }
}
```

The Email Message Builder stores all of the components that make up an email (subject, content body, recipients, etc.) until it is built into a MimeMessage (Java email object). The To: recipient and Cc: recipients are added to the builder from a single Email Recipient Provider.

```java
public class EmailMessageBuilder {

    private InternetAddress toAddress;
    private Set<InternetAddress> ccAddresses;
    /* ... */

    public EmailMessageBuilder withRecipients(EmailRecipientProvider emailRecipientProvider) {
        toAddress = emailRecipientProvider.getToAddress();
        ccAddresses = emailRecipientProvider.getCcAddresses();
        return this;
    }

    /* ... */

    public MimeMessage build(Session mimeSession) {
        /* ... */
    }
}
```

The User and Email Message Builder have no knowledge of each other, and this should be expected because they are core objects to their respective domains (user management and email management).

The Email Recipient Provider interface is self-explanatory, but we will look at it anyway:

```java
public interface EmailRecipientProvider {
    InternetAddress getToAddress();
    Set<InternetAddress> getCcAddresses();
}
```

### Emailing specific user groups

As an example, let’s consider three situations in which we want to send emails:

  1. A user has just been created. Email the user a welcome email.
  2. A user’s blog post has been published. Email the user and CC their co-workers so they can view it.
  3. It’s a user’s birthday! Email the user, and CC their co-workers and supervisor so that they can prepare a cake.

Creating individual Email Recipient Provider implementations for each of these these situations is verbose and clunky. Instead we can implement a builder that can keep track of the requirements, query a user and prepare all of the email addresses.

```java
public class UserEmailRecipientBuilder implements EmailRecipientProvider {

    private final Set<InternetAddress> ccAddresses = new HashSet<>();
    private final User user;

    public UserEmailRecipientBuilder(User user) {
        this.user = user;
    }

    public UserEmailRecipientBuilder andCcSupervisor() {
        ccAddresses.add(user.getSupervisor().getEmailAddress());
        return this;
    }

    public UserEmailRecipientBuilder andCcCoworkers() {
        user.getCoworkers().forEach(u -> ccAddresses.add(u.getEmailAddress()));
        return this;
    }

    public InternetAddress getToAddress() {
        return user.getEmailAddress();
    }

    public Set<InternetAddress> getCcAddresses() {
        return ccAddresses;
    }
}
```

This makes it easy to represent the logic for sending emails to a specific group of users:

```java
MimeMessage mimeMessage = new EmailMessageBuilder()
                .withSubject("Happy birthday!")
                .withBodyTemplate("happy-birthday.tpl")
                .withRecipients(new UserEmailRecipientBuilder(person)
                        .andCcCoworkers()
                        .andCcSupervisor())
                .build(mimeSession);
```
Easy!

(If you're wondering what I was doing in the `andCcCoworkers()` method above, look at [Java 8's upcoming support for Lambda functions](http://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)!)
