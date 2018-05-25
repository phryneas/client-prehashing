# Disclaimer

This is an early draft and published only for discussion, but not for a broader audience. Until this is published, more work needs to be done. (See roadmap)

# Roadmap

* create a first draft, get selected security experts' opinions on benefits, problems and adaptability of this approach
* create more in-depth documentation
* add examples
* create a easy-to-use javascript library (published as standalone .js for direct inclusion in the browser as well as npm module for packaging into bigger projects)
* identify server-side frameworks with big audience(e.g. FOSUserBundle for Symfony), try to get integration of the scheme upsteam or publish according bundles/modules/etc.

# Motivation

With the recent internal leaks of user passwords into log files at twitter and github, the question arises if a server should be trusted with a user's password at any given time. 
This draft suggests the use of a technique originally labeled as ["UserId-Salted-Client-Plus-Random-Salted-Server"](http://ithare.com/client-plus-server-password-hashing-as-a-potential-way-to-improve-security-against-brute-force-attacks-without-overloading-server/), which was originally discussed to reduce server load by moving parts of the password hashing to the client.
In doing so, the users' password is protected from attackers that have gained control over the host or the network between a SSL-terminating load balancer and the host. Also, the password is protected against accidental logging, which now has become a known risk.

## Summary

In the described scheme, a server publishes a public pepper, which is combined with the users' id to form a non-random salt. The password is then hashed cryptographically on the client side. The resulting hash is sent to the server, where it is combined with a per-user random salt and hashed again.

So the client sends Hash({static-pepper}+{user-id}, {password}) to the server, which calculates and stores Hash({random-salt}, Hash({static-pepper}+{user-id}, {password})).

## Pros

* the server has never any contact with the users' password, while the user can still authenticate

## Cons

* additional implementation on client-side required
* on signup or password change, password rules cannot be enforced by the server and have to be checked by the client
* in that context, checking the user password against a list of known weak passwords can add additional challenges ( although it is not impossible, see [Troy Hunt's Pwned Passwords](https://www.troyhunt.com/ive-just-launched-pwned-passwords-version-2/#cloudflareprivacyandkanonymity) )
* clients with disabled Javascript can not benefit from this scheme and supporting them adds extra logic (see next paragraph)

## Possible Problems

### Clients with disabled Javascript

In the case of disabled Javascript on the client, the curent "classic" approach of only server-side hashing can be applied. In that case, the client sends the password to the server, which identifies that the password has not been pre-hashed (e.g. by using another api endpoint for javascript-based submission) and then applies the operations that usually would be calculated on the client before aplying the default server-side operations before either storing or validating the calculated final hash.

It is important that the server detects the absence of, and executes the client-side hashing as otherwise an account with a client-side hashed password could not be logged in with disabled Javascript or vice versa. 

This increases the implementation size and complexity, as not only the client-side hashing has to be reimplemented, but also mechanisms like password rule validation (if required) need to be implemented on client side as well as a fallback counterpart on server side.

So, later versions of this draft need to contain very clear and detailed discussion of this edge case.

