---
layout: post
title: "Importance of Ubiquitous language in DDD"
category: Domain Drived Development
tags: [ddd]
author: "Anshuman Mukherjee"
---
{% include JB/setup %}

 As a programmer, one of the biggest challange I face is naming my variables and methods. The university I attended did not think it was important to teach the students the importance of naming identifiers correctly. However I have come to realise that this single aspect often decides the quality of code that we write. 

The Javabean specification requires using method names like get<propertyName>, set<propertyName> and is<propertyName>. This itself would not be a problem if the properties were named better. However, poorly chosen property names often make it difficult to understand what we are setting or getting. For instance the business might identify an Account having a primary user and a list of dependent users. Representing this using property names 'user' and 'users' break the mapping between code and business understanding. Badly named methods in a class further complicate this scenario. For instance an account accepting transfers should have a method 
```sh
acceptTransfer(amount)
```
rather than 
```sh
add(amount)
```
I like the way method names are structed in certain languages like Objective-C. Instead of a single identifier, Objective-C allows breaking the method names into multiple identifiers. For instance 
```sh
void insertElement:element atIndex:index
```
makes more sense than 
```sh
void inserElementAtIndex(element, index)
```

This problem would mainly arise if the developers and domain experts use different language to describe the business operations. For instance withdrawal and deposit for domain expert mapping to reduce and add for developers. Such differences lead to difference in understanding of business requirements among the team members and eventually leads to poor software.

Ideally each member of the team (considering that domain expert is part of the team) should specify the business requirements using a uniquitous language. Very often this language is created progressively as the understanding of the domain grows. The developers would write code that reflects the understanding of this language and the testers would test the code against it. This allows the domain experts to influence design, coding and testing. It also allows the coders and testers to discover ambiguity and discrepancy in the business requirements and clarify them with domain experts.
