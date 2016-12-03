# Introduction to Property Rental System

## Bonus in Web System

### Toolbar in ListGroup

A toolbar was added on the bottom of each property card in the property list. Each toolbar has some buttons to provide express access to some fequently used functionalities such as update & delete properties and declare & cancel interest.

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
				<a class="list-group-item-baritem col-md-6 col-xs-6"
					href="/property/update/<%= item.id %>">Update</a>
				<a class="list-group-item-baritem col-md-6 col-xs-6" 
					style="color: #d9534f;"
					data-danger-action="/property/delete/<%= item.id %>">Delete</a>
		<% } else { %>
				<a class="list-group-item-baritem" 
					href="/property/show/<%= item.id %>">Detail</a>
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
			res.view('person/show', { person: p, list: p.properties || [] });
		});
	}
} 
```

We found that most of these controllers are working in a similar pattern:

- First, retrieve some data from a database (e.g MongoDB), then
- if the client is a browser and it wants a page, send a view filling with the data
- if the client is an mobile app or the `XMLHttpRequest`, send JSON data to the client
