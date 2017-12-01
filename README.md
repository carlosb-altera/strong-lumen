# strong-lumen

Lumen, but with a bunch of security-centric features

LAST AUDIT: December 2017

# Middleware

We added modern security Middleware for Lumen to ensure our APIs are a litte
more hardened than a default install.

The internet is a dangerous place, and non-security minded developers often
make mistakes that could easily be avoided.

## App ID

This is the Identifier someone needs to send through to access your application.

This allows you to add a layer of annoyance to endpoints that do not need authentication,
for example, endpoints that provide certain variables to applications, but that you do
not want to hardcode into your applications.

It's also useful for providing different configuration information from a common
endpoint based on the application in question, useful for things like white labels.

Set `APP_ID` in your .env and wrap your route in the middleware.

Example HTTP Header

```
App: 8A53A5C5-8B3D-4624-ACFA-C14945EC4F88
```

## Registration Access Key

Use this to limit access to registration endpoints to add a layer of annoyance.

This is useful for allowing endpoints for newsletter signups etc to know a key
before being able to submit.

Set `REGISTRATION_ACCESS_KEY` in your .env and wrap your route in the middleware.

Example HTTP Header

```
Registration-Access-Key: 8647032F-AFA3-4EB1-ABEA-B0A517394A2B
```

## Throttle

Allows you to set limits per route as to how many requests may happen.

This is useful for mitigating DDoS, Brute Force, and Flooding style attacks.

`'throttle:3,1'` means 3 requests per minute. `'throttle:300,1'` means 300.

Certain common routes have default throttles.

Responds with headers indicting how manuy requests are left on these routes,
and information about when bans expire.

Before Limit:

```
X-RateLimit-Limit: 10
X-RateLimit-Remaining: 5
```

After Limit you get a `429 Too Many Requests`, and the Response Body contains

```
Too many consecutive attempts. Try again in 5s
```

## Content Security Policy

Includes a sane CSP, wrapping the entire application.

Currently adds

```
Content-Security-Policy: default-src 'none', connect-src 'self', 'upgrade-insecure-requests';
```

## Common Security Headers

Includes a set of Common security headers for browsers that support them.

Usefult for defense against XSS, Clickjacking, Content Type attacks,
Framable responses, and UI Redressing.

Also enables strict TLS.

Currently adds

```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=7776000; includeSubDomains
```

## No Cache Headers

Disables caching

```
Cache-Control: no-cache, must-revalidate
```

## Server Header

Adds information about the server.

Useful for overriding and obscuring the name of the technology running the web
server.

```
Server: APP_NAME (APP_VERSION)
X-Powered-By: APP_NAME (APP_VERSION)
```

Requires APP_NAME and APP_VERSION set in the .env file.

## CORS

Adds support for Cross Origin Resource Sharing.

See `config/cors.php` for all options.

Defaults to:

```
    'supportsCredentials' => true,
    'allowedOrigins' => ['*'],
    'allowedHeaders' => ['Content-Type', 'Content-Length', 'Origin', 'X-Requested-With', 'Debug-Token', 'Registration-Access-Key', 'X-CSRF-Token', 'App', 'User-Agent', 'Authorization'],
    'allowedMethods' => ['GET', 'POST', 'PUT',  'DELETE', 'OPTIONS'],
    'exposedHeaders' => ['Authorization'],
    'maxAge' => 0,
```

Should support OPTIONS Preflight with Authorization header.

# JWT

You should use a 512 bit, asymmetrical algo, with certificates.

It is suggested that you use the `RS512` algo if you are unable to use `ES512`.

You can make a `RS512` pair like this:

```bash
# DO NOT EVER COMMIT THE PRIVATE KEY
# Make private key
openssl genrsa -out rsa512-private.pem 4096
# Make the corresponding public key
openssl rsa -pubout -in rsa512-private.pem -out rsa512-public.pem
```

Make sure you set the appropriate variables in your .env

You may use a symmetrical also but then you'll be relying on a secret instead
of a keypair.

# UUIDs

It is suggested to use UUIDs in your responses instead of IDs, which are
generally enumerable.

This will help mitigate some forms of enumeration attacks.

It is suggested that you use UUID version 4, which is random.

```php
use Webpatser\Uuid\Uuid;
UUID::generate(4);
```

# User Registration

There is augmented functionality with the user models and registrations.

An `_id` field has been added which should be used to obscure the internal
id.

You should use a UUID to protect against various attacks.

Required fields are `username`, `password`, `first_name`, `last_name`, `email`.

Additionally, users are assigned an `_id` and an `api_key` when getting created.

Confirmation codes are sent out via email

See the RegistrationController for full details.

# Soft Deletes

Soft Deletes are enabled by default.

This is important for logging, compliance, and forensics should a destructive
action be entered into the system.

# Additional Audit Fields

In addition to the usual `created_at` and `deleted_at` that come with timestamps,
we've added some additional fields to help your API be more compliant.

* `created_by`
* `deleted_by`
* `updated_by`
* `updated_at`

# System User

There is a System User that must be seeded. This user is designed to unusable,
and should be used to indicate that the system has performed an action.

Set the appropriate `created_by` and `updated_by` type fields when performing
changes in the system using the system user.

# Additional Packages

These are the additional support packages in this project

* tymon/jwt-auth - Provides JWT Functionality
* phpseclib/phpseclib - Provides additional security features and algos
* barryvdh/laravel-cors - Provides CORS functionality
* webpatser/laravel-uuid - Provides the UUID functionality

# Recommended Installs

It is suggested you configure your server with the following:

* ext-libsodium - Additional modern algos
* ext-mcrypt - Speeds up some crypto operations
* ext-gmp (GNU Multiple Precision) - Speeds up arbitrary precision integer calculations

# Other Changes

## bootstrap/app.php

* Enabled Facades
* Enabled Eloquent
* Registered Middleware
* Registered CORS and JWT Providers
* Call CORS Config
