#  HyperResource - a self-inflating client for hypermedia APIs

## About

HyperResource is a client library for hypermedia web services.  It
is usable with no configuration other than API root endpoint, but
also allows incoming data types to be extended with Ruby code.

HyperResource supports the 
<a href="http://stateless.co/hal_specification.html" target="_blank">
HAL+JSON hypermedia format</a>, with support for
other hypermedia formats planned.

## Quick Tour

Set up API connection:

```ruby
api = HyperResource.new(root: 'https://api.example.com',
                        headers: {'Accept' => 'application/vnd.example.com.v1+json'},
                        auth: {basic: ['username', 'password']})
# => #<HyperResource:0xABCD1234 @root="https://api.example.com" @href="" @namespace=nil ... >
```

Now we can get the API's root resource, the gateway to everything else
on the API.

```ruby
api.get
# => #<HyperResource:0xABCD1234 @root="https://api.example.com" @href="" @namespace=nil ... >
```

What'd we get back?

```ruby
api.deserialized_response
# => { 'message' => 'Welcome to the Example.com API',
#      'version' => 1,
#      '_links' => {
#        'self' => {'href' => '/'},
#        'users' => {'href' => '/users{?email,last_name}', 'templated' => true},
#        'forums' => {'href' => '/forums{?title}', 'templated' => true}
#      }
#    }
```

Lovely.  Let's find a user by their email.

```ruby
jdoe_user = api.users(email: "jdoe@example.com").first
# => #<HyperResource:0x12312312 ...>
```

HyperResource has performed some behind-the-scenes expansions here.

First, the `users` link was
added as a method on the `api` object at the time the resource was
loaded with `api.get`.

Then, calling `first` on the `users` link
followed the link and loaded it automatically.

Finally, calling `first` on the resource containing one set of
embedded objects -- like this one -- delegates the method to
`.objects.first`, which returns the first object in the resource.

Here are some equivalent expressions to the above.  HyperResource offers
a very short, expressive syntax as its primary interface,
but you can always fall back to explicit syntax if you like or need to.


```
api.users(email: "jdoe@example.com").first
api.get.users(email: "jdoe@example.com").first
api.get.links.users(email: "jdoe@example.com").first
api.get.links['users'].where(email: "jdoe@example.com").first
api.get.links['users'].where(email: "jdoe@example.com").get.first
api.get.links['users'].where(email: "jdoe@example.com").get.objects.first[1][0]
```

## GET, POST, PUT, and DELETE

In addition to `.get`, HyperResource provides `.create`, `.update`, and 
`.delete` methods track their HTTP counterparts.

```ruby
new_user = api.users.create(:email => 'fng@example.com',
                            :first_name => 'New',
                            :last_name => 'Guy')

new_user.first_name = 'F. New'
new_user.update

new_user.delete
```

## Fake Real World Example

Now let's put it together, with our theoretical API.
Let's say Johnny ran his mouth in the
'Politics' forum one particular day and somehow managed to piss off the
entire Internet.  Let's try and satisfy the DDoSing horde by
amending his tone.

```ruby
forum = api.forums(title: 'Politics').first
jdoe_user.comments(date: '2013-04-01', forum: forum.href).each do |comment|
  comment.text = "OMG PUPPIES!!"
  comment.update
end
```

## Extending Data Types

If an API is returning data type information as part of the response --
and it really should be -- then we can assign those data types to
ruby classes so that they can be extended.

For example, in our hypothetical Example API above, a user object is
returned with a custom media type bearing a 'type=User' modifier.  We
will extend the User class with a few convenience methods.

```ruby
class ExampleAPI < HyperResource
  class User < ExampleAPI
    def full_name
      first_name + ' ' + last_name
    end
  end
end

api.namespace = ExampleApi

user = api.users.where(email: 'jdoe@example.com').first
# => #<ExampleApi::User:0xffffffff ...>

user.full_name
# => "John Doe"
```

Don't worry if your API uses some other method to indicate resource data
type; you can override the `get_resource_data_type` method and implement your
own logic.

## Error Handling

HyperResource raises a `HyperResource::ClientError` on 4xx responses,
and `HyperResource::ServerError` on 5xx responses.  Catch one or both
(`HyperResource::ResponseError`).  The exceptions contain as much of
`cause` (internal exception which led to this one), `response`
(`Faraday::Response` object), and `deserialized_response` (the response
as a `Hash`) as is possible at the time.

## Recent Changes

* Full GET/POST/PUT/DELETE support
* Supports object caching through +Marshal.load+/+.unload+
* Refactoring to allow Adapter classes

## Authorship and License

Copyright 2013 Pete Gamache,
[pete@gamache.org](mailto:pete@gamache.org).

If you got this far, you
should probably follow me on Twitter.  [@gamache](https://twitter.com/gamache)

Released under the MIT License.  See LICENSE.txt.
