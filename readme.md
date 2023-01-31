# ConnectWiseHostedApi Client API

The ConnectWiseHostedApi Client API is a JavaScript script hosting of your own web application in an iframe in ConnectWise in either a pod on the screen or a tab for a specific screen or a custom menu entry. Your application will receive messages baed on various ConnectWiseHostedApi events.

If instead you want to embed ConnectWise in your web application see the VendorHosted client api.

If instead you want your web application to be its own screen in ConnectWise see the Custom Menu Entry.

If instead you want to add one or more custom fields or mark non required fields required see the Custom Field feature.

##  The ConnectWiseHostedApi Client API and the ConnectWise API
In many cases the ConnectWiseHostedApi API will be used in conjunction with the ConnectWise API. 

An example workflow would be:

* Create the setup table entry for your web application and indicate the following settings:
	* The Origin of your web application.
	* What screen(s) you want it visible on
	* If you want it hosted as a Pod and/or Tab.
	* The title of your hosted web application.
	* The url of your hosted web application.
	Note that you can also us the ConnectWise API to create setup table entry.
* In your web application include the ConnectWiseHostedApiClientApi javascript file.
* Instantiate the ConnectWiseHostedApi script instance.
* Register for the events you are interested in receiving when instantiating.
* When the indicated screen(s) loads or is refreshed your web application will receive a message with the screen and id that was loaded.
* Use the ConnectWise API to retrieve the screen entity information.
* When the user saves the screen you will be notified and have the option to cancel the the save.
* When the user attempts to leave the screen you will be notified and can report yourself as 'dirty' prompting a confirmation.

##  Downloading the ConnectWiseHostedApi Client API
[Client API](https://developer.connectwise.com/Hosted_APIs)
The ConnectWiseHostedApi Client API consists of the *ConnectWiseHostedApiClientApi.X.x.js* JavaScript script file and a sample Javascript web application.

###  Versioning
The version of the ConnectWiseHostedApi Client API being used is appended to the JavaScript file name in the format of *X.x*. The ConnectWiseHostedApi Client API will maintain backwards capability within the same minor versions but might break compatibility with major version updates.

If the version of the Client API is incompatible with the embedded instance of ConnectWise an error will be returned when an api call is made.

##  Setting up your web application
Place a script element in your web application with the *src* attribute pointing to where you are hosting the ConnectWiseHostedApi.js file.
```
<script src='ConnectWiseHostedAPI.1.0.js'></script>
```
In your JavaScript instantiate an instance of the ConnectWiseHostedApi. See the API specification below for further information on the constructor arguments for the ConnectWiseHostedApi.
```
// instantiate ConnectWiseHostedApi passing in the ConnectWise Origin domain and a collection of event handlers.
var connectWiseClientApi = new ConnectWiseHostedApi("https://my.connectwise.com",
	{
		"eventHandlers" :[
			{"event": "onLoad", "callback" : function},
			{"event": "beforeSave", "callback" : function},
			{"event": "beforeBack", "callback" : function}
		]
	}
);
```
# ConnectWiseHostedApi Client API Specification
Please note your web application must be hosted by a web server such as IIS or Apache. The ConnectWiseHostedApi Client API will not work directly off the hard drive.

## ConnectWiseHostedApi Instantiation
Before requests can be made of the ConnectWiseHostedApi it must be instantiated.

When setting up your web application in the ConnectWise setup table, you must set the hosting application's *Origin* domain setting. You will need security access to access this table. Please contact your ConnectWise Administrator if you do not have access.

### Method (constructor)
```
ConnectWiseHostedApi(String origin, eventHandlers, debug)
```
### Arguments
*Origin* - The domain of the ConnectWise instance. This is needed for security reasons when passing messages between domains. See (http://www.w3.org/TR/webmessaging/#web-messaging) for more information.

**WARNING**. If you do not set the Origin in your ConnectWise Client API Setup to your hosting application domain then the Client API will not work properly. See the *ConnectWiseClientAPI Instantiation* section above.

*Debug* - [Optional] if set to true, verbose information will be logged to the console to aid in debugging issues.

### Example
```
 	connectWiseClientApi = new ConnectWiseHostedAPI('http://127.0.0.1:8888', 
					{
						"eventHandlers" :[
							{"event": "onLoad", "callback": onLoadCallback},
							{"event": "beforeSave", "callback": beforeSaveCallback}
						]
					}				
				,true);
```

## Register for Events
Most interactions with the ConnectWiseHostedAPI Client API will be by registering for events by passing the event handlers into the constructor.

### Method
```
{"event": "onLoad", "callback" : function }
```

### Arguments
*JSON object* - A JSON Object with at least a *event* parameter. The *event* parameter specifies which event to register the callback for. 

*Function callback* - A function that will be invoked when the request completes. The function is passed a JSON Object *Event*. See the *Event* section for further information on this object.

## Event
The *Event* JSON Data object passed to the callback consists of the following properties.
* onSuccess: Function to be called if the event is allowed to continue.
* onFailure: Function to be called if the event is not allowed to continue.

**WARNING** If the event passed in has an onSuccess/onFailure function these must be called for the screen to properly work.

## ScreenObject
The *ScreenObject* will be returned in the callback response of certain requests (in lower case). The *ScreenObject* consists of the following properties:

The *Screen* is a text enumeration of the below possible values:

* Ticket
* Contact
* Company
* SalesOrder
* Configurations (Not Done)
* Activity (Not Done)
* Project (Not Done)
* Opportunity (Not Done)
* CompanyFinance (Not Done)
* MyAccount (Not Done)
* Invoice (Not Done)
* Agreements (Not Done)
* PurchaseOrder (Not Done)
* ConnectWiseToday (Not Done)

The *ID* is a number representing the record id of the specific screen.

*hostedAs* indicates how your web application is being hosted in Manage. It is a text enumeration of the below possible values:

* Pod
* Tab

### Example
```
	{ "screen" : "ticket", "id" : 123445, "hostedAs" : "pod"}
```

## onLoad Event
The load event is triggered whenever the user loads or refreshes a screen.

*JSON Event*
* event: load
* screen : The Screen Enumeration that loaded
* id: The entity ID that was loaded

### Example
```
callback('{"event" : "onLoad", "args" : { "screen" : "ticket", "id": 444} })
```
## beforeSave Event
The save event is triggered whenever the user saves the screen. This event is only valid if your web application is loaded in a pod on the screen and you registered a beforeSave event handler.

*JSON Event*
* event: save
* screen : The Screen Enumeration that was saved
* id: The entity ID that was saved
* resultCallback: An object with two methods, onSuccess and onError.
	* onSuccess takes no arguments and allows the save to continue successfully.
	* onError takes a json collection of strings and stops the save event. The errors will be displayed in the message area.
	* **IMPORTANT** You must either call the onSuccess or onError method or the screen will not save.

### Example
```
callback('{"event" : "beforeSave", "args" : { "screen" : "ticket", "id": 444} , resultCallback})
```
## Dirty Flag
The dirty flag can be set to indicate that your screen has had user modifications done. If the user attempts to navigate away from the screen after the dirty flag has been set to true they will be warned that the page contains unsaved changes. The user can then choose to cancel the navigation or continue.

*JSON Event*
* request: setDirty
### Example
```
connectWiseClientApi.post({"request" : "setDirty", "args" : {"dirty" : true}});
```
## Request opening a screen
A request can be sent to the hosting ConnectWise instance to open a new screen in place or a in a new tab.

*JSON Event*
* request: openScreen
* args
	* newTab: If the new screen opens in the current tab or a new tab.
	* screen: The Screen Enumeration to load. Currently only 'ticket' and 'company' are supported.
	* id: The record ID to load.
### Example
```
connectWiseClientApi.post({"request" : "openScreen", "args" { "newTab" : true, "screen" : "ticket", "id": 444 } });
```

## Request the Member Authentication information
The current Member Authentication information can be retrieved with the following request.

*JSON request*
* request: getMemberAuthentication
* resultCallback: An object that contains the following fields.
	* The *Site* is a string of the current ConnectWise URL instance.
	* The *CompanyID* is a string of the current ConnectWise company.
	* The *MemberID* is a string of the current logged in member.
	* The *memberHash* is a string that can be used to confirm the currently logged in member with the ConnectWise API. See the ConnectWise API for further information.
	* The *memberContext* is the currently logged in context of the member and must be passed along with the memberHash when calling the ConnectWise API if using memberHash authentication.
	* The *memberEmail* is a string of the current logged in member primary email.
	
### Example
```
connectWiseClientApi.post({"request" : "getMemberAuthentication"}, function(data) { ... });
```
The callback data will be in the following format:
```
{"site":"http://my.connectwise.com/", "companyid" : "MyCompany", "memberid" : "CurrentMemberID", "memberHash" : "safddasfadfsdfsad", "memberEmail" : "my@coolemail.com" }
```

## Using URI Tokens
In some cases using the full ConnectWiseHostedAPI callback system is more then your web application needs. In that case, you can use *URI TOKENS* instead. The tokens `[cw_id]` and `[cw_screen]` will be replaced with the current screen *Record ID* and *Screen ID*.

### Example
```
https://my.embeddedwebapp.com?id=[cw_id]&screen=[cw_screen]
```
will be set as the below if loading a company screen pod with a company recid of 456.
```
https://my.embeddedwebapp.com?id=456&screen=company
```

## Request a screen refesh
**Pod Only** Requests that the current screen refresh just like the user hit the refresh button. Note the user will be prompted if their screen is in a dirty state and the user 
can then cancel the refresh

*JSON Event*
* request: refreshScreen
* args
	* None
### Example
```
connectWiseClientApi.post({"request" : "refreshScreen", "args" { } });
```
