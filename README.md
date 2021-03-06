# bPanel Ordering API

A fully RESTful server implementation for CodeIgniter using one library, one
config file and one controller.

## Requirements

1. PHP 5.4 or greater
2. Curl Library


## Order Flow

1. LLU Check (requestid)
2. Order (new,migrate)
3. Completed (orderid)


## API Information
URL : https://api.billi.be/api/v1/{function}
DATA to Post in array


## 1. LLU Check

Before ordering xDSL line on an address we would need to check xDSL avaibility on the address 
Function: checkaddress

DATA:
```array(
        'street' => 'Noordenstraat',
        'number' => '15',
        'city' => 'Diepenbeek',
        'postcode' => '3590'
  );```

Response: 
```{"status":"true","requestid":100095,"result":{"vdsl":"OK","adsl":"OK","userid":"35961","lex":"11DIE0","street":"Noordenstraat","number":"15","alpha":null,"mailbox":null,"floor":null,"block":null,"postcode":"3590","attn":"0.4010","city":null,"aup":"512.0 Kbps","vup":"4.0 Mbps","adown":"6.0 Mbps","vdown":"50.0 Mbps"}}```


## Ordering 1.1 (New)

After check above we would recieved data vdsl=>OK or adsl=OK  then we can order id using the requestid
Function: addorder

DATA: 
```array(
        'requestid' => '100095',
        'type' => 'new',
        'ppp_username' => 'up1562454@BILLI',
        'ppp_password' => '35d55xv90'
  );```


## Responses

The class provides a `response()` method that allows you to return data in the user's requested response format.

Returning any object / array / string / whatever is easy:

```php
public function index_get()
{
  $this->response($this->db->get('books')->result());
}
```

This will automatically return an `HTTP 200 OK` response. You can specify the status code in the second parameter:

```php
public function index_post()
  {
    // ...create new book
    $this->response($book, 201); // Send an HTTP 201 Created
  }
```

If you don't specify a response code, and the data you respond with `== FALSE` (an empty array or string, for instance), the response code will automatically be set to `404 Not Found`:

```php
$this->response([]); // HTTP 404 Not Found
```

## Multilingual Support

If your application uses language files to support multiple locales, `REST_Controller` will automatically parse the HTTP `Accept-Language` header and provide the language(s) in your actions. This information can be found in the `$this->response->lang` object:

```php
public function __construct()
{
  parent::__construct();

  if (is_array($this->response->lang))
  {
    $this->load->language('application', $this->response->lang[0]);
  }
  else
  {
    $this->load->language('application', $this->response->lang);
  }
}
```

## Authentication

This class also provides rudimentary support for HTTP basic authentication and/or the securer HTTP digest access authentication.

You can enable basic authentication by setting the `$config['rest_auth']` to `'basic'`. The `$config['rest_valid_logins']` directive can then be used to set the usernames and passwords able to log in to your system. The class will automatically send all the correct headers to trigger the authentication dialogue:

```php
$config['rest_valid_logins'] = ['username' => 'password', 'other_person' => 'secure123'];
```

Enabling digest auth is similarly easy. Configure your desired logins in the config file like above, and set `$config['rest_auth']` to `'digest'`. The class will automatically send out the headers to enable digest auth.

If you're tying this library into an AJAX endpoint where clients authenticate using PHP sessions then you may not like either of the digest nor basic authentication methods. In that case, you can tell the REST Library what PHP session variable to check for. If the variable exists, then the user is authorized. It will be up to your application to set that variable. You can define the variable in ``$config['auth_source']``.  Then tell the library to use a php session variable by setting ``$config['rest_auth']`` to ``session``.

All three methods of authentication can be secured further by using an IP whitelist. If you enable `$config['rest_ip_whitelist_enabled']` in your config file, you can then set a list of allowed IPs.

Any client connecting to your API will be checked against the whitelisted IP array. If they're on the list, they'll be allowed access. If not, sorry, no can do hombre. The whitelist is a comma-separated string:

```php
$config['rest_ip_whitelist'] = '123.456.789.0, 987.654.32.1';
```

Your localhost IPs (`127.0.0.1` and `0.0.0.0`) are allowed by default.

## API Keys

In addition to the authentication methods above, the `REST_Controller` class also supports the use of API keys. Enabling API keys is easy. Turn it on in your **config/rest.php** file:

```php
$config['rest_enable_keys'] = TRUE;
```

You'll need to create a new database table to store and access the keys. `REST_Controller` will automatically assume you have a table that looks like this:

```sql
CREATE TABLE `keys` (
	`id` INT(11) NOT NULL AUTO_INCREMENT,
	`key` VARCHAR(40) NOT NULL,
	`level` INT(2) NOT NULL,
	`ignore_limits` TINYINT(1) NOT NULL DEFAULT '0',
	`date_created` INT(11) NOT NULL,
	PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

The class will look for an HTTP header with the API key on each request. An invalid or missing API key will result in an `HTTP 403 Forbidden`.

By default, the HTTP will be `X-API-KEY`. This can be configured in **config/rest.php**.

```bash
$ curl -X POST -H "X-API-KEY: some_key_here" http://example.com/books
```

## Other Documentation / Tutorials

* [NetTuts: Working with RESTful Services in CodeIgniter](http://net.tutsplus.com/tutorials/php/working-with-restful-services-in-codeigniter-2/)

## Contributions

This project was originally written by Phil Sturgeon, however his involvement has shifted
as he is no longer using it.  As of 2013/11/20 further developement and support will be done by Chris Kacerguis.

Pull Requests are the best way to fix bugs or add features. I know loads of you use this, so please
contribute if you have improvements to be made and I'll keep releasing versions over time.

[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](https://raw.githubusercontent.com/chriskacerguis/codeigniter-restserver/master/LICENSE)
