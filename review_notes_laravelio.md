commit #`3ad7daacb40b7e411ad40b3aa7558696eee2e2be`

# Notes for you/your team

## Behavior

* What does it do? (business purpose)
  * Community Site for laravel.io, mostly forum and articles related to use of laravel
* Who does it do this for? 
  * External Users/Community Users
* What kind of information will it hold?
  * Laravel Community data???
* What are the different types of roles?
  * Users
  * Moderators
  * Admin
* What aspects concern your client/customer/staff the most?
  * No way to no, this is secret secret as we aren't  

## Tech Stack

* Framework & Language - Rails/Ruby, Django/Python, mux/Golang
  * PHP 8, Laravel
* 3rd party components,:
  * NPM
  * Composer - PHP Package Manager
  * Valet - Comparable to RVM
  * Oauth with Github
  * Algolia search (optional)
  * PHP Artisan - Runtime debugging
* Datastore - Postgresql, MySQL, Memcache, Redis, Mongodb, etc.
  * MySQL, but seems to be using DBAL so any backend _could_ be used
  * Redis, at least in composer.lock
  * AWS S3


## Brainstorming / Risks

* S3 is a question, how is that used?
* Out of date packages
* Redis set/get keys, see if they are user supplied
* OAuth client setup
* Authentication all runs through github, is there any bypass for registration/authentication?
* HTML Injection on pages
* Email enumeration (account enumeration) through forgot password
* Deleting an account has disastorous consequences

## Checklist of things to review

- [ ] `fruitcake/laravel-cors` - controls CORS implementation
- [ ] AWS S3 libraries, check out how those are used.
- [ ] `ramsey/uuid`
- [ ] Key storage/generation/blah
- [ ] Authorization functions are defined on a per function basis 

### Risks



## Authentication

* Authentication function checks
- [x] USER IMPERSONATION FINDING - Registration allows for takeover of other Github User's profiles!
  - [ ] Contact the dev team...
- [ ] 
- [ ] Password hashing mechanism
- [ ] Timing attacks - this could be username/password or HMAC operations verifying keys
- [ ] Forgot Password
- [ ] Password Complexity
  - Using some sort of breach/credential stuffing library
- [ ] 2 factor auth
- [ ] Enumeration... if it matters
- [ ] Signup
- [ ] Brute force attacks
- [ ] Session Management Issues
  - [ ] Session Fixation
  - [ ] Session Destruction
  - [ ] Session Length

* Is there service-to-service authentication?
  - [ ] Constant time comparison function used
  - [ ] HMAC generated using a secure algorithm (basically not SHA1/MD5)
  - [ ] Requests occur over SSL/TLS
    - [ ] Verification of SSL/TLS is not turned off
  - [ ] Reasonable TTL implemented (meaning, an hour or less would be normal.)
  - [ ] Accounts for time skew
  - [ ] Shared secret used and stored in vault (not hardcoded)
  - [ ] Unit-tests for:
    * Check fails if token/hmac/nonce/etc. is missing or mismatched
    * Failure if timestamp is missing or expired
    * Failure if signature verification fails

### Authorization
- [ ] Uses @login_required decorator, is it applied on all endpoints appropriately?
- [ ] Admin/Moderator access is enabled

### Auditing/Logging
- [ ] Logging configuration is in `settings.py`, check documentation for secure settings

### Injection
- [ ] ORM `where` function allows for string concatenation, search for all instances

### Cryptography
- [ ] References to base64 when handling passwords, is this bad?

### Configuration
- [ ] Code is ruby/rails, make sure and run brakeman before closing out

## Mapping / Routes

#### Routes directory
api.php
bindings.php
channels.php
console.php
web.php
- /
- /rules (VIEW)
- /terms (VIEW)
- /privacy (VIEW)
- /bin/{paste} (GET)
- articles/{article}/social.png (VIEW)
- Auth
  - login (GET/POST)
  - logout (GET)
  - password/reset (GET/POST)
  - password/reset/{token} (GET)
  - password/email (GET)
  - email/verify (GET)
  - email/verify/{id}/{hash} (GET)
  - email/resend (POST)
  - login/github (GET)
  - auth/github (GET)
- Users
  - /dashboard redirs to /user
  - user/{username} (GET)
  - notifications (VIEW)
  - settings (GET/POST) 





## Mapping / Authorization Policies

- [ ] `ArticlePolicy`
- [ ] `NotificationPolicy`
- [ ] `ReplyPolicy`
- [ ] `ThreadPolicy`
- [ ] `UserPolicy`
- Interesting Authorization Middleware from _Kernel.php_

```php
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\DisableFloc::class,
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            // \Illuminate\Session\Middleware\AuthenticateSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
            \App\Http\Middleware\RedirectIfBanned::class,
        ],

        'api' => [
            'throttle:api',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
    ];
```

## Mapping / Files
