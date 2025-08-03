---
layout: default
title: Stuff I didn't know about RESTful APIs
---

# Background

Recently I was working and I found in our codebase that we use `patch` in place of `post` in one of the API calls. I think I didn't know that there was such method in the APIs, shame on me, well. It turns out they all serve different purposes.

# Comparison of updating methods in RESTful APIs

Most common updating methods are POST, PUT and PATCH.

| Method | Purpose | Behaviour                               | Use Case                                     | Idempotent ?  |
| :----- | :-----: | ----:                                   | --------:                                    |-----:         |
| POST   |  Create | Adds a new resource                     | Submitting a form to add a new user          | NO            | 
| PUT    |  Replace| Replaces an entire resource             | Updating a full user profile                 | YES           |
| PATCH  |  Update | Partially updates an existing resource  | Changing onmly the user's email or status    | NO            | 


# POST requests

- Used to submit an entity to a resource
- When POST request is sent, the server creates a new resource and assigns a new URL to it
- Contain request line, headers and request body (the data to be submitted is in the body in a format specified in headers Content-Type)
- request body is optional

# PUT requests

- Used to update a resource or create a new resource if it does not exist - hence the ID in the URI
- When PUT request is sent, the entire resource is replaced with the new data
- Contain request line, headers and body

> PUT request is **IDEMPOTENT**, meaning that calling the same PUT request multiple times will result in the same resource state on the server. The entire resource is updated with the new data in the request body, which means that any fields not included in the request body qill be overqritten with null or default values. 

# PATCH requests

- Used to **partially** update a resource. 
- Good to update only a few fields of a resource
- Contain request line, headers, request body (instructions how to modify the resource)

Example PATCH request:

```
PATCH /api/users/1234 HTTP/1.1
Host: example.com
Content-Type: application/json-patch+json

[
 { "op": "replace, "path": "/name", "value": "Rahul Chauahan" },
 { "op": "add", "path": "/address", "value": "123 Main St." },
 { "op": "remove", "path": "/password"}
]
```

Replaces the data of user with id "1234" with the following commans:
- replaces name
- adds address
- removes password

The `json-patch+json` indicates a set of JSON patch instructions. 

> PATCH methodis **not idempotent** - calling the same PATCH request multiple times may result inn different resource states on the server


# REST best practices

- Use POST for operations that are not CRUD, such as triggering an action on the server
- Use PUT only when you have the entire representation of the resource, including fields that are not being updated
- Use PATCH only when you need to update a portion of the resource
- Always provide appropriate response codes (e.g. 201 for succesfull POST, 200 for a succesfull PUT or PATCH)
- Ensure that resource identifier (URI) is consistent across all operations
- Consider the idempotency of the HTTP method. Idempotent methods can be called repeatedly without changing the outcome. For example, a GET request is idempotent since it always returns the same response for the same resource. On the other hand, a POST request is not idempotent since it creates a new resource every time it is called.
- Consider the safety of the HTTP method. Safe methods do not modify the resource on the server. GET and HEAD requests are safe methods, while PUT, POST, DELETE and PATCH are not.
- Consider the complexity of the resource being updated. If the resource is simple, it may be easier to use PUT to update the entire resource. However, if the resource is complex, PATCH may be a better option since it allows you to update specific fields without overwriting the entire resource.
- Consider the API's compatibility with existing HTTP implementations. Some HTTP clients or servers may not support certain HTTP methods or may have restrictions on their use. For example, some firewalls may bloack HTTP methods other than GET and POST
- Consider the security implications of the HTTP method. some may be more secure than others. For example, a GET request with sensitive information in the query string may be logged in server logs, whereas a POST request with sensitive information in the request body may be encrypted



[back](./)
