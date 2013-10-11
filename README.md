<h1>Callback Basics</h1>

<h2>What's a callback?</h2>
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


<h2>The problem with synchronous (linear) code in javaScript</h2>
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

<h2>The right way: use callbacks</h2>

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

<h2>Callbacks are are great for event-driven design</h2>

jQuery uses a lot of callbacks to connect a function with an event. Take this simple example:
```javascript
$('button').on('click', function()
{
	alert("The button was clicked!");
});
```
In the above example, every time the user clicks a button on the web page, a message will be shown. The first parameter to the <b>on</b> function is a string, the event name, and the second parameter is a callback function to execute when that event occurs.

<h1>Async Program Flow</h1>

<h2>Linear Operations with Nested Callbacks</h2>
Again, let's take a simplified Java server-side code block that calls multiple web services from some API.
```java
Integer userID = getActiveUser();
Array friendsArray = getFriendsOfUser(userID);
System.out.println("The user has " + friendsArray.length + " friends.");
```

This is how that example would look with callbacks in JavaScript:
```javascript
getActiveUser(function(userID)
{
	getFriendsOfUser(userID, function(friendsArray){
		console.log("The user has " + friendsArray.length + " friends.");
	});
});
```

In both examples above:
The function <b>getActiveUser</b> is executed first. When that function returns the user's ID, the function <b>getFriendsOfUser</b> is executed. Then when that function returns the array of friends, the number of friends is displayed.

Let's make it slightly more complicated by adding one more function...

In Java:
```java
Integer userID = getActiveUser();
Array friendsArray = getFriendsOfUser(userID);
Boolean sent = sendMessageToUsers(userID, friendsArray, "Hello");
System.out.println("The messages were sent: " + sent);
```

In JavaScript:
```javascript
getActiveUser(function(userID)
{
	getFriendsOfUser(userID, function(friendsArray){
		sendMessageToUsers(userID, friendsArray, "Hello", function(sent)
		{
			console.log("The messages were sent: " + sent);
		});
	});
});
```

<h2>Async Functions in For Loops</h2>
Let's say we need to make multiple calls to a web service, and then do something afterwards. Assume for this example that we have two user IDs and a function <b>sendMessage</b> that sends a message to a single user.

To send a message to those users, we could write our JavaScript like this:
```javascript
sendMessage(userID_A, "Hello", function()
{
	sendMessage(userID_B, "Hello", function()
	{
		console.log("sent messages");
	});
});
```

This is a very clean way to structure our code. It will send "Hello" to userA first, then send a message "Hello" to userB and finally show "sent messages."

But what if you had to send messages to 5 users? or 10? or 100? Should we still use nested callbacks like this? No - that would be impossible to read.

I see a lot of developers try to write something like this:
```javascript
var userIDs = [....]; //an array of X length

for(var i=0; i<userIDs.length; i++)
{
	var userID = userIDs[i];
	sendMessage(userID_B, "Hello", function()
	{
		console.log("sent one message");
	});
}

console.log("sent messages");
```

Here's what will happen at runtime:
<ol>
<li>X requests will be made to the <b>sendMessage</b> web service</li>
<li>The console will log "sent messages"</li>
<li>X responses will return (at some time), followed by the log "sent one message"</li>
</ol>

Notice what's happening here: the final log is executing after the requests to the web service have been made, NOT after the responses have returned. We want all X responses to return and then to log "sent messages." Here's how:

A simple solution to this is creating one more function and a counter equal to the number of asyncronous calls:
```javascript
var messagesToSend = userIDs.length;
function sentMessage()
{
	messagesToSend--;
	if(messagesToSend == 0)
	{
		//all web service responses have been received!
		console.log("sent messages");
	}
}
```

This function will be called every time <b>sendMessage</b> executes it's callback. It will only execute the final log, after it has been called X times.

Here's the correct way to structure this code:
```javascript
var userIDs = [....]; //an array of unknown or large size
var messagesToSend = userIDs.length;

for(var i=0; i<userIDs.length; i++)
{
	var userID = userIDs[i];
	sendMessage(userID_B, "Hello", sentMessage);
}

function sentMessage()
{
	console.log("sent one message");
	
	messagesToSend--;
	if(messagesToSend == 0)
	{
		//all web service responses have been received!
		console.log("sent messages");
	}
}
```

<h2>A more complicated example</h2>
Building slightly on the previous example, let's say we want to call another async function afterwards to create a log.

To keep things organized, we can wrap the previous example in it's own function, which accepts the userIDs array and a callback:
```javascript
function sendMessages(userIDs, sentMessagesCallback)
{
	var messagesToSend = userIDs.length;

	for(var i=0; i<userIDs.length; i++)
	{
		var userID = userIDs[i];
		sendMessage(userID_B, "Hello", sentMessage);
	}

	function sentMessage()
	{
		console.log("sent one message");
		
		messagesToSend--;
		if(messagesToSend == 0)
		{
			//all web service responses have been received!
			sentMessagesCallback();
		}
	}
}
```

And then treat this like any other asynchronous function, like so:
```javascript
var userIDs = [...]; //an array of X user IDs

sendMessages(userIDs, function()
{
	createLog(function()
	{
		console.log("all messages have been sent and a log has been created!");
	});
});
```
