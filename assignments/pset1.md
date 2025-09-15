# Problem Set 1

## Exercise 1

### 1. Invariants
One invariant is that the item associated with a request will be the same item associated with all of the purchases from its set of purchases. Another invariant is that count associated with a request should be greater than or equal to the total counts across its set of purchases. The invariant that is more important is the second one, since this would make sure that the registry cannot sell more than what it has. This invariant affects the purchase action which preserves the invariant by decrementing the request count by exactly the purchased amount.

### 2. Fixing an action
There could potentially be a race condition due to synchronization where multiple purchases are made at the same time, each attempting to decrement the same request count. This could cause the total purchases to exceed the request count, breaking the invariant. To fix this, the purchase action should be made atomic, so that the registry checks availability and decrements the count in a single synchronized operation, preventing concurrent over-purchasing.


### 3. Inferring behavior
Yes, a registry can be opened and closed repeatedly, since the open and close actions only require the registry to exist and be in the opposite state. A reason to allow this is that people, when buying items, may temporarily close their registry while examining their items and which items were purchased already, but may want to reopen it to add more, different items or revise which items they want in general.

### 4. Registry deletion
In practice, this would likely not matter too much since registries are often associated with special occasions like weddings. Instead of deleting them, people would typically just close them. In the long-term, having an action to delete a registry may be useful if there are eventually old and unneeded registries for the purposes of data management.

### 5. Queries
Two common queries would be:

1. (Registry owner) Which items have been purchased and by whom?
2. (Giver) Which items are still available for purchase and how many?

### 6. Hiding purchases
To augment the concept specification to support hiding purchases, we could add a simple flag per registry, like purchaseVisibility. The registry owner could choose to toggle this flag on or off upon closing their registry so that when the flag is set to False, they cannot see purchases and maintain the element of surprise.

### 7. Generic types 
This is preferable to representing items with their names, descriptions, etc. since this supports abstraction/extensibility. Essentially, because User and Item are generic parameters, the concept can be reused across many potentially different contexts (while names, descriptions, etc. may change). For example, Item could have multiple different implementations like by SKU codes as mentioned or IDs.

## Exercise 2

### 1. 

**state**

- a set of Users with 
  - a username String
  - a password String

### 2.

**actions**

    register (username: String, password: String): (user: User)

        **requires** username not already registered

        **effects** creates a new user with this username and password and returns the user

    authenticate (username: String, password: String): (user: User)

        **requires** user already exists with this username and password matches

        **effects** returns the user


### 3. 
An essential invariant is that each username is unique and maps to only 1 user. This is preserved through the register action which checks for duplicated (and no other action mutates usernames).

### 4.

**state**

For the state, we add a flag as follows, such that it is now:

- a set of Users with 
  - a username String
  - a password String
  - an authenticated Flag


**actions**

    register (username: String, password: String): (user: User, token: Token)

        **requires** username not already registered

        **effects** creates a new user with this username and password, authenticated = False, and returns the user as well as a secret token (to be sent by email)

    confirm (username: String, token: Token)

        **requires** user already exists with this username and the correct token

        **effects** set authenticated = True

    authenticate (username: String, password: String): (user: User)
    
        **requires** user already exists with this username, authenticated = True, and password matches

        **effects** returns the user


## Exercise 3

### PersonalAccessToken

**concept** PersonalAccessToken

**purpose** permit users to authenticate to services without exposing their password

**principle** a user generates a random token string which can be used instead of a password to authenticate the user, and these tokens can be individually revoked without affecting other tokens or the password

**state**

- a set of Users with
  - a username String
  - a set of Tokens

- a set of Tokens with
  - a val String
  - an owner User



**actions**

    createToken (user: User): (token: Token)

      **requires** user already exists

      **effects** generate a new token with a unique val for this user (associating it with the correct user by adding it to the set of tokens for this user), and return the token

    removeToken (user: User, token: Token)

      **requires** this token already exists for this user

      **effects** remove the token from the set of tokens associated with this user

    authenticate (username: String, token: Token): (user: User)

      **requires** there is a user such that user.username is username and this token also already exists for this user

      **effects** return the user

This differs from PasswordAuthentication because tokens can be individually removed/revoked but passwords cannot. Also, instead of having one password for a user, they may have multiple different tokens. 

To improve the GitHub documentation, we could explicitly say that tokens are removable and machine-created alternatives to passwords to emphasize that they provide a level deeper of control rather than saying that we treat them exactly like passwords.


## Exercise 4

### URL Shortener

**concept** URLShortener

**purpose** provide short and unique URLs that redirect to longer target URLs

**principle** users submit long URLs, receive a truncated suffix, and then visiting the short URL redirects to the long one

**state**

- a set of Mappings with
  - a suffix String
  - a targetURL String
  - an owner User

**actions**

    truncate (owner: User, targetURL: String): (suffix: String)

      **requires** targetURL is a valid URL

      **effects** generate a unique suffix and mapping

    customTruncate (owner: User, targetURL: String, suffix: String)

      **requires** suffix not already used and targetURL is a valid URL

      **effects** create mapping with given suffix


### Conference Room Booking

**concept** RoomBooking

**purpose** manage reservations for conference rooms

**principle** users can reserve a room for a specific time or time period if it is available, and these reservations prevent conflicting bookings

**state**

- a set of Rooms with
  - an identifier String
  - a set of Reservations

- a set of Reservations with
  - a room Room
  - a startTime Datetime
  - an endTime Datetime
  - an owner User

**actions**

    reserve (room: Room, startTime: Datetime, endTime: Datetime, user: User): (res: Reservation)

      **requires** room exists and the time period (startTime to endTime) does not overlap an existing reservation

      **effects** create reservation

    cancel (res: Reservation, user: User)

      **requires** res exists and user is owner of the res

      **effects** remove reservation

### Time-Based One-Time Password (TOTP)

**concept** TOTP

**purpose** provide stronger authentication by requiring a short-lived token in addition to a password

**principle** each user shares a secret key with the server and the user's authenticator app generates time-based codes from this secret; authentication requires both the password and the current code

**state**

- a set of Users with
  - a username String
  - a password String
  - a secretKey String?

**actions**

    createUser(username: String, password: String): (user: User)

      **requires** username not already registered (i.e., username is unique)

      **effects** create a new user (with this username and password), setting user's secretKey to null and then adding to Users, and return the new user

    register (username: String, password: String): (secretKey: String)

      **requires** user exists in Users with this username and password

      **effects** generate new secretKey for user and return secretKey (to be used in the authenticator app)

    authenticate (username: String, password: String, code: String): (user: User)

      **requires** user exists with this username and password in Users, and code matches value computed from secretKey and current time
      
      **effects** return user

**notes**: I added createUser in addition to the other actions so that the concept covers both initial account creation and TOTP enrollment (so that we can handle setup for a new user).
