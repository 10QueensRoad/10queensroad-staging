---
layout: post
title: "Under the hood of Guava Lists Transformation"
category: java
tags: [java, guava, functional-programming]
author: "Bin Chen"
---

{% include JB/setup %}

Guava is a Java library provided by Google, which contains a lot of helpful utility functions. In our development team, two of the very useful functions we use daily are [`Lists.transform`](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Lists.html) and [`Collections2.transform`](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Collections2.html).

Recently I encountered some unexpected behaviour when I was using the transform operation. To simplify the problem, let's say I have a class `UploadData` which represents some data uploaded by the end user. The user can also specify an effective date to indicate when the data will be effective.

<!--end excerpt-->

```java
    class UploadData {
	
      public Data getData() { /* ... */ }
      public Date getEffectiveDate() { /* ... */ }
	
      /* ... */
    }
```

And in the web page, there is a drop-down box that lists all the `UploadData` objects. Each entry in the drop-down box shows the effective date of the `UploadData`, ordered chronologically from the newest to the oldest. The record that is currently effective should be selected by default.

For example, let's say that there are three records of upload data, with the following effective dates:

+ January 1, 2013
+ March 8, 2013
+ December 15, 2013

Today is October 6, 2013, so the UI should list them as below - 

+ December 15, 2013
+ **March 8, 2013** (selected, as it is currently effective)
+ January 1, 2013

There is a data transformation class created for this case.

```java
	class UploadDataDto {
	 
	  private final boolean currentlyEffective;
	  private final Date effectiveDate;
	
	  public UploadDataDto(boolean currentlyEffective, Date effectiveDate) {
	    this.currentlyEffective = currentlyEffective;
	    this.effectiveDate = effectiveDate;
	  }
	
	  public boolean isCurrentlyEffective() {
	    return currentlyEffective;
	  }
	
	  public Date getEffectiveDate() {
	    return effectiveDate;
	  }
	}
```

In the controller layer, we created a method to get the `UploadData` from a repository and transform them to `UploadDataDto` objects. To do this, we are using the Guava API to apply the transformation.

```java
	public List<UploadDataDto> listUploads() {
	
	  // get a list of uploads ordered by effective date in descending order
	  List<UploadData> uploads = uploadRepository.listOrderByEffectiveDateDesc();
	
	  final Date now = new Date();
	  return Lists.transform(uploads,
	               new Function<UploadData, UploadDataDto>() {
	
	      boolean foundCurrentlyEffective = false;
	
	      @Override
	      public UploadDataDto apply(UploadData upload) {
	        if (!foundCurrentlyEffective &&
	            now.after(upload.getEffectiveDate())) {
	          foundCurrentlyEffective = true;
	          return new UploadDataDto(true, upload.getEffectiveDate());
	        }
	        return new UploadDataDto(false, upload.getEffectiveDate());
	      }
	    });
	}
```

As you can see, we passed in an instance of a `Function` to the `Lists.transform` function. This anonymous function takes an `UploadData` and converts into a `UploadDataDto` object. A boolean flag inside the `Function` instance is used to bookmark whether we have found the currently effective data.

The transformation works as expected and we can see the expected options listed on the web page. So far so good?

A few days later, I tried to convert an existing Java unit test into Scala Specs2 test. Surprisingly I found that a few cases failed after the conversion. After more than 90 minutes of debugging with the help from my colleagues, I realized the problem was caused by the `List` returned by `Lists.transform` function:

+ The returned `List` is a wrapper of the original list.
+ Any invocation on the returned list will invoke the passed in `Function`

Below is the code snapshoot of the `transform` function. As you can see in the [Lists class source code](https://code.google.com/p/guava-libraries/source/browse/guava/src/com/google/common/collect/Lists.java), both the `TransformingRandomAccessList` and `TransformingSequentialList` are wrappers of the original list. They keep a reference to the `Function` object, so they don’t evaluate the function until the client calls to loop over or access element from the result list.


The Lists.transform function:

```java
	public static <F, T> List<T> transform(
	    List<F> fromList, Function<? super F, ? extends T> function) {
	  return (fromList instanceof RandomAccess)
	    ? new TransformingRandomAccessList<F, T>(fromList, function)
	    : new TransformingSequentialList<F, T>(fromList, function);
	}
```

That being said, the returned `List` is lazy, not cached. In this scenario, the root cause of the issue is the boolean flag `foundCurrentlyEffective` in the `Function` instance, which introduced the side effect. If this list is only looped through one time, everything is fine. But as soon as it's looped second time, it won't flag the currently effective upload any more… Unfortunately in my Scala test, I used an implicit conversion to convert the `List` into a Scala `Seq` before I made any assertions on the elements of the `List` (`Seq`).

The conculsion is that we should provide a pure function (without any side effect) to the `Lists.transform` method, as it would help to avoid this kind of issue.