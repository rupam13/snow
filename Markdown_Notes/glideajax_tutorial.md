# Learning ServiceNow GlideAjax: From Zero to Hero 🚀

Welcome! If you've ever felt confused by **GlideAjax** in ServiceNow, you are not alone. This guide is designed to explain it in plain, simple English (layman's terms) and walk you through three practical examples, starting from super easy to advanced.

---

## 🍔 The Layman's Metaphor: The Restaurant Analogy

Imagine you are sitting at a table in a restaurant. 

* **You (The Customer):** The **Client Script** (runs on your browser). You want information or want something done, but you can't walk into the kitchen yourself.
* **The Kitchen (The Chef):** The **Script Include** (runs on the ServiceNow server). The database lives here, and this is where the actual work/cooking happens.
* **The Waiter:** **GlideAjax**. The waiter takes your order, carries it to the kitchen, waits for the chef to cook it, and brings the food back to your table.

**Why use a Waiter (GlideAjax)?**
Without a waiter, you'd have to freeze the entire restaurant, walk into the kitchen, cook the food, and walk back. In web terms, doing things *synchronously* freezes the user's screen. GlideAjax lets the user keep typing or clicking while the server handles the request in the background (**Asynchronously**).

---

## 🛠️ The Anatomy of GlideAjax (The Blueprint)

Every GlideAjax setup requires two parts:
1. **The Server Side (Script Include):** The chef who prepares the data. *Crucial: Must be marked "Client callable".*
2. **The Client Side (Client Script):** The customer who requests the data and uses it.

Here is the basic boilerplate structure:

### 1. The Script Include (Server)
```javascript
var MyCustomServerScript = Class.create();
MyCustomServerScript.prototype = Object.extendsObject(AbstractAjaxProcessor, {

    myServerFunction: function() {
        // 1. Get the information sent by the client
        var name = this.getParameter('sysparm_user_name');
        
        // 2. Do some work (database query, calculations, etc.)
        var result = "Hello " + name + " from the server!";
        
        // 3. Return the answer
        return result;
    },

    type: 'MyCustomServerScript'
});
```

### 2. The Client Script (Client)
```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') {
        return;
    }

    // 1. Summon the waiter and tell him which kitchen script to go to
    var ga = new GlideAjax('MyCustomServerScript');
    
    // 2. Tell the waiter which function (chef) to call
    ga.addParam('sysparm_name', 'myServerFunction');
    
    // 3. Hand the waiter any extra details he needs to bring to the kitchen
    ga.addParam('sysparm_user_name', newValue);
    
    // 4. Send the waiter off and tell him who to talk to when he gets back (the callback function)
    ga.getXMLAnswer(myCallBackFunction); 
}

// 5. This runs when the waiter returns with the food!
function myCallBackFunction(response) {
    g_form.addInfoMessage(response); // Shows: "Hello [newValue] from the server!"
}
```

> [!IMPORTANT]  
> Always use `sysparm_name` to specify the server function. Any custom parameters you send must start with `sysparm_`.

---

## 📈 Practical Examples: Easy to Hard

### 🟢 Level 1: The Simple Greeting (Easy)
**Goal:** When a user changes a custom text field `u_nickname`, we send it to the server, and the server returns a formatted greeting.

#### Server-Side: Script Include
* **Name:** `GreetingAjax`
* **Client Callable:** Check/True ✅

```javascript
var GreetingAjax = Class.create();
GreetingAjax.prototype = Object.extendsObject(AbstractAjaxProcessor, {

    getGreeting: function() {
        // Read the nickname sent from the client
        var nickname = this.getParameter('sysparm_nickname');
        
        if (!nickname) {
            return "Hello Friend!";
        }
        return "Welcome back, " + nickname + "! Have a great day.";
    },

    type: 'GreetingAjax'
});
```

#### Client-Side: Client Script (onChange of `u_nickname`)
```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') {
        return;
    }

    var ga = new GlideAjax('GreetingAjax');
    ga.addParam('sysparm_name', 'getGreeting');
    ga.addParam('sysparm_nickname', newValue);
    
    // Get the XML answer asynchronously
    ga.getXMLAnswer(displayGreeting);
}

function displayGreeting(answer) {
    g_form.showFieldMsg('u_nickname', answer, 'info');
}
```

---

### 🟡 Level 2: Fetching User Details (Medium)
**Goal:** When the "Caller" field changes on an Incident form, go to the server, find the Caller's email and phone number, and auto-populate them on the form.

#### Server-Side: Script Include
* **Name:** `GetUserDetailsAjax`
* **Client Callable:** Check/True ✅

```javascript
var GetUserDetailsAjax = Class.create();
GetUserDetailsAjax.prototype = Object.extendsObject(AbstractAjaxProcessor, {

    getUserContactInfo: function() {
        var userId = this.getParameter('sysparm_user_id');
        var responseObj = {
            email: '',
            phone: ''
        };

        // Query the sys_user table
        var userGR = new GlideRecord('sys_user');
        if (userGR.get(userId)) {
            responseObj.email = userGR.getValue('email') || 'No Email';
            responseObj.phone = userGR.getValue('phone') || 'No Phone';
        }

        // We convert the object to a string so it can travel over the network
        return JSON.stringify(responseObj);
    },

    type: 'GetUserDetailsAjax'
});
```

#### Client-Side: Client Script (onChange of `caller_id`)
```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') {
        return;
    }

    var ga = new GlideAjax('GetUserDetailsAjax');
    ga.addParam('sysparm_name', 'getUserContactInfo');
    ga.addParam('sysparm_user_id', newValue);
    
    ga.getXMLAnswer(populateContactDetails);
}

function populateContactDetails(answer) {
    if (answer) {
        // Convert the string back into a Javascript Object
        var userInfo = JSON.parse(answer);
        
        g_form.setValue('u_caller_email', userInfo.email);
        g_form.setValue('u_caller_phone', userInfo.phone);
    }
}
```

---

### 🔴 Level 3: Validation & Complex Check (Hard)
**Goal:** Before submitting a change request, check if the assigned user has any open high-priority incidents. If they do, display a warning banner on the form asking the user to double-check their choice.

#### Server-Side: Script Include
* **Name:** `AssignedUserCheckAjax`
* **Client Callable:** Check/True ✅

```javascript
var AssignedUserCheckAjax = Class.create();
AssignedUserCheckAjax.prototype = Object.extendsObject(AbstractAjaxProcessor, {

    checkOpenIncidents: function() {
        var assignedTo = this.getParameter('sysparm_assigned_to');
        var result = {
            hasCriticalIncidents: false,
            incidentCount: 0,
            latestIncidentNumber: ''
        };

        if (!assignedTo) {
            return JSON.stringify(result);
        }

        var incidentGR = new GlideRecord('incident');
        // Active is true, assigned to this user, and priority is 1 (Critical) or 2 (High)
        incidentGR.addActiveQuery();
        incidentGR.addQuery('assigned_to', assignedTo);
        incidentGR.addQuery('priority', 'IN', '1,2');
        incidentGR.orderByDesc('sys_created_on');
        incidentGR.query();

        result.incidentCount = incidentGR.getRowCount();
        if (result.incidentCount > 0) {
            result.hasCriticalIncidents = true;
            if (incidentGR.next()) {
                result.latestIncidentNumber = incidentGR.getValue('number');
            }
        }

        return JSON.stringify(result);
    },

    type: 'AssignedUserCheckAjax'
});
```

#### Client-Side: Client Script (onChange of `assigned_to`)
```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') {
        g_form.clearMessages();
        return;
    }

    var ga = new GlideAjax('AssignedUserCheckAjax');
    ga.addParam('sysparm_name', 'checkOpenIncidents');
    ga.addParam('sysparm_assigned_to', newValue);
    
    ga.getXMLAnswer(handleIncidentCheckResult);
}

function handleIncidentCheckResult(answer) {
    g_form.clearMessages();
    
    if (answer) {
        var result = JSON.parse(answer);
        
        if (result.hasCriticalIncidents) {
            var msg = "⚠️ Warning: This user has " + result.incidentCount + 
                      " open High/Critical incident(s). Most recent: " + result.latestIncidentNumber;
            g_form.addErrorMessage(msg);
        }
    }
}
```

---

## 🔍 How to Debug GlideAjax

If it's not working, follow the waiter's path:
* **Step 1: Is the client sending it?** Put a `jslog('Sending ID: ' + newValue);` right before `ga.getXMLAnswer()`. Press `F12` in Chrome/Edge to view the Console.
* **Step 2: Is the server receiving it?** Put `gs.info('Server received ID: ' + userId);` inside your Script Include function. Check the ServiceNow **System Logs** (`syslog` table) to see if it prints.
* **Step 3: Is the server sending the right thing back?** Print the value right before your `return` statement in the Script Include.
* **Step 4: Is the client receiving it?** Put `jslog('Received from server: ' + answer);` inside your callback function. Check the browser console.
