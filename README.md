oauth2yii
=========

An OAuth2 client / server extension for the Yii framework.

**NOTE: CLIENT IS NOT IMPLEMENTED YET**

# Introduction

This extension is a wrapper around [OAuth 2.0 Server PHP](http://bshaffer.github.io/oauth2-server-php-docs/)
that makes it easy to add OAuth2 authentication to your application.

Before you can start using this extension you need to understand some basics about how
OAuth2 works. You can read all the details in [RFC 6749](http://tools.ietf.org/html/rfc6749)
but for a quick start we try to explain the main concepts here.

The main idea is that a *resource owner* (a.k.a. end user) hosts some data at a *server*,
and can grant permission to a *client* (a.k.a. a third party) to access this data or parts of it.
Therefore the client can obtain an *access token* which represents this permission.

So we have three roles involved:

 * **User** the end user that has a username and password at the server
 * **Server** a website where the end user has an account and stored data
 * **Client** a third party website that wants to access user data from the server.
   Clients have to register with the server first and receive a client_id and client_secret.

OAuth2 mainly defines four differnet flows for how to get an access token which are
supported by this extension.

## Authorization Code

This is the famous "Login with your FB account" type: A client site wants to authenticate its
users through another server and maybe also access the users data on that other website.

The basic flow here is:

 1. Client site redirects the user to the server site
 1. User logs in there and is asked to grant permission to the client
 1. User is redirected back to client site with an *authorization code* in the URL
 1. Client site can use this *authorization token*  to request an *access token* directly from the server
 1. Client site also receives the expiry time of the access token and optionally a *refresh token*
 1. With the *access token* the client can now access some of the user's data on the server
 1. Client can request a new access token with the refresh token

The server here has to provide two main actions:

 * **authorize**: This is a page, where the user first has to login and is then asked for permission
   to grant access to the client. This can either be a simple question like "Do you allow website
   Foo to authenticate with us?" or involve the selection of **scopes** like "Which of the following
   permissions do you want to grant to website Foo?"
 * **token**: This is the action where the client will send the authorization code to that
   it got as URL parameter when the user returend from the authorization page


## Implicit Flow (or Client-Side flow)

This is for pure Javascript applications that run in a browser. Note, that this grant type is considered
to be insecure and should be avoided. There's no client involved here.

 1. User is redirected to server site via javascript
 1. User logs in there and is asked to grant permission to the application
 1. User is redirected back to client site with an *access token* as URL parameter

The server here has to provide only one action:

 * **authorize**: This is the same page as in the step above, asking the user for permission.
   The access token will be a hash in the URL where the user is redirected afterwards.


## Resource Owner Password

If the client is a trusted entity, e.g. part of the providers enterprise then it can be
trusted to ask the user for his credentials itself.

 1. Client POSTs the user's credentials directly to the authorization provider
 1. Client recieves an *access token* in response and optionally a refresh token

The server here again has to provide only one action:

 * **token**: The client will send the username and password to this action and receive an
   access token in return.


## Client Credentials

The last grant type is used if the client has to authenticate itself agains the server
to manage it's own data. Here no user is involved and the access token only allow the
client to access his own data.

 1. Client POSTs its own credentials directly to the authorization provider
 1. Client recieves an access token in response


The server here again has to provide only one action:

 * **token**: The client will send his client_id and client_secret to this action and receive an
   access token in return.


# Installation

We recommend to install the extension with [composer](http://getcomposer.org/). Add this to
the `require` section of your `composer.json`:

    'codemix/oauth2yii' : 'dev-master'

> Note: There's no stable version yet.

You also need to include composer's autoloader on top of your `index.php`:

    require_once __DIR__.'/protected/vendor/autoload.php';

Make sure to fix the path to your composer's `vendor` directory. Finally you also need to
configure an `alias` in your `main.php`:

```
$vendor = realpath(__DIR__.'/../vendor');
return array(
    'alias' => array(
        'OAuth2Yii' => $vendor.'/codemix/oauth2yii/src/OAuth2Yii',
    ),
    ...
```

# Configuration

## Server

If you want to run an OAuth2 server you have to decide which of the above grant type or types you
want to support. You can enable them in the main application component.

### Configure Application Component

Add the application component to your `main.php` and decide, which grant types you want to enable.

```php
'components' => array(
    'oauth2' => array(
        'class'                     => 'OAuth2Yii\Component\ServerComponent',
        // Enable one or more Autho
        'enableAuthorization'       => true,
        'enableImplicit'            => true,
        'enableUserCredentials'     => true,
        'enableClientCredentials'   => true,

    ),
```

As explained above you also need to provide one or two actions. The actions required for each grant
type are:

 * Authorization code: `authorization` and `token`
 * Implicit: `authorization` and `token`
 * User credentials: `token`
 * Client credentials: `token`


### Configure the `token` action

This action is required by all grant types. It's available as an
[action class](http://www.yiiframework.com/doc/guide/1.1/en/basics.controller#action) that you
can configure in any controller you want. We recommend to create a `OauthController` and
import the action as follows.

```php
<?php
class OAuthController extends CController
{
    public function actions()
    {
        return array(
            'token' => array(
                'class' => 'OAuth2Yii\Action\Token',
                // Optional: configure the name of the server component if it's not oauth2
                //'oauth2Component' => 'oauth'
            ),
        );
    }
}
```

If you use URLs in path format the URL should then be `oauth/token`. This is the URL you
need to communicate to all clients that want to authenticate with you.


### Configure the `authorize` action

TODO
