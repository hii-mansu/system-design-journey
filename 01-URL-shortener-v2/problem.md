# Problem:

## Build a URL Shortening Service that also tracks the total number of clicks for each shortened URL.


### A URL:

`https://myurl.com/abc123`

### Which redirects to:

`https://www.google.com/search?q=system+design`


### Constraints:

* **Users:** 500
* **URLs:** 5000
* **Clicks per day:** 10000
* **Architecture:**
* Single Server
* Single Database
* No Cache
* No Redis
* No Load Balancer
* No Microservices




### Functional Requirements:

1. User can submit a long URL.
2. System generates a unique short URL.
3. System stores the mapping between short URL and original URL.
4. User can access the short URL.
5. System count the total number of clicks per URL and save in DB.
6. User can retrieve the total click count for a short URL.
7. System redirects the user to the original URL.

### Non Functional Requirements:

* Each short URL should be unique.
* System should be simple and easy to maintain.
* Click count should be accurate.