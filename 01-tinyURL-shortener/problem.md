Problem:
Build a URL shortening service like:

A URL:
https://myurl.com/abc123

Which redirects to:
https://www.google.com/search?q=system+design

For:
100 users, 1000 URLs

Functional Requirements:
User can submit a long URL.
System generates a unique short URL.
System stores the mapping between short URL and original URL.
User can access the short URL.
System redirects the user to the original URL.

Non Functional Requirements:
Each short URL should be unique.
System should be simple and easy to maintain.