---
layout: post
title: "The Importance of Ubiquitous language in DDD"
category: domain-driven-development
tags: [ddd]
author: "Anshuman Mukherjee"
---
{% include JB/setup %}

As a programmer in a team environment, one of the biggest challenges I face is naming my variables and methods. The university I attended did not think it was valuable to teach the students the importance of naming identifiers correctly. However, I have come to realise that this single aspect often determines the quality of code that we write. 

<!--end excerpt-->

The [JavaBeans specification](http://www.oracle.com/technetwork/java/javase/documentation/spec-136004.html) requires using method names like `get<propertyName>`, `set<propertyName>` and `is<propertyName>`. This itself would not be a problem if the properties had more meaningful names. However, poorly chosen property names often make it difficult to understand what we are setting or getting. As an example, a business might identify an Account having a primary user and a list of dependent users. Representing this using property names `user` and `users` breaks the mapping between code and business understanding. Badly named methods in a class further complicate this scenario.

For instance, an account accepting transfers should have a method named

```java
    acceptTransfer(amount)
```

as opposed to

```java
    add(amount)
```

I like the way method names are structured in certain languages like Objective-C. Instead of a single identifier, Objective-C allows breaking the method names into multiple identifiers.

For instance,

```objective-c
	(void)insertElement:(id)element atIndex:(NSInteger)index;
```

makes more sense than 

```java
	void insertElementAtIndex(element, index);
```

This problem is most likely to arise if the developers and domain experts use different language to describe the business operations. An example of this case is when `deposit` and `withdraw` operations are simplified to `add` and `subtract` by developers.  Such differences lead to a difference in understanding of the business requirements among team members and eventually leads to poor software.

Ideally, each member of the team (considering that domain expert is part of the team) should specify the business requirements using a ubiquitous language. Very often this language is created progressively as the understanding of the domain grows. The developers would then write code that reflects the shared understanding of this domain and the testers would test the code against it. This allows the domain experts to influence design, coding and testing. It also allows the coders and testers to discover ambiguity and discrepancy in the business requirements and clarify issues with domain experts.
