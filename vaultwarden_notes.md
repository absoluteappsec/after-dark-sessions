# Secure Code Review - VaultWarden

[Code Repository](https://github.com/dani-garcia/vaultwarden)
We assessed commit `#06f8e69c7026d4aacc15bbbe0a87377e131336db`

# Findings
Nothing so far.

## 1. SQL Injection


---

# Notes for you/your team

## Behavior

* What does it do? (business purpose)
  * Open source implementation of Bitwarden (bitwarden.com), rewritten in Rust
* Who does it do this for? (internal / external customer base)
  * External Users
* What kind of information will it hold?
  * Passwords, User Data, TOTP, Email, Name, Priv/Pub Key, Password Hints (for users)
  * 2fa supported types: Yubikey, Authenticator, Duo, Yubikey, OrganizationDuo, Webauthn
* What are the different types of roles?
  * User have user_collections
  * Organizations have collections
  * Owner, Admin, User, Manager
* What aspects concern your client/customer/staff the most?
  * Loss of secrets, passwords, any data

## Tech Stack

* Framework & Language
  Rocket 0.5.0-rc.1 / Rust 1.59
  Template lang = handlebars
* 3rd party components, Examples:
  Encoding libraries, several
  Token libraries: jsonwebtoken, yubico, totp-lite, webauthn-rs, u2f
* Datastore - Postgresql, MySQL, SQLite
 Diesel ORM + Query Builder

## General information

* The application will go ahead and generate JWTs and [then store them inside of cookies](https://github.com/dani-garcia/vaultwarden/blob/42136a70973f60086749c62439c6a965d4589c02/src/api/admin.rs#L186-L194)

## Brainstorming / Risks

* Mass Assignment - All Access
* It's a password vault, how are secrets and values being stored in the database? Are encryption keys being used properly?
* Account takeovers/AuthN
* Randomization and token generation
* JSON Web Tokens - Is crypto algorithm defined?
* API section handles most functions, retrieving secrets without permissions.
* Re-invite users in bulk, abuse patterns here?
* Any ability to forge the contents of a JWT - Auth.rs "stuff" is based on the validity of the sessions.
* Look at how UUIDs are generated
* securitystamp is

## Checklist of things to review

- [ ] From the [Rust Handlebar Documentation](https://docs.rs/handlebars/latest/handlebars/), `no_escape` is the primary way to go about not escapign templates
  * I noticed that in the handlebars spec usually {{{ }}} means raw HTML
  * [This issue is interesting](https://github.com/sunng87/handlebars-rust/issues/393) because it talks about escaping in various contexts 


### Risks
- [ ] Verify that OrgHeaders does not have a weighting/ranking/confusion issues because it takes the UUID from both the query string and parameters.

### Authentication
- [ ] Login page give error messages, check for enumeration
- [ ] Signup page allows for freeform passwords, does it implement proper password complexity?

### Authorization
- [ ] Uses @login_required decorator, is it applied on all endpoints appropriately?

### Auditing/Logging
- [ ] Logging configuration is in `settings.py`, check documentation for secure settings

### Injection
- [ ] ORM `where` function allows for string concatenation, search for all instances

### Cryptography
- [ ] References to base64 when handling passwords, is this bad?

### Configuration
- [ ] Code is ruby/rails, make sure and run brakeman before closing out

## Mapping / Routes
* The pattern is - as an example - `/admin` goes to admin.rs - admin.rs has a list of functions in the routes array, and the functions define the paths inside themselves
* Rocket Routes use the following patterns `#[get("/foo")]`
* `/` - web_routes
  * `/` - web_index
  * `/app-id.json` - app_id
  * `/<p..>` - web_files
  * `/attachments/<uuid>/<file_id>` - attachments
  * `/alive` - alive
  * `/vw_static/<filename>` - static_files
* `/api/` - core_routes
  * `/accounts/verify-password` - verify_password
  * `/accounts/password-hint`
  * `/accounts/api-key`
  * `/accounts/rotate-api-key`
  * `/accounts/verify-email-token`
  * `/accounts/verify-email`
  * `/accounts/email-token`
  * `/accounts/security-stamp`
  * `/accounts/key`
  * `/accounts/password`
  * `/sync` - ciphers_sync
  * `/ciphers`
  * `/emergency-access/trusted`
  * `/emergency-access/granted`
  * `/emergency-access/<emer_id>`
  * `/emergency-access/invite`
- [ ] `/admin` - admin_routes
  * [ADMIN_TOKEN](https://github.com/dani-garcia/vaultwarden/blob/7c89bc619a55debf3fa7b4ff7f5c53d0a1125a85/.env.template#L237-L238) is placed in the .env file and therefore is statically set - even if changed this could be for sure problematic
* `/identity` - identity_routes
* `/icons` - icons_routes
* `/notifications` - notification_routes

## Mapping / Authorization Decorators

- [ ] `OwnerHeaders`
 * Authorization takes two things into account - do you belong to the org you're requesting to do something against (calls OrgHeaders) and checks that you're an owner (assuming the first check succeeds). Both use JWTs
- [ ] `AdminHeaders`
  * Calls OrgHeaders
  * Then checks if you are an org Admin
- [ ] `ManagerHeadersLoose`
  * OrgHeaders check
  * Just have to be a manager or above (admin/owner)
- [ ] `ManagerHeaders`
  * Invokes org headers
  * Checks that you have a manager user type or above (admin, owner)
  * If collection ID passed, then it'll do a verification that you have "full_access" but will not verify your user belongs to that collection
  * Collection enumeration
- [ ] `OrgHeaders`
  * So it takes a UUID from a request but it does this first from the path name and second from the query string as annotated in [this comment](https://github.com/dani-garcia/vaultwarden/blob/19b8388950e5f97703ed21c9e4cd47b303e3db81/src/auth.rs#L411-L414)
  * One issue is that the [get_org_id](https://github.com/dani-garcia/vaultwarden/blob/19b8388950e5f97703ed21c9e4cd47b303e3db81/src/auth.rs#L414-L415) function may say the path is invalid, default to the query string, and then later at a defined route endpoint they could be pulling the UUID from the path, formatting it differently, and it not being considered an invalid UUID at parse time which could lead to insecure direct object reference.
  * Interesting errors to look at
- [ ] `ClientIp`
  * Just gets an IP but only if its configured to retrieve IPs from headers. Important from logging perspective.
- [ ] `Headers`
  - Coarse grained - basically validates JWT is valid through timestamp/securitystamp
  - This appears to be doing route matching based on a security stamp exception - we're guessing the exception occurs because someone has been temporarily granted access to an org resource - we say this because its used by orgheaders


## Mapping / Important Files

- [ ] `auth.rs`

https://thissite.com/organization/8428348?org_id=1234
