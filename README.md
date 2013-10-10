Lessons In JavaScript
===================

<h2>Lesson 1: Callback Basics</h2>

<h3>What's a callback?</h3>
A callback is nothing more than a function that you pass to another function as an argument or parameter.

Let's look at a very simple example of using a callback design pattern.

The following function takes two numbers as parameters and returns the sum:
```javascript
function sum(numberOne, numberTwo)
{
	var value = numberOne + numberTwo;
	return value;
}

var value = sum(2, 5); //value is 7
```

You can rewrite that function using the callback design pattern:
```javascript
function sum(numberOne, numberTwo, callbackFunction)
{
	var value = numberOne + numberTwo;
	callbackFunction(value);
}

var value = null;

sum(2, 5, function(result)
{
	//result is 7
	value = result;
});
```

See the difference? The two pieces of code above do exactly the same thing, but the second uses a callback. Using a callback for this example is obviously not needed, but that's not the point.


<h3>The problem with synchronous (linear) code in javaScript</h3>
A lot of Java developers have trouble grasping the asyncronous nature of JavaScript. In Java server-side code, to retrieve data from a web service you might do something like this:
```java
String xmlResponse = getDataFromWebService();
System.out.println(xmlResponse);
```

If we implement our code this way in JavaScript, the browser will be hung while we are waiting for data to be returned from the web service.

Why? Because the browser will not update the screen (the DOM elements) while JavaScript is running.

Think about that: nothing will update on the UI until your JavaScript has finished executing!

Take this example in JavaScript:

```javascript
function updateView(text)
{
	$('#output').text(text);
}

updateView('Loading...');
var xmlData = getDataFromWebService();
updateView(xmlData);
```

We want to show "Loading..." on the screen, and then replace that with the response from the web service. 

Unfortunately, "Loading..." will not be rendered on the screen until the JavaScript has finished executing. If the web service takes 2 minutes to return, the user will be stuck looking at a blank screen for 2 minutes!

Here's what actually happens:
<ol>
	<li>updateView is called</li>
	<li>The browser adds an event to it's queue to update the UI</li>
	<li>getDataFromWebService is called</li>
	<li>A request is sent to the web service</li>
	<li>The web service returns a response</li>
	<li>The value of xmlData is set to the xml response</li>
	<li>updateView is called</li>
	<li>The browser adds an event to it's queue to update the UI</li>
	<li>The browser updates the view with "Loading..."</li>
	<li>The browser updates the view with the value of xmlData</li>
</ol>

The user will see a white screen for 2 minutes, then the xml. The user will never see "Loading..."

<h3>The right way: use callbacks</h3>

Take a look at the above example re-written using a callback design pattern:

```javascript
function updateView(text)
{
	$('#output').text(text);
}

updateView('Loading...');
getDataFromWebService(function(xmlData) //create an anonymous function to pass in as a parameter
{
	updateView(xmlData);
});
```

In the above example, we assume that the function <b>getDataFromWebService</b> takes one parameter, a function. When <b>getDataFromWebService</b> has retrieved the web service data, it will execute the function, passing the xml data to it.

The above example was implemented using an anonymous function (a function that has no name), which you'll often see in JavaScript code.

Another way to write this example:

```javascript
function updateView(text)
{
	$('#output').text(text);
}

updateView('Loading...');
getDataFromWebService(updateView); //pass in the callback function as a parameter
```

<h3>Callbacks are are great for event-driven design</h3>

jQuery uses a lot of callbacks to connect a function with an event. Take this simple example:
```javascript
$('button').on('click', function()
{
	alert("The button was clicked!");
});
```
In the above example, every time the user clicks a button on the web page, a message will be shown. The first parameter to the <b>on</b> function is a string, the event name, and the second parameter is a callback function to execute when that event occurs.
