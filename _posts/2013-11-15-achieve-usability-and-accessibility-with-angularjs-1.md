---
layout: post
title: "Achieve Usability and Accessibility with AngularJs(1)"
category: javascript
tags: [javascript, angularjs, usability-accessibility]
author: "Peng Xiao"
---

{% include JB/setup %}

> ####_This blog post is written with love._

We have recently been using the [Web Content Accessibility Guidelines version 2.0 (WCAG 2.0)](http://www.w3.org/TR/WCAG20/) to improve the accessibility of our web projects. In this blog post, we will show a creative way to implement the item [ARIA2: Identifying required fields with the 'aria-required' property](http://www.w3.org/TR/2013/NOTE-WCAG20-TECHS-20130905/ARIA2).

<!--end excerpt-->

The following is a typical input field (note that `<input>` in HTML5 is a [void element](http://dev.w3.org/html5/markup/syntax.html#void-elements); the closing "/" is optional):
{% highlight html %}
<label for="user-name">User Name:</label>
<input id="user-name" type="text"/>
{% endhighlight %}
And in the browser, it looks like this:
>![text input field with label](/assets/images/2013-11-15-achieve-usability-and-accessibility-with-angularjs-1/input-element-ui.png "Text input field with label")

We would like to add an asterisk to the label, if the input field is required:
>![required input field with label](/assets/images/2013-11-15-achieve-usability-and-accessibility-with-angularjs-1/input-element-required-ui.png "Required input field with label")

The HTML for this element now looks like:
{% highlight html %}
<label for="user-name">User Name:<abbr title="required" class="required">*</abbr></label>
<input id="user-name" type="text"/>
{% endhighlight %}
Implementing this feature on many required fields can take a lot of time. To simplify the process, we are going to do it dynamically with [AngularJS](http://angularjs.org).

## Implementation

### The `asterisk` Directive

Let's first create an asterisk directive.

// *HTML*
{% highlight html %}
<asterisk></asterisk>
{% endhighlight %}

// *JavaScript*
{% highlight javascript %}
// 'ua' stands for 'usability and accessibility'
var uaModule = angular.module('ua', []);

uaModule.directive('asterisk', function(){
  return {
    restrict: 'E', // only apply to Elements
    template: '<abbr title="required" class="required"">*</abbr>',
    transclude: 'true',
    replace: 'true' // replace the current element
  }
});
{% endhighlight %}

Make sure you have bootstrapped your html with:

{% highlight javascript %}
angular.element(document).ready(function () {
  angular.bootstrap(document, ['ua']);
});
{% endhighlight %}

Now whenever you add an `asterisk` element in your DOM structure, you will see a "\*" on the page.

Check out [this plnkr](http://plnkr.co/edit/kIIBicQklGuZVDblkw0Z?p=preview) for a live demo of the example above.

### The `require` Directive

`required` is [a new attribute introduced in HTML5](http://www.w3schools.com/html/html5_form_attributes.asp), and [is supported by all of the mainstream browsers](http://docs.webplatform.org/wiki/html/attributes/required). There is a Javascript workaround (['html5shiv.js'](https://code.google.com/p/html5shim/)) to fix some issues with the IE9 implementation, but I have not verified how effectively it works. Please leave a comment if you are having trouble getting the `required` attribute to work in IE9 with `html5shiv`.

What we want to achieve is to have AngularJS add the `abbr` element to the label when it finds a `required` attribute on an `input` element.

First, add a `required` attribute to an `input` element. The code looks like:
{% highlight html %}
<label for="user-name">User Name:</label>
<input id="user-name" type="text" required/>
{% endhighlight %}

After the enhancement, it will become:
{% highlight html %}
<label for="user-name">User Name:
	<abbr title="required" class="required">*</abbr>
</label>
<input id="user-name" type="text" required/>
{% endhighlight %}
With the help of this directive:
{% highlight javascript %}
uaModule.directive('required', ['$document', '$compile', function (document, compile) {
  var linkFn, labelNode, labelElement, abbrElement;

  linkFn = function (scope, element, attributes) {
    // eliminate the dependency on jQuery
    labelNode = document[0].body.querySelector("label[for='" + attributes['id'] + "']");
    if (labelNode) {
      labelElement = angular.element(labelNode);
      // add an asterisk to the label of a required input field
      abbrElement = angular.element('<asterisk/>');
      labelElement.append(compile(abbrElement)(scope));
    }
  };

  return {
    restrict: 'A',
    link: linkFn
  };
}]);
{% endhighlight %}	

The final plnkr can be found [here](http://plnkr.co/edit/j7umgmvRg6VXUw7SC5XV?p=preview).
