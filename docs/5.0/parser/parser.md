---
layout: default
title: RFC3986 - RFC3987 Parser
---

URI Parser
=======

~~~php
<?php

use League\Uri;

class Parser
{
	public function __invoke(string $uri): array
	public function isHost(string $host): bool
	public function isScheme(string $scheme): bool
	public function isPort(mixed $port): bool
}

//function aliases

function parse(string $uri): array
function is_host(sttring $host): bool
function is_scheme(string $scheme): bool
function is_port(mixed $port): bool
~~~

## URI parsing

The `Parser::__invoke` method is a drop-in replacement to PHP's `parse_url` function, with the following differences:

### The parser is RFC3986/RFC3987 compliant

~~~php
<?php

use League\Uri\Parser;

$parser = new Parser();
var_export($parser('http://foo.com?@bar.com/'));
//returns the following array
//array(
//  'scheme' => 'http',
//  'user' => null,
//  'pass' => null,
//  'host' => 'foo.com',
//  'port' => null,
//  'path' => '',
//  'query' => '@bar.com/',
//  'fragment' => null,
//);

var_export(parse_url('http://foo.com?@bar.com/'));
//returns the following array
//array(
//  'scheme' => 'http',
//  'host' => 'bar.com',
//  'user' => 'foo.com?',
//  'path' => '/',
//);
// Depending on the PHP version
~~~

### The Parser returns all URI components.

~~~php
<?php

use League\Uri\Parser;

$parser = new Parser();
var_export($parser('http://www.example.com/'));
//returns the following array
//array(
//  'scheme' => 'http',
//  'user' => null,
//  'pass' => null,
//  'host' => 'www.example.com',
//  'port' => null,
//  'path' => '/',
//  'query' => null,
//  'fragment' => null,
//);

var_export(parse_url('http://www.example.com/'));
//returns the following array
//array(
//  'scheme' => 'http',
//  'host' => 'www.example.com',
//  'path' => '/',
//);
~~~

### No extra parameters needed

~~~php
<?php

use League\Uri\Parser;

$uri = 'http://www.example.com/';
$parser = new Parser();
$parser($uri)['query']; //returns null
parse_url($uri, PHP_URL_QUERY); //returns null
~~~

### Empty component and undefined component are not treated the same

A distinction is made between an unspecified component, which will be set to `null` and an empty component which will be equal to the empty string.

~~~php
<?php

use League\Uri\Parser;

$uri = 'http://www.example.com/?';
$parser = new Parser();
$parser($uri)['query'];         //returns ''
parse_url($uri, PHP_URL_QUERY); //returns null
~~~

### The path component is never equal to `null`

Since a URI is made of at least a path component, this component is never equal to `null`

~~~php
<?php

use League\Uri\Parser;

$uri = 'http://www.example.com?';
$parser = new Parser();
$parser($uri)['path'];         //returns ''
parse_url($uri, PHP_URL_PATH); //returns null
~~~

### The parser throws exception instead of returning `false`.

~~~php
<?php

use League\Uri\Parser;

$uri = '//example.com:toto';
$parser = new Parser();
$parser($uri);
//throw a League\Uri\Exception

parse_url($uri); //returns false
~~~

### The parser is not a validator

Just like `parse_url`, the `League\Uri\Parser` only parses and extracts from the URI string its components.

<p class="message-info">You still need to validate them against its scheme specific rules.</p>

~~~php
<?php

use League\Uri\Parser;

$uri = 'http:www.example.com';
$parser = new Parser();
var_export($parser($uri));
//returns the following array
//array(
//  'scheme' => 'http',
//  'user' => null,
//  'pass' => null,
//  'host' => null,
//  'port' => null,
//  'path' => 'www.example.com',
//  'query' => null,
//  'fragment' => null,
//);
~~~

<p class="message-warning">This invalid HTTP URI is successfully parsed.</p>

### function alias

<p class="message-info">available since version <code>1.1.0</code></p>

The library also provides a function alias to `Parser::__invoke`, `Uri\parse`:

~~~php
<?php

use League\Uri;

$components = Uri\parse('http://foo.com?@bar.com/');
//returns the following array
//array(
//  'scheme' => 'http',
//  'user' => null,
//  'pass' => null,
//  'host' => 'foo.com',
//  'port' => null,
//  'path' => '',
//  'query' => '@bar.com/',
//  'fragment' => null,
//);
~~~

## Scheme validation

<p class="message-info">available since version <code>1.2.0</code></p>

If you have a scheme **string** you can validate it against the parser. The scheme is considered to be valid if it is:

- an empty string;
- a string which follow [RFC3986 rules](https://tools.ietf.org/html/rfc3986#section-3.1);

~~~php
<?php

use League\Uri\Parser;

$parser = new Parser();
$parser->isScheme('example.com'); //returns false
$parser->isScheme('ssh+svn'); //returns true
$parser->isScheme('data');  //returns true
$parser->isScheme('data:'); //returns false
~~~

The library also provides a function alias `Uri\is_scheme`:

~~~php
<?php

use League\Uri;

Uri\is_scheme('example.com'); //returns false
Uri\is_scheme('ssh+svn'); //returns true
Uri\is_scheme('data');  //returns true
Uri\is_scheme('data:'); //returns false
~~~

## Host validation

If you have a host **string** you can validate it against the parser. The host is considered to be valid if it is:

- an empty string;
- a IPv4;
- a formatted IPv6 (with or without its zone identifier);
- a registered name;

A registered name is a [domain name](http://tools.ietf.org/html/rfc1034) subset according to [RFC1123](http://tools.ietf.org/html/rfc1123#section-2.1). As such a registered name can not, for example, contain an `_`. The registered name can be RFC3987 or RFC3986 compliant.

~~~php
<?php

use League\Uri\Parser;

$parser = new Parser();
$parser->isHost('example.com'); //returns true
$parser->isHost('/path/to/yes'); //returns false
$parser->isHost('[:]'); //returns true
$parser->isHost('[127.0.0.1]'); //returns false
~~~

The library also provides a function alias `Uri\is_host`:

~~~php
<?php

use League\Uri;

Uri\is_host('example.com'); //returns true
Uri\is_host('/path/to/yes'); //returns false
Uri\is_host('[:]'); //returns true
Uri\is_host('[127.0.0.1]'); //returns false
~~~

## Port validation

<p class="message-info">available since version <code>1.2.0</code></p>

If you have a port, you can validate it against the parser. The port is considered to be valid if it is:

- a numeric value which follow [RFC3986 rules](https://tools.ietf.org/html/rfc3986#section-3.2.3);

~~~php
<?php

use League\Uri\Parser;

$parser = new Parser();
$parser->isPort('example.com'); //returns false
$parser->isPort(888);           //returns true
$parser->isPort('23');    //returns true
$parser->isPort('data:'); //returns false
~~~

The library also provides a function alias `Uri\is_port`:

~~~php
<?php

use League\Uri;

Uri\is_port('example.com'); //returns false
Uri\is_port(888);           //returns true
Uri\is_port('23');    //returns true
Uri\is_port('data:'); //returns false
~~~
