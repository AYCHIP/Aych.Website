---
title: Request matchers (Caddyfile)
---

# Request Matchers

**Request matchers** can be used to filter (or classify) requests by specific criteria.

### Menu

- [Syntax](#syntax)
	- [Examples](#examples)
	- [Wildcard matchers](#wildcard-matchers)
	- [Path matchers](#path-matchers)
	- [Named matchers](#named-matchers)
- [Standard matchers](#standard-matchers)


## Syntax

In the Caddyfile, a **matcher token** immediately following the directive can limit that directive's scope. The matcher token can be one of these forms:

1. **`*`** to match all requests (wildcard; default).
2. **`/path`** start with a forward slash to match a request path.
3. **`@name`** to specify a _named matcher_.

Matcher tokens are [usually optional](/docs/caddyfile/directives#matchers). If a matcher token is omitted, it is the same as a wildcard matcher (`*`).


#### Examples

This directive applies to [all](#wildcard-matchers) HTTP requests:

```
reverse_proxy localhost:9000
```

And this is the same:

```
reverse_proxy * localhost:9000
```

But this directive applies only to requests having a [path](#path-matchers) starting with `/api/`:

```
reverse_proxy /api/* localhost:9000
```

To match on anything other than a path, define a [named matcher](#named-matchers) and refer to it using `@name`:

```
@post {
	method POST
}
reverse_proxy @post localhost:9000
```




### Wildcard matchers

The wildcard matcher `*` matches all requests, and is only needed if a matcher token is required. For example, if the first argument you want to give a directive also happens to be a path, it would look exactly like a path matcher! So you can use a wildcard matcher to disambiguate, for example:

```
root * /home/www/mysite
```

Otherwise, this matcher is not often used. It is convenient to omit it when possible; just a matter of preference.


### Path matchers

Because matching by path is so common, a single path matcher can be inlined, like so:

```
redir /old.html /new.html
```

Path matcher tokens must start with a forward slash `/`.

[Path matching](/docs/caddyfile/matchers#path) is an exact match by default; you must append a `*` for a fast prefix match. Note that `/foo*` will match `/foo` and `/foo/` as well as `/foobar`; you might actually want `/foo/*` instead.


### Named matchers

Defining a matcher with a unique name gives you more flexibility, allowing you to combine [any available matchers](#standard-matchers) into a set:

```
@name {
	...
}
```

Then you can use the matcher like so: `@name`

For example:

```
@websockets {
	header Connection *Upgrade*
	header Upgrade    websocket
}
reverse_proxy @websockets localhost:6001
```

This proxies only the requests that have a header field named "Connection" containing the word "Upgrade", and another field named "Upgrade" with a value of "websocket".

Like directives, named matcher definitions must go inside the site blocks that use them.

A named matcher definition constitutes a _matcher set_. Matchers in a set are AND'ed together; i.e. all must match. For example, if you have both a `header` and `path` matcher in the set, both must match.

For most matchers that accept multiple values, those values are OR'ed; i.e. one must match in order for the matcher to match.





## Standard matchers

Full matcher documentation can be found [in each respective matcher module's docs](/docs/json/apps/http/servers/routes/match/).

- [file](#file)
- [header](#header)
- [header_regexp](#header-regexp)
- [host](#host)
- [method](#method)
- [not](#not)
- [path](#path)
- [path_regexp](#path-regexp)
- [protocol](#protocol)
- [query](#query)
- [remote_ip](#remote-ip)


### file

```
file {
	root      <paths>
	try_files <files...>
	try_policy first_exist|smallest_size|largest_size|most_recent_modified
}
```

By files.

- `root` defines the directory in which to look for files. Default is the current working directory, or the `root` [variable](https://caddyserver.com/docs/json/apps/http/servers/errors/routes/handle/vars/) (`{http.vars.root}`) if set.
- `try_files` checks files in its list that match the try_policy.
- `try_policy` specifies how to choose a file. Default is `first_exist`.
	- `first_exist` checks for file existence. The first file that exists is selected.
	- `smallest_size` chooses the file with the smallest size.
	- `largest_size` chooses the file with the largest size.
	- `most_recent_modified` chooses the file that was most recently modified.


### header

```
header <field> <value>
```

By request header fields.

- `<field>` is the name of the HTTP header field to check.
- `<value>` is the value the field must have to match.
	- If prefixed with `*`, it performs a fast suffix match.
	- If suffixed with `*`, it performs a fast prefix match.
	- If enclosed by `*`, it performs a fast substring match.
	- Otherwise, it is a fast exact match.


### header_regexp

```
header_regexp [<name>] <field> <regexp>
```

Like `header`, but supports regular expressions. Capture groups can be accessed via placeholder like `{http.regexp.name.capture_group}` where `name` is the name of the regular expression (optional, but recommended) and `capture_group` is either the name or number of the capture group in the expression.


### host

```
host <hosts...>
```

Matches request by the `Host` header field of the request. It is not common to use this in the Caddyfile, since most site blocks already indicate hosts in the address of the site. This matcher is mostly used in site blocks that don't define specific hostnames.


### method

```
method <verbs...>
```

By the method (verb) of the HTTP request. Verbs should be uppercase, like `POST`.


### not

```
not {
	<any other matchers...>
}
```

Encloses other matchers and negates their result.


### path

```
path <paths...>
```

By request path, meaning the path component of the request's URI. Path matches are exact, but wildcards `*` may be used:

- At the end, for a prefix match (`/prefix/*`)
- At the beginning, for a suffix match (`*.suffix`)
- On both sides, for a substring match (`*/contains/*`)
- In the middle, for a globular match (`/accounts/*/info`)


### path_regexp

```
path_regexp [<name>] <regexp>
```

Like `path`, but supports regular expressions. Capture groups can be accessed via placeholder like `{http.regexp.name.capture_group}` where `name` is the name of the regular expression (optional, but recommended) and `capture_group` is either the name or number of the capture group in the expression.


### protocol

```
protocol http|https|grpc
```

By request protocol.


### query

```
query <key>=<val>...
```

By query string parameters. Should be a sequence of `key=value` pairs.


### remote_ip

```
remote_ip <ranges...>
```

By remote (client) IP address. Accepts exact IPs or CIDR ranges.
