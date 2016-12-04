# Introduction to Property Rental System
WU DIFEI 16428234

The Proprty Rental System is a web-based system to provide a easy way to share flat resources and look for roomates. This system also provides a moblie apps for moblie devices. This report will mainly talk about bonus features of the system.

## Bonus in Web System

### Toolbar in ListGroup

A toolbar was added on the bottom of each property card in the property list. Each toolbar has some buttons to provide express access to some frequently used functionalities such as update & delete properties and declare & cancel interest.

Since Bootstrap does not support putting two or more `list-group-items` into one single line, we implemented this feature.

First we add some styles in `assets/styles/importer.less`:
```less
.list-group-item-bar {
    padding: 0 0 !important;
}

.list-group-item-baritem {
    display: block;
    padding: 10px 15px;
    text-decoration: none;
    color: #555;
    text-align: center;
    border-right: 1px solid #ddd;

    &:last-child {
        border-right: 0;
    }

    &:hover {
        background-color: #f5f5f5;
        text-decoration: none;
    }
}
```

And then add the toolbar in `views/property/list.ejs`:
```ejs
<span class="list-group-item-bar list-group-item clearfix">
	<% if (item.owner) { %>
		<% if ((typeof item.owner === 'object' && item.owner.id === req.session.uid) ||
			item.owner === req.session.uid || 
			req.session.user_group === 'admin') { %>
				<a class="col-md-6 col-xs-6"
					href="/property/update/<%= item.id %>">Update</a>
				<a class="col-md-6 col-xs-6" 
					style="color: #d9534f;"
					data-danger-action="/property/delete/<%= item.id %>">Delete</a>
		<% } else { %>
				<a href="/property/show/<%= item.id %>">Detail</a>
		<% } %>
	<% } %>
</span>
```

And this is what it looks like:

![ListGroupToolbar](https://ob22ak52h.qnssl.com/ListGroupToolbar2.png?v2)


### Confirm Dialogs

Confirm Dialogs are necessary for some irrevocable operations, like deleting properties. However, the `confirm()` function provided by browsers will block whole execution thread of javascript, and thus the web page can do nothing else before the function returns. Therefore, **Angular-Bootstrap** is used in this project to provide non-blocking confirm dialogs.

In `list.ejs`, we add `<script>` tags for importing angular.js and angular-ui-bootstrap to our project:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.5.8/angular.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-bootstrap/2.3.0/ui-bootstrap-tpls.min.js"></script>
```

Since the project is already using bootstrap, we don't need to import it again.

After that, create controller scope for the list:
```html
<div ng-controller="PropertyList" ng-app="app">
    ...
</div>
```

Create the template of confirm dialog in the scope:
```html
<script type="text/ng-template" id="confirmModal.html">
    <div class="modal-header">
        <h4 class="modal-title" id="modal-title">{{ title }}</h4>
    </div>
    <div class="modal-body" id="modal-body">
        {{ msg }}
    </div>
    <div class="modal-footer">
        <button class="btn btn-danger" type="button" ng-click="ok()" tabindex="-1">Delete</button>
        <button class="btn btn-default" type="button" ng-click="cancel()">Cancel</button>
    </div>
</script>
```

Add `ng-click` attribute to the delete button:
```html
<a class="col-md-6 col-xs-6" 
    style="color: #d9534f;" 
    ng-click="confirm('/property/delete/<%= item.id %>', 'Delete Property')">Delete</a>
```

Finally, implement the controller:

```js
angular.module('app', ['ui.bootstrap'])
.controller('PropertyList', function($scope, $uibModal, $log, $http) {
	$scope.confirm = function(url, title, msg) {
		$uibModal.open({
			templateUrl: 'confirmModal.html',
			size: 'sm',
			controller: function($scope, $uibModalInstance) {
				$scope.title = title || 'Confirm';
				$scope.msg = msg || 'Are you sure? This can not be undone!'
				$scope.ok = function() {
					location.href = url;
					$uibModalInstance.close();
				};
				$scope.cancel = function() {
					$uibModalInstance.dismiss();
				};
			}
		})
	};
});
```

With confirm dialogs, users will not easily delete properties by mistake.

![ListGroupToolbar](https://ob22ak52h.qnssl.com/confirmModal.png)


### Sails Controllers

The web server of the project was based on **sails.js**, a easy-to-use MVC framework of node.js. A typical controler of the project would look like this:

```javascript
var PersonController = {
	show: function(req, res) {
		Person.findOne(req.params.id).exec(function(err, person)) {
			if (!person) return res.view('404', { msg: 'No such user' });
			if (!req.wantsJson) {
				res.view('person/show', { person: p, list: p.properties || [] });
			}
			else {
				res.json(person);
			}
		});
	}
} 
```

We found that most of these controllers are working in a similar pattern:

- Accept request from the client, and then
- retrieve some data from a database (e.g. MongoDB) via ORM, then
- if the client is a browser and it wants a page, send a view filling with the data
- if the client is an mobile app or the `XMLHttpRequest`, send JSON data to the client
- if error occurs, send an error message in HTML or in JSON format (depends on the client)

Thinking of this, I tried to encapsulate the common parts of controllers into a single function, so these controllers can be _described_ in a clearer way.

Now `PersonController.show` looks likes this:

```javascript
var PersonController = {
	show: ctrlInfo({
		// Accept GET request
		GET: {
			// Retrieve some data from database
			act: (req) => Person.findOne(req.params.id).populateAll().then(person => {
				if (!person) throw 'No such user';
				return person;
			}),
			// Send JSON if the client wants
			json: p => p,
			// Send HTML if the client wants
			view: p => ['person/show', { person: p, list: p.properties || [] }],
			// Send error page if error occurs
			viewError: err => ['404', { msg: err }]
		}
	})
}
```

The code of function `ctrlInfo(...)` can be found [here](https://github.com/nickdoth/sails-ctrlinfo).

## Bonus features in the mobile app

### Like/Unlike Toggle Button

Instead of showing both `Like` and `Unlike` on the toogle button at right top corner of the property card, the button will show what users will actully do when they click on it: When the interest is declared, the button's title will be 'Unlike', and vice versa. 

This means that each property card should:

- be able to access the user session data, in order to judge whether the user has declared interest on the property no not;
- be able to update state of the like button when the interest was declared or cancelled from somewhere outside the property card.

To do these two, we firstly export user session from the user controller, so that they can be accessed across the app. And then make the user controller fire an `ProfileChanged` event when the user profile changed. Finally, in the property card controller, we subscribe the `ProfieChanged` event, and change title of the interest decalration button on demand.

In `app/controllers/user.js`:

```javascript
Alloy.Globals.userSession = {
    updateUserProfile: updateUserProfile,
    toggleInterest: toggleInterest,
    isInterested: isInterested,
    state: state,
    onProfileChanged: function(listener) {
        $.getView().addEventListener('profilechanged', listener);
        return function() {
            $.getView().removeEventListener('profilechanged', listener);
        };
    }
};

function toggleInterest(id) {
    if (!state.logged) {
        alert('Please login to declare/cancel interests.');
        login();
        return;
    }
    var whiches = state.interested.map(function(interest) { return interest.which });
    var url = '', addOp = true;
    if (whiches.indexOf(id) >= 0) {
        url = Alloy.Globals.serverUrl + '/interest/delete/' + id;
        addOp = false;
    }
    else {
        url = Alloy.Globals.serverUrl + '/interest/add/' + id;
        addOp = true;
    }
    var xhr = Ti.Network.createHTTPClient();
    xhr.onload = function(e) {
        //handle response, which at minimum will be an HTTP status code
        var json = JSON.parse(this.responseText);
        updateUserProfile();
        alert(addOp ? 'Interest declared' : 'Interest deleted');
    };
    xhr.open('GET', url);
    xhr.send();
}

function isInterested(id) {
    return state.logged && state.interested.map(
        function(interest) { return interest.which }).indexOf(id) >= 0;
}

// ...other parts of the controller
```

In `app/controllers/property.js`:
```javascript
function init() {
    var user = Alloy.Globals.userSession;
    $.btn_like.title = user.isInterested($.args.uid) ? 
        'Unlike' : 'Like';

    var removeListener = user.onProfileChanged(function(ev) {
        $.btn_like.title = user.isInterested($.args.uid) ? 
            'Unlike' : 'Like';
    });
	
	$.getView().addEventListener('close', function() {
		removeListener();
		$.destroy();
	});
}

init();
```
