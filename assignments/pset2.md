# Problem Set 2

## Concept Questions

1. The contexts are for providing a namespace for nonce generation where every context has its own set of used strings (which ensures uniqueness within that context as mentioned). In the URL shortening app, a context will end up being the shortUrlBase, such that nonces are unique within each domain but can be reused across different domains.

2. NonceGeneration must store sets of used strings because it is needed to guarantee uniqueness (i.e., it is necessary to maintain a record of which strings have already been generated since otherwise duplicate strings could be created). The set of used strings in the specification is related to the counter in the implementation because then we could have some abstraction function used like this: **seen = { f(0), f(1), ..., f(counter-1) }** such that f(x) is the string generated for counter value x (and we thus map the counter-based representation to the set of used strings). 

3. One advantage of this scheme is readability for the user -- using common dictionary words makes shortenings easier to read and type (and, obviously, remember, as said). One disadvantage of this scheme is related to its scalability. If there are more generated links, then there will be increasingly complex combinations of words (in order to maintain uniqueness of the "shortened" URL) which would be harder to remember for the user and contradicts the purpose of the URL being short and memorable. To modify NonceGeneration, we can add a variable called availWordList which is a list of strings (words) available and then modify the generate action to randomly pick an unused word or sequence of words from availWordList. Then, mark this word or word combination as used. 

## Synchronization Questions

1. This is because the first sync just needs to generate a nonce based on the requested base domain (so there is no need for targetURL yet, only shortUrlBase, since we only generate a unique string in the correct namespace). The second sync, however, needs targetURL because it actually registers the full short URL and associates it with its destination, which requires knowing what this short URL should be pointing to.

2. This convention isn't used in every case because omitting names can introduce ambiguity when the variable and argument names are different or when multiple arguments can be confused

3. The Request.shortenUrl is representative of user input and basically triggers the process of shortening a URL. Generating a nonce in sync 1 and registering the shortened URL in sync 2 both require being linked to that user request (they are user-driven steps). Sync 3 occurs after a URL has been registered and so it is not triggered by a user action but by a system event and therefore does not include Request.shortenURL.

4. If there is only a fixed domain name, then we would no longer need shortUrlBase as an argument and instead it would be a constant (implicitly). The generate action would just take one global context (or no context) and the register action would automatically prepend, in this example, "bit.ly/" to the suffix. 

5. We should have a sync to remove a URL shortening from the system so that it does not resolve when a resource expires. This sync would look like:

        sync expire
            when ExpiringResource.expireResource(): (resource: Resource)
            then UrlShortening.delete(shortUrl: resource)


## Extending the design

### 1. Additional Concepts

**concept** Analytics

**purpose** track and report the number of accesses for each short URL

**principle** after each lookup of a short URL, its access count is increased

**state**

    a Map with
        a shortURL String mapped to a count Number

**actions**

    accessed (shortUrl: String)
        **effects** if shortUrl not in map, insert with count = 1; otherwise increment existing count by 1

    getCount (shortUrl: String) : (count: Number)
        **requires** shortUrl already exists in the map
        **effects** returns the number of recorded accesses for shortUrl

**concept** Ownership

**purpose** ensure that analytics can only be accessed by the user who created the shortened URL

**principle** assigns and verifies owners with shortened URLs

**state** 

    a Map with 
        a shortUrl String mapped to a user User


**actions**

    assignOwnership(user: User, shortUrl: String) 
        **effects** creates a new entry in the map with shortUrl mapped to user

    verifyOwnership(user: User, shortUrl: String) : (verified: Boolean)
        **requires** shortUrl exists in the map
        **effects** returns true (and allows user access to analytics) if user is the owner of shortUrl, false otherwise
    

### 2. Three Essential Synchronizations

1. Assign ownership of a shortened URL when it is registered

        sync assignOwnership
            when UrlShortening.register(user, targetUrl, shortUrlBase) : (shortUrl)
            then Ownership.assignOwnership(user, shortUrl)

    - This should run immediately after a short URL is created
    - This associates the URL with the user who created it so that later analytics can be gated by ownership

2. Increment analytics access count on lookup

        sync incrementCount 
            when UrlShortening.lookup(shortUrl)
            then Analytics.accessed(shortUrl)

    - Every time a shortened URL is looked up, the count is incremented or initialized to 1 if it is the first time accessing
    - This only records traffic so there is no need for user context

3. Viewing analytics if ownership verified only

        sync viewAnalytics
            when 
                Request.viewAnalytics(user, shortUrl)
                Ownership.verifyOwnership(user, shortUrl) : (verified)
            then
                Analytics.getCount(shortUrl)

    - This is triggered when a user requests analytics for a given URL and uses verifyOwnership to ensure that they are authorized (then calls getCount if they are indeed verified)
    - Sync doesn't proceed if the verification fails

### 3. Feature Requests

1. To allow users to choose their own short URLs, then we can add an optional parameter, chosenSuffix, to the register action of UrlShortening. We would adjust this action by adding a requirement that the chosen suffix is not already being used, but no other changes are required -- therefore, we can see that the solution is indeed modular since this feature is just a change to the UrlShortening concept.

2. To use the "word as nonce" strategy, we would modify NonceGeneration such that it supports an available word list mode where generate chooses an unused word or word combination. This also shows that my solution is modular since the changes only occur in NonceGeneration.

3. This feature would be an undesirable addition. Its purpose conflicts with the intent of analytics as a concept, which is to understand how short links are used and not to monitor activity at the target URLs themselves. Also, it could raise privacy issues: for instance, different owners of shortened URLs that resolve to the same target URL might gain visibility into each other's traffic data.

4. This feature would be an undesirable addition. While it would be possible to modify NonceGeneration to produce cryptographically secure random strings, doing so would make the resulting shortened URLs much harder to guess or remember. This nullifies the core purpose of a shortened URL, which is to provide a shorter/simpler or more memorable way to link.

5. This feature would be an undesirable addition. Adding support for reporting analytics to non-registered creators would pose potential security risks (if tokens that are used to access the analytics are leaked or guessed) and weakens access control. Since analytics are a value-add feature, it seems reasonable to require users to register to keep the system simpler and more secure. 

*Notes*: Just for completeness, if this feature needed to be added, we would allow the assignOwnership action to also store an email address or token as the owner of a short Url and then add an action to create a one-time link that allows analytics viewing without a user account.