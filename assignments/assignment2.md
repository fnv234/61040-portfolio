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


this is going to be edited
### Place
 
  **concept** Place

  **purpose** represent a physical cafe/location that serves matcha

  **principle** places store structured metadata and can be updated or claimed

  **state**
  
    a Map from place ID to  
        - name String  
        - address String  
        - city String  
        - lat Float  
        - lon Float  
        - tags List of String  
        - hours String  
        - source_url String  
        - verified Bool  
        - sample_rating_score Float  
        - sample_review_count Int  
        - notes String

  **actions**

    view (placeId: ID): (place: Place)  
      **requires** place exists  
      **effect** returns the place metadata

    update_meta (placeId: ID, fields: Map)  
      **requires** place exists  
      **effect** updates metadata fields  

    claim_by_owner (placeId: ID, userId: ID)  
      **requires** place is unclaimed  
      **effect** links owner to place  

    flag_for_review (placeId: ID, userId: ID)  
      **requires** place exists  
      **effect** marks place for admin verification  

    aggregate_rating_update (placeId: ID)  
      **requires** place exists  
      **effect** recalculates average rating from reviews  


### Review[Target=Place]

  **concept** Review [Target=Place]

  **purpose** structured user-generated review about a Place  

  **principle** reviews are unique per user–place pair and contribute to aggregate scores  

  **state**
  
    a Map from review ID to  
      - targetPlaceId Place ID  
      - userId User ID  
      - grade Int  
      - sweetness Int  
      - froth Int  
      - preparation String  
      - notes String  
      - images List of URL  
      - timestamp DateTime  

  **actions**  

    create (targetPlaceId: ID, userId: ID, attrs: Map): (reviewId: ID)  
      **requires** target place exists and no prior review by user  
      **effect** creates new review linked to the place  

    edit (reviewId: ID, updates: Map)  
      **requires** review exists and belongs to user  
      **effect** updates fields of review  

    delete (reviewId: ID)  
      **requires** review exists and belongs to user  
      **effect** deletes review  

    compute_partial_score (reviewId: ID): (score: Float)  
      **requires** review exists  
      **effect** computes score contribution for aggregation  


### UserProfile

  **concept** UserProfile

  **purpose** store user identity and preferences  

  **principle** users manage their own saved places, reviews, and settings  

  **state**

    a Map from user ID to  
      - displayName String  
      - savedPlaces List of Place IDs  
      - preferences Map  
      - claimedPlaces List of Place IDs  

  **actions**

    save_place (userId: ID, placeId: ID)  
      **requires** place exists and user exists  
      **effect** adds place to user's saved list  

    unsave_place (userId: ID, placeId: ID)  
      **requires** place exists and user exists  
      **effect** removes place from user's saved list  

    export_saved (userId: ID): (summaryDoc: File)  
      **requires** user exists  
      **effect** generates exportable summary of saved places  

## Essential Synchronizations

- **Review - Place aggregate update**
  - When a new `Review` is created, the associated `Place` must update its `sample_rating_score` and `sample_review_count`.
  - When a `Review` is deleted or edited, the corresponding `Place` must recalculate its aggregate rating.

  In other words:  
    when `Review.create(targetPlaceId, userId, attrs)`  
    then `Place.aggregate_rating_update(targetPlaceId)`

    when `Review.edit(reviewId, updates)`  
    then `Place.aggregate_rating_update(targetPlaceId)`

    when `Review.delete(reviewId)`  
    then `Place.aggregate_rating_update(targetPlaceId)`


- **UserProfile - Place claim verification**
  - When a user claims a place, the system should update the place's claimed status and optionally flag it for admin verification.

  In other words:  
    when `UserProfile.claim_place(userId, placeId)`  
    then `Place.claim_by_owner(placeId, userId)`  
    then optionally `Place.flag_for_review(placeId, userId)`

- **UserProfile - Personal saved places sync**
  - When a user saves a place, that place should appear in their personal saved list and optionally be reflected on a map overlay.

  In other words:  
    when `UserProfile.save_place(userId, placeId)`  
    then update`UserProfile.savedPlaces(userId)`  
    then optionally update a potential map layer (for saved)


### Concept Roles in the App

The app centers on **`Place`** as the anchor concept, representing matcha-serving locations or brands. **`Review`** contributes community insights tied to places and drives discovery through aggregate ratings. **`UserProfile`** provides identity, ownership, and personalization — managing saved places, preferences, and private tastings. **`LogEntry`** is the atomic data structure representing personal tasting experiences, while **`TastingLog`** aggregates these into summaries and insights.  

Generic type parameters are used transparently:  
- In `Review<Target=Place>`, the `Target` is always instantiated as `Place`.  
- References such as `user_id`, `place_id`, and `logEntryId` are foreign keys to the `UserProfile`, `Place`, and `LogEntry` maps respectively.  

Together, these concepts form a modular structure: **community-facing contributions** (reviews, claimed places) and **personal-facing records** (logs, summaries) are synchronized through shared identifiers, allowing both private journaling and public discovery to coexist in the same system. 

## UI Sketches

See [here](./UI_sketches_assgn2/UI_sketches.pdf)

## User Journey - meet Mia

Mia is a young professional who recently developed a love for matcha during her visits to local cafes. She often struggles to remember which cafes she's visited, how she liked their drinks, and where she might find her next great cup of matcha. Mia wants a way to track her experiences and discover new places without spending hours searching online.

1. Step 1 — Discovery
One afternoon, Mia hears about matchamatch from a friend. She downloads it and opens the welcome screen (Wireframe 1). After quickly creating her account and setting her taste preferences with sliders (Wireframe 2), she lands on the Home/Map screen (Wireframe 3). The interactive map displays nearby cafes with tags like "ceremonial," "vegan-friendly," etc. Mia spots a new cafe that looks interesting.

2. Step 2 — Exploring a Place
When Mia taps the cafe, the Place Detail screen opens (Wireframe 6). She sees the address, photos, and reviews with structured ratings for sweetness, strength, and more. Curious, she saves the cafe to her personal list with the "Save" button (UserProfile.save_place).

3. Step 3 — Recording Her Experience
A few days later, Mia visits the cafe and enjoys their ceremonial matcha. Afterward, she opens matchamatch, returns to the cafe's page, and selects "Add Review" (Wireframe 7). She fills in ratings for sweetness and strength, writes a note, and uploads a photo. When she submits (Review.create), the cafe's overall scores update (Place.aggregate_rating_update). The review also stays tied to the place in her Saved Places (Wireframe 5), making it easy to revisit later.

4. Step 4 — Reflection
Later, Mia browses her Saved Places screen (Wireframe 5). She sees a summary of her favorite cafes and her own reviews for each one. Looking across them, she realizes she tends to prefer less-sweet matcha with a strong flavor.

5. Final Outcome
By combining discovery, reviews, and saving, matchamatch helps Mia keep track of her experiences and preferences without extra work. She no longer forgets which cafes she enjoyed, and she feels confident trying new spots while building a personal shortlist of favorites.
