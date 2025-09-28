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

**concept** PlaceDirectory

**purpose** represent and manage all known matcha-serving locations

**principle** places store structured metadata and can be saved, logged, or claimed by users  

**state**  

    a set of Places with
        a placeId PlaceId
        a name String
        an address String
        coordinates (Float, Float)
        preparationStyles set String
        priceRange String
        hours lone String
        photos set URL


**actions**  

    create_place(name: String, address: String, coords: (Float, Float), styles: set String): PlaceId
        **requires** name and address are non-empty
        **effects** add new Place with these attributes

    find_nearby(coords: (Float, Float), radius: Float): set PlaceId
        **requires** radius > 0
        **effects** return {p in placeId | distance(p.coordinates, coords) ≤ radius}

    search_by_name(query: String): set PlaceId
        **effects** return {p in placeId | query in p.name}

    get_details(p: PlaceId): PlaceInfo
        **requires** p in placeId
        **effects** return all information for place p

    filter_by_style(style: String): set PlaceId
        **effects** return {p in placeId | style in p.preparationStyles}



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

    register_user(id: UserId, display: String, emailAddr: String): UserId
      **requires** id not in userId and display, emailAddr are non-empty
      **effects** add new User with these attributes

    save_place(u: UserId, p: PlaceId)
      **requires** u in userId and p in placeId
      **effects** add p to u.savedPlaces

    unsave_place(u: UserId, p: PlaceId)
      **requires** u in userId and p in u.savedPlaces
      **effects** remove p from u.savedPlaces

    update_preferences(u: UserId, prefs: Map[String, String])
      **requires** u in userId
      **effects** set u.preferences = prefs

    get_saved_places(u: UserId): set PlaceId
      **requires** u in userId
      **effects** return u.savedPlaces


**concept** ExperienceLog[UserId, PlaceId]  

**purpose** capture a user's personal experience at a place  

**principle** each log entry captures one user's assessment of one place at a specific time; logs are structured, personal reviews (not just free text), stored privately (or optionally shared in an expanded scope of the app)

**state**  

    a set of Logs with
        a logId LogId
        a user User
        an item PlaceId
        a timestamp DateTime
        a rating Integer
        sweetness Integer
        strength Integer
        notes String?
        photo URL?


**actions**  

    create_log(u: UserId, i: PlaceId, rating: Integer): LogId
        **requires** rating is in the range [1,5] inclusive
        **effects** add new Log with these attributes and timestamp = now()

    update_log(log: LogId, rating: Integer, sweet: Integer, str: Integer, prep: String, note: String)
        **requires** log in logId and rating is in the range [1,5] inclusive
        **effects** update attributes of log

    get_user_logs(u: UserId): set LogId
        **effects** return {log in logId | log.user = u}

    get_item_logs(u: UserId, i: PlaceId): set LogId
        **effects** return {log in logId | log.user = u and log.item = i}

    delete_log(log: LogId)
        **requires** log in logId
        **effects** remove log from logId



**concept** PersonalCollection[UserId, PlaceId]

**purpose** organize and filter a user's saved and tried places.  

**principle** provide unified view of a user's personal data with filtering capabilities

**state**  

    a set of Collections with
      a user UserId
      saved set PlaceId
      tried set PlaceId


**actions**  

    add_to_saved(u: UserId, i: PlaceId)
      **requires** u and i exist
      **effects** add i to Collections[u].saved

    remove_from_saved(u: UserId, i: PlaceId)
      **requires** i in Collections[u].saved
      **effects** remove i from Collections[u].saved

    add_to_tried(u: UserId, i: PlaceId)
      **requires** u and i exist and ExperienceLog.get_user_logs(u) contains a log for i
      **effects** add i to Collections[u].tried

    get_saved(u: UserId): set PlaceId
      **effects** return Collections[u].saved

    get_tried(u: UserId): set PlaceId
      **effects** return Collections[u].tried

    filter_by_rating(u: UserId, minRating: Integer): set PlaceId
      **requires** minRating is within the range [1,5] inclusive
      **effects** return places from Collections[u].tried with average rating ≥ minRating



### Synchronizations

when PlaceDirectory.find_nearby(coords, radius) returns places
  then UserDirectory.save_place(u: UserId, placeId: PlaceId) can reference discovered places

when UserDirectory.save_place(u: UserId, placeId: PlaceId)
  then PersonalCollection.add_to_saved(u: UserId, placeId: PlaceId)

when ExperienceLog.create_log(u: UserId, placeId: PlaceId, rating: Integer)
  then PersonalCollection.add_to_tried(u: UserId, placeId: PlaceId)

when PersonalCollection.filter_by_rating(u: UserId, minRating: Integer)
  then use ExperienceLog.get_item_logs(u: UserId, placeId: PlaceId) for filtering

### Concept Roles in the App

The app centers on PlaceDirectory as the anchor concept, representing all known matcha-serving locations with their metadata for discovery and reference. ExperienceLog captures individual user experiences at places, providing the atomic structure for personal tasting records. User manages identity, saved places, and personal preferences while serving as the connection point for all user-generated content. PersonalCollection organizes and filters a user's saved and tried places.

Generic type parameters are not used in this design — all concepts work directly with concrete types:

- PlaceDirectory manages location data and metadata for all places and supports searching and filtering by style or proximity.

- UserDirectory maintains saved places (referencing Place IDs) and preferences for users.

- ExperienceLog links users to places through log entries containing ratings, attributes, notes, and timestamps.

- PersonalCollection organizes a user's saved and tried places and supports filtering by rating.

Together, these concepts form a coherent structure: PlaceDirectory provides discoverable locations, User manages personal identity and saved places, ExperienceLog captures detailed experiences, and PersonalCollection organizes and filters the user's activity. The synchronizations ensure that user actions (discovering places, saving places, logging experiences, updating preferences) update all relevant collections and personal views, creating a seamless discovery-to-memory workflow.

## UI Sketches

See [here](./UI_sketches_assgn2/UI_sketches.pdf)

## User Journey - meet Emma

Emma is a graduate student who recently developed a love for matcha but struggles to find good places beyond the one cafe she knows. She wants to explore more options while avoiding disappointing experiences.

1. Step 1 — Discovery Need
Emma opens matchamatch after a friend's recommendation. The onboarding explains how she can discover matcha places and track her experiences. She grants location access and sets her basic preferences for traits like sweetness and strength for her personal tracking. 

2. Step 2 — Exploring Options
On the Home/Discovery Map, Emma sees pins for 8 matcha places, including several she's never heard of. She can filter by specifics like distance from her location. Tapping on one shows details like hours, photos if available, and other details.

3. Step 3 — Saving Interesting Places
Emma finds 3 promising places and saves them to her personal collection for future visits. These saved places now appear highlighted on her map and in her "Saved Places" view.

4. Step 4 — First Experience
Emma visits one of her saved places and tries their ceremonial matcha. Afterward, she opens the app and logs her experience: 4-star rating, notes about the perfect froth and slightly sweet taste, plus a photo of the beautiful presentation.

5. Step 5 — Building Personal Collection
Over three weeks, Emma visits 4 more places, logging each experience with ratings and detailed notes. Some are great discoveries (5 stars), others disappointing (2 stars). Her Personal Collection now shows both saved places for future visits and tried places with her personal ratings.

6. Step 6 — Confident Decision Making
When Emma's parents visit, she opens her Personal Collection, filters for 4+ star experiences, and confidently takes them to her highest-rated spot. She can explain exactly why it's special based on her logged notes about preparation style and taste.

7. Final Outcome
matchamatch solves both Emma's discovery problem (finding new places) and organization problem (remembering experiences). She can discover places on the map, save promising ones, and build a personal reference of her actual experiences - no complex algorithms needed, just discovery, saving, and memory tracking.