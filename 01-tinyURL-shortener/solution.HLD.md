A: We have only "100 users" so:
- we dont need multi server,
- we dont need load balancer.


B: For "1000 URL's" so:
- Dont need seprate DB for Read & Write,
- and dont need Database Sharding.



API Design

Create Short URL:
POST /create

Request

{
  "url": "https://www.google.com/search?q=system+design"
}

Response

{
  "shortUrl": "https://myurl.com/abc123"
}


Redirect URL:
GET /abc123

Response

302 Redirect
Location: https://www.google.com/search?q=system+design



Database Design:

        Column             Type

        id                 BIGINT
        main_url           TEXT
        short_code         VARCHAR(10)
        created_at         TIMESTAMP

    
Components:
1. User
2. API Server
3. Database.

User:
- create URL,
- View url

API Server:
- handle creation and redirection logic

Database:
- store long url, shortCode, id and creation date.


Creation flow:
- user send long URL via creation api POST/create,
- server handle the unique shortCode creation and DB logic,
- Database return the saved confirmation,
- server returns short url to user in response

Redirection flow:
- user visits tinyUrl.com/abc123,
- tiny url server extract shortCode "abc123",
- server lookup in databse for this shortCode,
- databse returns the long url,
- server return HTTP 302 redirect,
- browser opens the long url