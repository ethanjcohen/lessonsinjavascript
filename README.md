Lessons In JavaScript
===================

<h2>Lesson 1: Callbacks</h2>

A lot of Java developers have trouble grasping the asyncronous nature of JavaScript. In Java server-side code, to retrieve data from a web service you might do something like this:
```java
String xmlResponse = getDataFromWebService();
System.out.println(xmlResponse);
```

<h3>The problem with asynchronous code in javaScript</h3>
If we implement our code this way in JavaScript, the browser will be hung while we are waiting for data to be returned from the web service.

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

<b>But JavaScript runs in a SINGLE thread!</b>

That means that the browser uses only one thread to both execute JavaScript and render content on the screen - "Loading..." will not be rendered on the screen until the JavaScript has finished executing. If the web service takes 2 minutes to return, the user will be stuck looking at a blank screen for 2 minutes!

<h3>The right way: use callbacks</h3>

A callback is nothing more than a function that you pass to another function as an argument or parameter.

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

<h3>One more time!</h3>
Let's look at another example of using a callback.

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

See the difference? The two pieces of code above do exactly the same thing, but one uses a callback. Using a callback for this example is obviously not needed, but that's not the point.

<h3>Callbacks are are great for event-driven design</h3>

jQuery uses a lot of callbacks to connect a function with an event. Take this simple example:
```javascript
$('button').on('click', function()
{
	alert("The button was clicked!");
});
```
In the above example, every time the user clicks a button on the web page, a message will be shown. The first parameter to the <b>on</b> function is a string, the event name, and the second parameter is a callback function to execute when that event occurs.
