---
layout: post
title: "Under the hood of Guava Lists Transformation"
category: java
tags: [java, guava, functional-programming]
author: "Bin Chen"
---

{% include JB/setup %}

Guava is a Java library provided by Google, which contains a lot of good utility functions. In our deveopment team, one of the very useful functions we use in daily work is [`Lists.transform`](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Lists.html) and [`Collections2.transform`](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Collections2.html).

Recently I encountered an interesting bug while I was using the transform operation. To simplify the problem, let's say I have a class `UploadData` which represents some data uploaded by the end user. And the requirement is to allow user to specify an effective date to indicate since when the data will be effective.

<!--end excerpt-->

```java
	class UploadData {
	
	  public Data getData() { /* ... */ }
	  public Date getEffectiveDate() { /* ... */ }
	
	  /* ... */
	}
```

And in the web page there's a dropdown box which lists all the uploaded data. The label is the effective date of each record and they should be ordered by time, from the latest to the oldest. Besides, the one which is currently effective should be selected by default. For example, let's say there're three records of upload data, and here're their effective dates:

+ January 1, 2013
+ March 8, 2013
+ December 15, 2013

Today is October 6, 2013, so the UI should list them as below - 

+ December 15, 2013
+ March 8, 2013 (**selected** as it is currently effective)
+ January 1, 2013

There is a data transformation class created for this case.

```java
	class UploadDataDto {
	 
	  private final boolean currentlyEffective;
	  private Date effectiveDate;
	
	  public UploadDataDto(boolean isCurrentlyEffective, Date effectiveDate) {
	    this.currentlyEffective = isCurrentlyEffective;
	    this.effectiveDate = effectiveDate;
	  }
	
	  public boolean isCurrentlyEffective() {
	    return currentlyEffective;
	  }
	
	  public Date effectiveDate() {
	    return effectiveDate;
	  }
	}
```

In MVC layer we created a method to get the `UploadData` from repository and transform them to `UploadDataDto` objects. And we use the Guava API to apply the transformation.

```java
	public List<UploadDataDto> listUploads() {
	
	  // get a list of uploads ordered by effective date descdently
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

As you can see, we passed in an instance of anonymous `Function` to the `transform` function. This ananomous function takes a `UploadData` and converts into a `UploadDataDto` object. Besides, a boolean flag is used to bookmark whether we have found the currently effective data.

The transformation works as expected and we saw the expected options listed in the web page. So far so good?

After a few days, I tried to convert an existing Java unit test into Scala Specs2 test. Suprisingly I found that a few cases failed after the conversion. After more than 90 minutes debug with the help from my colleagues, I realized the problem is caused by the `List` returned by `Lists.transform` function:

1. The returned `List` is a wrapper of the original list.
2. Any invocation on the returned list will invoke the passed in `Function`

Below is the code snapshoot of the `transform` function. As you can see in the [source code](https://code.google.com/p/guava-libraries/source/browse/guava/src/com/google/common/collect/Lists.java), both the `TransformingRandomAccessList` and `TransformingSequentialList` are wrappers of the original list. They keep the reference to the `Function` object, therefore they won't evaluate the result list until client calls to loop over or access element from the result list.

```java
	public static <F, T> List<T> transform(
	    List<F> fromList, Function<? super F, ? extends T> function) {
	  return (fromList instanceof RandomAccess)
	    ? new TransformingRandomAccessList<F, T>(fromList, function)
	    : new TransformingSequentialList<F, T>(fromList, function);
	}
```

That being said, the returned `List` is lazy, and not cached. In this scenario, the root cause of the bug is the boolean flag `foundCurrentlyEffective` in the `Function` instance, which introduced the side effect. If this list is only used one time, everything is fine. But as soon as it's looped second time, it won't flag the currently effective upload any more… Unfortunately in my Scala test, I used an implicit conversion to convert the `List` into a Scala `Seq` before any assertion on the elements of the `List` (`Seq`).

The conculsion is that, try to provide a pure function (without any side effect) to the `Lists.transform` method. It would help to avoid this kind of subtle bugs.

