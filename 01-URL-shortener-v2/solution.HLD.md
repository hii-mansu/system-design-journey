## Architecture Decisions

### **A. Scaling Infrastructure (Based on 500 Users)**

* We do **not** need multiple servers.
* We do **not** need a load balancer.
* **Why:** A single baseline application instance can easily handle 500 active users.

### **B. Database Architecture (Based on 5,000 URLs)**

* We do **not** need separate databases for Read & Write operations.
* We do **not** need Database Sharding.
* **Why:** 5,000 rows is an small dataset. The entire database can comfortably sit in the memory of a single database server..

### **C. Traffic & Counter Management (Based on 10,000 Clicks/Day)**

* We do **not** need a separate table for tracking the `clicks` count.
* We do **not** need a dedicated counter service or a message queue to handle increments.
* **Why:** 10,000 clicks per day averages out to roughly **10000 clicks / 24*60*60 = 0.11 requests per second (RPS)**. Even if traffic spikes to a few requests per second, a standard relational database can easily handle the workload by executing atomic counters directly on the main table.

---

## API Design

### 1. Create Short URL

* **Protocol:** `POST /create`
* **Request Body:**
```json
{
  "url": "https://www.google.com/mansu-singh"
}

```


* **Response Body:**
```json
{
  "shortUrl": "https://myurl.com/abc123"
}

```



### 2. Redirect URL

* **Protocol:** `GET /{short_code}` (e.g., `GET /abc123`)
* **Response:** * `302 Redirect`
* **Headers:** `Location: https://www.google.com/mansu-singh`



### 3. Analytics API

* **Protocol:** `GET /{short_code}/analytics` (e.g., `GET /abc123/analytics`)
* **Response Body:**
```json
{
  "shortUrl": "https://myurl.com/abc123",
  "clicks": 500
}

```



---

## Database Design

### **Table: `urls**`

| Column | Type | Description / Constraints |
| --- | --- | --- |
| `id` | `BIGINT` | Primary Key, Auto-incrementing |
| `main_url` | `TEXT` | The original long URL |
| `short_code` | `VARCHAR(10)` | Unique code for short URL |
| `created_at` | `TIMESTAMP` | Timestamp of creation |
| `clicks` | `BIGINT` | Total counter, defaults to 0 |

---

## Core Components

```
[ User ] <---> [ API Server ] <---> [ Database ]

```

1. **User:** * Submits long URLs for shortening.
* Accesses shortened links.
* Views performance analytics per URL.


2. **API Server:** * Unique `short_code` generation.
* Executes the business logic for URL redirection.


3. **Database:** * Data mappings (`id`, `short_code`, `main_url`, `created_at`).
* Atomically increments and updates the click counter.



---

## System Workflows

### **1. Creation Flow**

1. The **User** sent a long URL via the creation API: `POST /create`.
2. The **API Server** processes the request, computes a unique `short_code`, and update the database record.
3. The **Database** saves the entry and returns a success confirmation.
4. The **API Server** constructs the final short URL string and replies to the user.

### **2. Redirection Flow**

1. The **User** Visit short link (e.g., `myurl.com/abc123`).
2. The **API Server** catch the request and extracts the unique code (`abc123`).
3. The **API Server** queries the **Database** for a record matching the `short_code`.
4. The **Database** resolves the record and passes back the matching `main_url`.
5. The **API Server** return an HTTP `302 Found` response containing the destination URL.
6. The client **Browser** processes the redirect header and automatically loads the destination page.

### **3. Click Counting Flow**

1. The **User** open to a short link (e.g., `myurl.com/abc123`).
2. The **API Server** catch the request and extracts the token (`abc123`).
3. The **API Server** queries the database to retrieve the target destination.
4. The **Database** increments the specific row's counter by 1 using an **atomic database transaction**.
5. The **Database** returns the `main_url`.
6. The **API Server** issues an HTTP `302 Found` redirect response.
7. The client **Browser** loads the final destination website.

---

## Engineering Trade-offs

### **Inline Counter vs. Dedicated Analytics Table**

Embedding the `clicks` column directly into the primary `urls` record table provides maximum simplicity. It eliminates the overhead of managing table joins or secondary data stores for a project operating with (500 users, 5,000 links, and 10,000 daily clicks).

Updates are processed directly and safely via an atomic query:

```sql
UPDATE urls 
SET clicks = clicks + 1 
WHERE short_code = 'abc123';

```

**Risk Assessment:** If traffic grows, updating the same row on every redirect could become a bottleneck, and a separate analytics service or queue might be needed..