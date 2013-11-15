---
layout: post
title: "Achieve Usibility and Accessibility with AngularJs(1)"
category: javascript
tags: [javascript, angularjs, usibility-accessibility]
author: "Peng Xiao"
---

{% include JB/setup %}

> ####_This blog is written with love._

Australia government has endorsed [Web Content Accessibiity Guidelines version 2.0 (WCAG 2.0)](http://www.w3.org/TR/WCAG20/) as [a mandatory requirement for all government websites](http://webguide.gov.au/accessibility-usability/accessibility/). In this blog, we will show a creative way to implement the item [ARIA2: Identifying required fields with the aria-required property](http://www.w3.org/TR/2013/NOTE-WCAG20-TECHS-20130905/ARIA2).

The following is a typical input field(`input` is a [void element](http://dev.w3.org/html5/markup/syntax.html#void-elements). the closing "/" is optional.):
{% highlight html %}
<label for="user-name">User Name:</label>
<input id="user-name" type="text"/>
{% endhighlight %}
And in the browser, it looks like this in a browser:
>![text input with label](/assets/images/input-element-ui.png "Input element with label")

We would like to add asterisk to the label, if the input field is required:
>![required input field with label](/assets/images/input-element-required-ui.png "Required input field with label")

The HTML element looks like:
{% highlight html %}
<label for="user-name">User Name:<abbr title="required" class="required">*</abbr></label>
<input id="user-name" type="text"/>
{% endhighlight %}
There are many fields need to be updated, to make it simple, we are going to do it dynamically with [angularjs](angularjs.org).

## Implementation

### Directive `asterisk`

Let's create a aterisk directive first.

//*HTML*
{% highlight html %}
<asterisk></asterisk>
{% endhighlight %}

//*JavaScript*
{% highlight javascript %}
// ua means usability and accessibility
var uaModule = angular.module('ua', []);

uaModule.directive('asterisk', function(){
  return {
    restrict: 'E', // only apply to Element
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

Please checkout this pluckr: http://plnkr.co/edit/kIIBicQklGuZVDblkw0Z?p=preview 

### Directive `require`

`required` is [a new attribute introduced in HTML5](http://www.w3schools.com/html/html5_form_attributes.asp), and [is supported by all main stream browsers](http://docs.webplatform.org/wiki/html/attributes/required). Though there is a ['html5shiv.js'](https://code.google.com/p/html5shim/) to fix some issue on IE9, I have NOT verified it. Please leave comments, if it does not work on your IE9 with `html5shiv`.

What we want to achive is have angularjs add the required `span` when it finds a `required` attribute on the `input` element.

First, add `required` attribute to `input` element. The code looks like:
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

{% highlight javascript %}
uaModule.directive('required', ['$document', '$compile', function (document, compile) {
  var linkFn, labelNode, labelElement, abbrElement;

  linkFn = function (scope, element, attrs) {
    // eliminate the dependency on jQuery
    labelNode = document[0].body.querySelector("label[for='" + attrs['id'] + "']");
    if (labelNode) {
      labelElement = angular.element(labelNode);
      // @ add asterisk to the label of a required input field
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