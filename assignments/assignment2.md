# Assignment 2

## Problem Statement

### Problem Domain — "Personal beverage discovery and tracking"

Many beverage enthusiasts struggle to remember and organize their experiences with specialty drinks across different locations. As someone who enjoys exploring matcha at various cafes, I've experienced the frustration of forgetting which places served exceptional drinks, what specifically made them memorable, and how to find new places worth trying. This domain matters because specialty beverage exploration is deeply personal - what constitutes "good" matcha varies significantly based on individual preferences for preparation style, sweetness, texture, and authenticity, yet these nuanced experiences are easily forgotten without systematic tracking.

### Problem — "Forgetting and losing track of good matcha experiences"

Some matcha enthusiasts may not effectively discover new quality locations and remember their experiences across different places, leading to missed opportunities and repeated disappointing visits. While general review platforms exist, they lack the specific attributes that matter for matcha (preparation method, grade, froth quality) and don't help users build personal preference understanding over time. This results in limited exploration beyond familiar places, wasted time on poor experiences, and inability to make confident recommendations to others.

### Stakeholders

1. Matcha Enthusiasts (Primary user) - People who regularly seek out and evaluate matcha experiences across different locations and want to both discover new places and build personal knowledge.

2. Occasional Matcha Drinkers (Secondary user) - People who occasionally enjoy matcha and want to find good options without extensive research overhead.

3. Cafe Owners - Business owners who could benefit from being discoverable to their target customers and understanding detailed preferences.

4. Friends/Social Circle - People who might benefit from accessing curated recommendations from trusted friends with similar tastes.

5. Journalists/Bloggers - People who may benefit from tracking matcha experiences for a specific article or project that they may be working on.


### Evidence and Comparables

1. Local listicles and blogs — e.g., [Buzzfeed: I Tried Matcha Lattes From 5 Popular Coffee Chains — Here's My Honest Ranking](https://www.buzzfeed.com/dannicaramirez/best-iced-matcha-latte-ranking) - pieces like this show recurring and often localized interest and unofficial taste tests. Articles ranking matcha spots provide discovery value but can't account for individual preferences or help users remember their own experiences over time.

2. Reddit threads (r/Matcha, local city subreddits) — users repeatedly ask where to find authentic matcha and the community crowdsources recommendations. This shows a demand for knowledge about matcha in specific locations, like this [Reddit thread: where to find the best matcha latte in Boston](https://www.reddit.com/r/boston/comments/uug25r/where_can_i_find_the_best_iced_matcha_latte_in/). Generally, people struggle to find quality options beyond well-known chains.

3. Specialty tracking apps (e.g., Untappd for beer) - These demonstrate successful models for combining discovery with personal experience tracking, but no equivalent exists for matcha or specialty tea experiences.

4. Social media food posts - Instagram and TikTok posts about matcha experiences show high engagement around discovery but poor organization for future reference and decision-making.

5. General review apps (Yelp, Google Reviews) - These capture broad impressions but miss the specific attributes that matter for specialty beverages and don't support building personal preference understanding. Google Maps is another example where you can search for nearby places, but once again it will not track your personal preferences in more detail (especially related to matcha).

6. Personal note-taking apps (e.g., Apple Notes) - Many users may resort to informal notes about cafes, but these lack structure for discovery and become hard to search or compare over time (and have fewer built-in features for tracking these matcha experiences).

## Application Pitch

**Name:** matchamatch

**Motivation:** Discover great matcha places and remember your experiences so you never miss a perfect cup or repeat a disappointing visit.

**Pitch:**  

matchamatch combines place discovery with personal experience tracking for matcha enthusiasts. Users can explore a curated database of matcha-serving locations on an interactive map, then log their personal experiences at each place with structured notes about preparation, taste, and satisfaction. The app builds both a discovery tool for finding new places to try and a personal memory system for tracking what you've enjoyed. Over time, logged experiences create personalized insights about preferences and generate smart recommendations for new places to explore based on your taste patterns.

**Key Features:**

1. **Matcha Place Discovery** — Interactive map showing matcha-serving cafes and specialty shops with basic info, photos if available, and other key details.

2. **Personal Experience Logger** — Quick logging interface for capturing your personal rating, preparation details, and notes after visiting any place, building your private experience history.

3. **Personal Collection View** — Browse your saved places and logged experiences with simple filtering and search to easily find that great place you tried last month.

## Concept Design

1. 
**concept** PlaceDirectory

**purpose** represent and manage known matcha-serving locations

**principle** places store structured metadata and can be discovered through search and location-based queries

**state**  

    a set of Places with
        a placeId PlaceId
        a name String
        an address String
        coordinates (Float, Float)
        preparationStyles set String
        priceRange String
        hours String?
        photos set URL


**actions**  

    create_place(name: String, address: String, coords: (Float, Float), styles: set String): PlaceId
        **requires** name and address are non-empty
        **effects** adds new Place with  placeId and given attributes to the set of Places

    find_nearby(coords: (Float, Float), radius: Float): set PlaceId
        **requires** radius > 0
        **effects** return {p.placeId | p in the set of Places and distance(p.coordinates, coords) <= radius}

    search_by_name(query: String): set PlaceId
        **effects** return {p.placeId | p in the set of Places and query in p.name}

    get_details(placeId: PlaceId): Place
        **requires** placeId in {p.placeId | p in the set of Places}
        **effects** return p where p.placeId = placeId

    filter_by_style(style: String): set PlaceId
        **effects** return {p.placeId | p in the set of Places and style in p.preparationStyles}


2. 
**concept** UserDirectory

**purpose** represent app users with identity, preferences, and saved places

**principle** each user maintains independent saved places and preferences 

**state**  

    a set of Users with
        a userId UserId
        a displayName String
        an email String
        preferences Map[String, String]
        savedPlaces set PlaceId

**actions**  

    register_user(userId: UserId, displayName: String, email: String): UserId
        **requires** userId not in {u.userId | u in the set of Users} and displayName, email are non-empty
        **effects** adds new User with given attributes and empty savedPlaces, preferences to the set of Users

    save_place(userId: UserId, placeId: PlaceId)
        **requires** userId in {u.userId | u in the set of Users}
        **effects** update user u where u.userId = userId: u.savedPlaces' = u.savedPlaces + {placeId}

    unsave_place(userId: UserId, placeId: PlaceId)
        **requires** userId in {u.userId | u in the set of Users} and placeId in user.savedPlaces
        **effects** update user u where u.userId = userId: u.savedPlaces' = u.savedPlaces - {placeId}

    update_preferences(userId: UserId, newPrefs: Map[String, String])
        **requires** userId in {u.userId | u in the set of Users}
        **effects** update user u where u.userId = userId: u.preferences' = newPrefs

    get_saved_places(userId: UserId): set PlaceId
        **requires** userId in {u.userId | u in the set of Users}
        **effects** return u.savedPlaces where u.userId = userId

    get_tried_places(userId: UserId): set PlaceId
        **requires** userId in {u.userId | u in the set of Users}
        **effects** return {log.placeId | log in ExperienceLog.logs and log.userId = userId}

3. 
**concept** ExperienceLog[User, Place]  

**purpose** capture a user's personal experience at a place with structured ratings and notes

**principle** each log entry represents one user's assessment of one place at a specific time; users can track and reference their personal experiences

**state**  

    a set of Logs with
        a logId LogId
        a userId User
        a placeId Place
        a timestamp DateTime
        a rating Integer
        sweetness Integer
        strength Integer
        notes String?
        photo URL?


**actions**  

    create_log(userId: User, placeId: Place, rating: Integer): LogId
        **requires** rating is in the inclusive range [1,5]
        **effects** adds new Log with new logId, given params, timestamp = now() to the set of Logs

    update_log(logId: LogId, rating: Integer?, sweetness: Integer?, strength: Integer?, notes: String?, photo: URL?)
        **requires** logId in {log.logId | log in the set of Logs} and if rating given then rating is in the inclusive range [1,5]
        **effects** update log where log.logId = logId with non-null parameters

    get_user_logs(userId: User): set Log
        **effects** return {log | log in the set of Logs and log.userId = userId}

    get_place_logs(userId: User, placeId: Place): set Log
        **effects** return {log | log in the set of Logs and log.userId = userId and log.placeId = placeId}

    delete_log(logId: LogId)
        **requires** logId in {log.logId | log in the set of Logs}
        **effects** updates the set of Logs such that: logs' = logs - {log | log.logId = logId}

    get_average_rating(userId: User, placeId: Place): Float
        **effects** return average of {log.rating | log in the set of Logs and log.userId = userId and log.placeId = placeId}


4. 
**concept** RecommendationEngine[User, Place]

**purpose** suggest places for users to try based on basic matching criteria

**principle** recommend places the user hasn't tried yet, prioritizing those similar to their saved places or matching their stated preferences

**state**

    a Map of Recommendations with
        a user User to a set Place
    
    a Map of lastUpdated with 
        a user User to a DateTime

**actions**

    get_recommendations(userId: User): set Place
        **effects** return recommendations[userId] if exists and recent, otherwise compute fresh recommendations

    refresh_recommendations(userId: User)
        **effects** recommendations[userId] = compute_suggestions(userId), lastUpdated[userId] = now()

    compute_suggestions(userId: User): set Place
        **effects** return places not in ExperienceLog.get_tried_places(userId), 
                scored by: similarity to saved places, preference matching, and/or global popularity

    clear_recommendations(userId: User)
        **effects** remove recommendations[userId] and lastUpdated[userId]


### Synchronizations

    sync SavedPlaceSync:
        when PlaceDirectory.create_place(name, address, coords, styles) returns placeId
        then UserDirectory.save_place(userId, placeId) can reference the new place

    sync ExperienceToTriedSync:
        when ExperienceLog.create_log(userId, placeId, rating) returns logId  
        then placeId becomes part of UserDirectory.get_tried_places(userId)

    sync LogDeletionSync:
        when ExperienceLog.delete_log(logId) removes user's last log for placeId
        then placeId is removed from UserDirectory.get_tried_places(userId)

    sync RecommendationRefreshSync:
        when UserDirectory.save_place(userId, placeId)
        then RecommendationEngine.refresh_recommendations(userId)

    sync PreferenceUpdateSync:
        when UserDirectory.update_preferences(userId, prefs)
        then RecommendationEngine.refresh_recommendations(userId)

    sync ExperienceUpdateSync:
        when ExperienceLog.create_log(userId, placeId, rating)
        then RecommendationEngine.refresh_recommendations(userId)

    sync NewPlaceSync:
        when PlaceDirectory.create_place(name, address, coords, styles) 
        then RecommendationEngine.refresh_recommendations(all users)

### Concept Roles in the App

PlaceDirectory serves as the central repository of all matcha-serving locations, providing the foundation for discovery through location-based search, name search, and style filtering. UserDirectory manages user identity and personal data, including saved places (bookmarks for future visits) and preferences that inform recommendations.

ExperienceLog captures the core value proposition - detailed personal experiences at places that build into a searchable memory system. Each log contains structured ratings, taste attributes, notes, and photos that help users remember what made each place special or disappointing. The concept provides both individual experience lookup and aggregate functions like average ratings.

RecommendationEngine maintains personalized suggestions for each user, caching computed recommendations until user behavior changes warrant a refresh. It operates by analyzing user preferences, saved places, and experience history to suggest relevant places they haven't tried yet. The cached approach provides consistent recommendations while being responsive to user actions.

Together, these concepts create a discovery-to-memory cycle: PlaceDirectory enables discovery of new locations, UserDirectory tracks personal interests through saved places and preferences, ExperienceLog converts visits into structured memories, and RecommendationEngine uses all this data to suggest new places to explore. The synchronizations ensure that user actions trigger recommendation updates, keeping suggestions relevant without constant recomputation.

## UI Sketches

See [here](./UI_sketches_assgn2/UI_sketches.pdf)

## User Journey - meet Emma

Emma is a graduate student who recently developed a love for matcha but struggles to find good places beyond the one cafe she knows. She wants to explore more options while avoiding disappointing experiences. Emma opens matchamatch after a friend's recommendation. The onboarding screen explains how she can discover matcha places and track her experiences. She grants location access and sets her basic preferences for sweetness and strength levels that will help with her personal tracking and recommendations. The app briefly shows her how saving places and logging experiences will build her personal matcha guide.

Emma lands on the Home/Map screen showing an interactive map with a few matcha place pins around her area, including several she's never heard of. She can see distance from her location and filter by preparation styles like ceremonial or latte. Tapping on an intriguing pin called "Zen Tea House" takes her to the Place Detail screen, which shows photos, hours, price range, and preparation styles offered. She taps "Save" to bookmark it for later, then explores two more places the same way, saving the most promising ones. The next week, Emma visits Zen Tea House and tries their ceremonial matcha. Impressed by the perfect froth and subtle sweetness, she opens the app and navigates to the Log Entry screen. She rates it 5 stars, adjusts sliders for sweetness and strength, adds notes about the "perfect froth texture and beautiful presentation," and uploads a photo of the elegant ceramic bowl. After saving her log, she feels satisfied having captured exactly what made this experience special.

Over the following weeks, Emma visits 4 more saved places, logging each experience immediately afterward through the same Log Entry process. Her Collection screen now shows two organized sections: "Saved Places" (her future to-visit list) and "Logs" (her personal rating history). When she filters the logs for 4+ stars, she can quickly see her top discoveries. Each logged experience includes her detailed notes, making it easy to remember what made each place memorable or disappointing.

When Emma's parents visit, she opens her Collection screen, reviews her logged experiences, and confidently takes them to Zen Tea House - her highest-rated spot. She can explain exactly why she chose it based on her logged notes about preparation quality and taste. matchamatch has transformed her from someone who knew one good place into a confident matcha explorer with a curated, searchable personal guide to her city's best options.