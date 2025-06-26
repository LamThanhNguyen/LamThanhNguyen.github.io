---
date: 2025-06-20
description: "12 Key Design Pattern in MongoDB"
featured_image: "/images/12-key-design-pattern-mongodb.png"
tags: ["mongodb", "nosql", "design pattern"]
title: "I. 12 Key Design Pattern In MongoDB With Real World Use Case"
---

# Introduction
MongoDB provides a variety of design patterns to help you model data efficiently for real-world use cases. Below are 12 key patterns every developer should know, each illustrated with practical examples.

## 1. Attribute Pattern
### Scenario:

An online store has multiple product categories, such as smartphones, laptops, and clothing. Each category contains items with a wide range of attributes. For example:
- Smartphones: screen size, battery capacity, camera resolution, etc.
- Laptops: processor type, RAM size, storage type, etc.
- Clothing: size, color, material, etc.

Due to the variability and unpredictability of these attributes, using a rigid schema is inefficient.

### Applying the Attribute Pattern:

Instead of defining a schema with fixed fields, the attribute pattern suggests creating an array of key-value pairs within the product document. This approach simplifies schema management and provides flexibility.
```json
{
  "_id": "product123",
  "name": "iPhone 16",
  "category": "Smartphones",
  "price": 1299,
  "attributes": [
    {"k": "screen_size", "v": "6.3 inches"},
    {"k": "battery_capacity", "v": "3500mAh"},
    {"k": "camera_resolution", "v": "48MP"},
    {"k": "color", "v": "Silver"}
  ]
}
```

```js
db.products.find({
  attributes: {
    $elemMatch: { k: "screen_size", v: "6.3 inches" }
  }
})
```

### Advantages:

- **Flexible schema**: Easily accommodates new attributes.
- **Efficient querying**: Facilitates searches based on various product specifications.
- **Cleaner schema management**: Simplifies maintenance and reduces complexity.

This makes the **Attribute Pattern** an excellent choice for applications managing *diverse* and *unpredictable* data models.

## 2. Computed Pattern
### Scenario:

An online store wants to efficiently display the total amount for each customer order without having to recalculate it every time the order is viewed.
- Each order contains a list of products (items), with each product having a price and quantity.
- The total order amount can be calculated as the sum of (price × quantity) for all items.

### Applying the Computed Pattern:

With the **Computed Pattern**, you store the computed field (e.g., totalAmount) directly in the order document, updating it only when the order changes (items are added/removed or quantities change).

```json
{
  "_id": "order789",
  "customerId": "customer123",
  "items": [
    { "productId": "prod001", "price": 50, "qty": 2 },
    { "productId": "prod002", "price": 30, "qty": 1 }
  ],
  "totalAmount": 130  // 50*2 + 30*1
}
```

### Advantages:
- **Performance**: Retrieving the total is instantaneous—no need to aggregate on the fly.
- **Simplicity**: Application logic is simplified; the value is always present.
- **Scalability**: Beneficial for analytics dashboards and reports that display many order totals at once.

The **Computed Pattern** is ideal when you have data that is derived from other fields and you want fast, repeated access—such as *totals*, *averages*, or *scores*.

## 3. Bucket Pattern
### Scenario:

Suppose you’re building an IoT application that collects temperature readings from thousands of sensors every few seconds. Each reading includes a timestamp, sensor ID, and the temperature value.
- Inserting each reading as a separate document quickly leads to millions of tiny documents, which is inefficient for both storage and querying.
- Retrieving and analyzing readings for a specific period (for example, all readings from the last hour) becomes slow and costly.

### Applying the Bucket Pattern:

With the **Bucket Pattern**, instead of storing each sensor reading as an individual document, you group multiple readings into a single “bucket” document—for example, all readings from one sensor within a specific hour.
Here, the document collects all temperature readings from sensorA between 14:00 and 15:00 on June 25, 2025:
```json
{
  "_id": "sensorA_2025-06-25T14",
  "sensorId": "sensorA",
  "date": "2025-06-25T14",
  "readings": [
    { "timestamp": "2025-06-25T14:01:02Z", "temperature": 24.5 },
    { "timestamp": "2025-06-25T14:03:45Z", "temperature": 24.7 },
    { "timestamp": "2025-06-25T14:05:20Z", "temperature": 24.6 }
    // ... more readings within this hour
  ]
}
```

```js
db.sensor_readings.findOne({
  sensorId: "sensorA",
  date: "2025-06-25T14"
})
```

```js
db.sensor_readings.find({
  sensorId: "sensorA",
  date: { $gte: "2025-06-25T14", $lte: "2025-06-25T16" }
})
```

```js
db.sensor_readings.aggregate([
  { $match: { sensorId: "sensorA", date: "2025-06-25T14" } },
  { $unwind: "$readings" },
  { $group: {
      _id: null,
      avgTemperature: { $avg: "$readings.temperature" }
    }
  }
])
```

### Advantages:
- **Storage Efficiency**: Fewer documents, reduced overhead.
- **Query Performance**: Fast retrieval of a range of readings for a time window.
- **Aggregation**: Easier to calculate statistics (like average temperature per hour).

The **Bucket Pattern** is great for *time-series* data or any scenario with high-frequency, append-only events—like logs, metrics, or financial ticks—where grouping related records together in a single document optimizes performance and cost.

## 4. Schema Versioning Pattern
### Scenario:

Suppose you’re building a social networking app. As your product grows, you frequently add new features, such as a bio section, social links, or profile themes.
Over time, the user profile schema changes, but you want to maintain backward compatibility and avoid downtime for existing users.
- **Old user profiles** may only have basic fields: *username*, *email*, and *createdAt*.
- **New user profiles** may include additional fields: *bio*, *avatar*, *socialLinks*, *theme*, etc.

### Applying the Schema Versioning Pattern:

You add a schemaVersion field to each profile document, which indicates the structure and fields used. When reading a document, your application can adjust its logic based on the version, and you can perform migrations as needed.

```json
{
  "_id": "user123",
  "username": "bestuser",
  "email": "bestuser@email.com",
  "createdAt": "2023-05-01T10:00:00Z",
  "schemaVersion": 1
}
```

```json
{
  "_id": "user456",
  "username": "newbie",
  "email": "newbie@email.com",
  "createdAt": "2024-03-10T09:15:00Z",
  "bio": "DevOps enthusiast",
  "avatar": "https://example.com/avatar.png",
  "theme": "dark",
  "schemaVersion": 2
}
```

### Advantages:
- **Backward Compatibility**: Your app can read and handle old and new data formats.
- **Safe Rollouts**: You can roll out new features without disrupting existing data or users.
- **Flexible Migrations**: Migrate data in the background or on-demand, instead of requiring a “big bang” update.

The **Schema Versioning Pattern** is ideal for applications that evolve quickly, need to support multiple data versions, or must migrate data gradually without breaking existing functionality.

## 5. Tree Pattern
### Scenario:

You’re running an online marketplace that sells many types of products. Products are organized into a **hierarchical category tree**:
- Electronics
  - Computers
    - Laptops
    - Desktops
  - Smartphones
- Home & Garden
  - Furniture
  - Kitchen

Each category may have subcategories, and subcategories may have their own children, forming a tree structure. You want to store and efficiently query this hierarchy in MongoDB.

### Applying the Tree Pattern:

With the **Tree Pattern**, you represent each node (category) as a document that references its parent (and optionally its children). The most common approach is to store a parentId field in each category document.

**Parent Reference:**
```json
{
  "_id": "electronics",
  "name": "Electronics",
  "parentId": null
}
{
  "_id": "computers",
  "name": "Computers",
  "parentId": "electronics"
}
{
  "_id": "laptops",
  "name": "Laptops",
  "parentId": "computers"
}
```

**Child References:**
```json
{
  "_id": "computers",
  "name": "Computers",
  "children": ["laptops", "desktops"]
}
```

**Materialized Path:**
```json
{
  "_id": "laptops",
  "name": "Laptops",
  "path": ["electronics", "computers", "laptops"]
}
```

**Closure Table:**

Stores every ancestor–descendant pair in a separate collection.
```json
{ "ancestor": "electronics", "descendant": "laptops", "depth": 2 }
```

### Advantages:
- **Flexible**: Supports any number of levels in the hierarchy.
- **Easy to Traverse**: You can traverse up (find parent) or down (find children) using queries.
- **Efficient Storage**: Only stores references, not the whole tree, making updates and inserts simple.

The **Tree Pattern** is ideal for representing organizational structures, menus, categories, or any nested, hierarchical data in MongoDB.

## 6. Subset Pattern
### Scenario:

You run a blog platform where each blog post can have **hundreds or thousands of comments**. When displaying the post detail page, you want to **show only the top N (e.g., 5 or 10) comments** to keep the page load fast and the UI clean, while still storing all comments for deeper browsing or moderation.

### Applying the Subset Pattern:

- In your posts collection, each post document contains an array field (e.g., topComments) that holds just the first N comments (the “subset”).
- The rest of the comments are stored separately, either in another collection (comments), or in another array, or are fetched via a paginated API.

```json
{
  "_id": "post456",
  "title": "10 Tips for MongoDB",
  "content": "...",
  "topComments": [
    {
      "user": "alice",
      "text": "Great tips, thanks!",
      "createdAt": "2025-06-24T09:00:00Z"
    },
    {
      "user": "bob",
      "text": "Helped me a lot.",
      "createdAt": "2025-06-24T09:10:00Z"
    }
    // ... up to N comments
  ],
  "commentsCount": 250  // total comments
}
```

### Advantages:

- **Performance**: Quickly load and display the most relevant or recent comments without scanning all comments.
- **UX**: The user immediately sees a representative sample (“best” or “most recent”) comments.
- **Scalability**: Supports posts with large numbers of comments while keeping documents lean for the most common query.

The **Subset Pattern** is ideal when you need to quickly access or display a **small sample** of a potentially large set of subdocuments, such as:
- Top reviews in a product listing,
- Latest activities in a user’s feed,
- Most recent messages in a chat preview.

## 7. Approximation Pattern
### Scenario:

Imagine you’re running a large social media platform (like Twitter or Instagram).
Some posts go viral and can get millions of likes in a very short time.
Storing and counting every single like accurately for real-time display is costly and can overload the database—especially when displaying lists of trending posts or feeds.

### Applying the Approximation Pattern:

Instead of keeping an exact, real-time count (which would require updating the main document for every single like), you store an approximate count that is:
- Fast to read and update
- Close enough for display purposes
- Periodically reconciled with the true value

You might use an **approximate counter** (e.g., incrementing by batches, sampling, or using probabilistic algorithms like HyperLogLog for unique counts).

```json
{
  "_id": "post888",
  "content": "Look at this cute cat!",
  "likeCountApprox": 106500,  // Approximate!
  "lastRecountAt": "2025-06-25T13:00:00Z"
}
```

- Each time a like event is received, you **don’t increment the counter immediately** for every single event. Instead, you might:
  - Update the counter every 10, 100, or 1,000 likes (batching)
  - Use a lightweight cache or a message queue to aggregate increments
  - Occasionally **recount the true number** (e.g., every hour or when accuracy matters, like before a contest ends)

### Advantages:
- **Performance**: Dramatically reduces write load and document contention on very hot/viral data.
- **Scalability**: System easily handles bursts of high-volume events.
- **User Experience**: Users see counts that are "good enough" for social interaction and engagement.

The Approximation Pattern is perfect when **absolute precision isn’t needed in real time**, but speed and scalability are crucial—especially for “hot” documents that change frequently and have high contention.

## 8. Outlier Pattern
### Scenario:

Imagine you’re building a social networking platform where most users have a modest number of followers—say, under 1,000. You store follower user IDs in an array inside each user document for fast access:
```json
{
  "_id": "user123",
  "name": "AverageJoe",
  "followers": ["userA", "userB", "userC", "..."]  // usually a manageable size ~ 5K-10K entries
}
```

But, occasionally, you have **outlier users**—celebrities or influencers—who might have hundreds of thousands or millions of followers. Storing all those follower IDs in a single document would:
- **Bloat the document size** (MongoDB’s document limit is 16MB)
- Slow down updates
- Impact performance for both reading and writing

### Applying the Outlier Pattern:
- For *most users*, keep the follower array right in the main user document for simplicity and speed.
- For *outlier users* (with huge arrays), **split off** the large follower list into a separate collection or multiple “bucket” documents, and just keep a reference in the main user document.

**user collection:**
*Regular User:*
```json
{
  "_id": "user123",
  "name": "AverageJoe",
  "followers": ["userA", "userB", "userC"]  // inline
}
```

*Celebrity User (Outlier):*
```json
{
  "_id": "user999",
  "name": "FamousStar",
  "followerBuckets": [
    "followers_user999_1", 
    "followers_user999_2"
  ]
}
```

### followerBuckets collection:
```json
{
  "_id": "followers_user999_1",
  "followers": ["userX", "userY", "userZ", "..."]  // up to a manageable chunk
}
```

### Advantages:
- **Performance**: Keeps “normal” documents fast and small.
- **Scalability**: Supports rare, extreme outliers without hitting document size limits.
- **Flexible**: Handles the “big fish” in your data set without complicating your main document model.

The **Outlier Pattern** is perfect when most data fits a simple model, but you occasionally have “giant” documents (outliers) that would otherwise break limits or kill performance—such as viral posts with millions of likes, super users, or anything with long-tail growth.

## 9. Polymorphic Pattern
### Scenario:

Suppose you’re building a social app with an activity feed that can show many types of activities, such as:
- A user posts a status update
- A user uploads a photo
- A user likes a comment
- A user follows another user
Each activity has some common fields (e.g., who performed the action, timestamp), but each type also has unique fields.

### Applying the Polymorphic Pattern:

With the Polymorphic Pattern, each activity document contains the fields required for its type, along with a type (or activityType) field to describe which kind of activity it is.
This allows you to store all activity types in a single collection—simplifying queries and aggregations—while supporting a flexible schema for each activity type.

```json
{
  "_id": "activity1001",
  "userId": "user123",
  "timestamp": "2025-06-25T15:00:00Z",
  "type": "status_update",
  "text": "Just got a new job!"
}
{
  "_id": "activity1002",
  "userId": "user123",
  "timestamp": "2025-06-25T15:05:00Z",
  "type": "photo_upload",
  "photoUrl": "media#image/pic123.jpg",
  "caption": "Loving the new view!"
}
{
  "_id": "activity1003",
  "userId": "user234",
  "timestamp": "2025-06-25T15:10:00Z",
  "type": "like",
  "targetType": "comment",
  "targetId": "comment789"
}
```

### Advantages:
- **Single collection** for all activities—easy to query, aggregate, and paginate.
- Each document has **fields specific to its type** (polymorphism), saving space and improving clarity.
- **Schema flexibility**: Easy to add new activity types later without migration.

The **Polymorphic Pattern** is perfect when you want to store documents of different “shapes” (but with some commonality) in a single collection—making it easy to manage, search, and display mixed-type data.

## 10. Extended Reference Pattern
### Scenario:

Suppose you’re building an online store. Each order document references the products purchased. Normally, you might just store an array of product IDs in the order and fetch product details separately (classic referencing).
However, for performance and user experience, you want to show product previews (name, image, price) directly when viewing an order—*without extra database lookups*.

### Applying the Extended Reference Pattern:

With this pattern, the order document stores product references (IDs) alongside key product details (“extended” information). This way, the order always displays product snapshots as they appeared at purchase time—even if the main product collection changes later.

```json
{
  "_id": "order001",
  "userId": "user123",
  "createdAt": "2025-06-25T10:10:00Z",
  "products": [
    {
      "productId": "prod789",
      "name": "Wireless Headphones",
      "image": "media#images/headphones.jpg",
      "price": 99.99,
      "qty": 1
    },
    {
      "productId": "prod456",
      "name": "Bluetooth Speaker",
      "image": "media#images/speaker.jpg",
      "price": 49.99,
      "qty": 2
    }
  ]
}
```

### Advantages:
- **Performance**: No need for extra lookups when displaying orders—fast page loads.
- **Data Consistency**: The order reflects the product details at the time of purchase (important if product name/price changes later).
- **UX**: You can show product thumbnails, names, and prices right away in order history.

The **Extended Reference Pattern** is perfect when you want to “cache” or “snapshot” important referenced data alongside IDs, balancing normalization with performance and data consistency.

## 11. Pre-aggregation Pattern
### Scenario:

Suppose you run a popular blog platform with thousands of posts and millions of comments and likes.
You want to show **real-time dashboards** that display, for each post:
- Total number of comments
- Total number of likes
- Average rating
- Calculating these values on the fly every time a user loads a dashboard would require scanning and aggregating massive collections, leading to *slow performance* and high database load.

### Applying the Pre-aggregation Pattern:

With this pattern, you **store summary statistics directly on the post document** and keep them updated as new comments, likes, or ratings are added.
This way, your app can retrieve stats instantly, with no aggregation needed at read time.

```json
{
  "_id": "post999",
  "title": "Scaling MongoDB for Millions",
  "content": "...",
  "commentCount": 1823,
  "likeCount": 3450,
  "averageRating": 4.8
}
```
### Advantages:

- **Instant reads**: Dashboards and lists load with a single, fast query.
- **Low server load**: No aggregation at query time, saving compute resources.
- **Great for high-traffic apps**: Performance stays fast, even as data grows.

The **Pre-aggregation Pattern** is perfect for scenarios where you need real-time or frequent access to aggregate metrics, and where recalculating aggregates on every read would be slow or expensive.

## 12. Document Versioning Pattern
### Scenario:

Suppose you’re building a wiki platform (like Wikipedia) where users can collaboratively edit pages.
You want to keep a history of all edits so users can:
- View previous versions
- Restore an older version
- See what changed and when

### Applying the Document Versioning Pattern:

Instead of overwriting the existing wiki page document on each edit, you **create a new versioned document** for every update.
You can do this by:
- Adding a version field,
- Storing all versions in the same collection, or
- Keeping the latest version in the main collection and older versions in an “archive” collection.

```js
{
  "_id": "page123_v5",
  "pageId": "page123",
  "version": 5,
  "title": "History of Databases",
  "content": "...latest content...",
  "editedBy": "user321",
  "editedAt": "2025-06-25T10:30:00Z"
}
{
  "_id": "page123_v4",
  "pageId": "page123",
  "version": 4,
  "title": "History of Databases",
  "content": "...older content...",
  "editedBy": "user123",
  "editedAt": "2025-06-25T09:45:00Z"
}
```

### Advantages:
- **Complete audit/history**: Every change is tracked, with who/when/what.
- **Easy rollbacks**: Users can restore any previous version.
- **Change tracking**: View differences between versions for transparency.

The **Document Versioning Pattern** is ideal whenever you need to preserve history, audit changes, or support rollbacks in collaborative or regulated environments.

# Summary
Understanding these 12 MongoDB design patterns—and when to apply each—can help you design scalable, maintainable, and high-performance applications. Choose the right pattern for your needs, and combine them when necessary for best results.