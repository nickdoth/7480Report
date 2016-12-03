# Introduction to Property Rental System

## Bonus in Web System

### Toolbar in ListGroup

A toolbar was added on the bottom of each property card in the property list. Each toolbar has some buttons to provide express access to some fequently used functionalities such as update and delete properties.

Since Bootstrap does not support putting two or more `list-group-items` into one single line, we implement this feature.

Firest we add some styles in `assets/styles/importer.less`:
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

![ListGroupToolbar](https://ob22ak52h.qnssl.com/ListGroupToolbar.png)
