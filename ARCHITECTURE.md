# 🏗️ Complete System Architecture Documentation

## 📋 Table of Contents
1. [System Overview](#system-overview)
2. [Tech Stack](#tech-stack)
3. [System Architecture](#system-architecture)
4. [Database Schema](#database-schema)
5. [Knowledge Graph Design](#knowledge-graph-design)
6. [Similar Movie Recommendation Algorithm](#similar-movie-recommendation-algorithm)
7. [DSA Implementation Details](#dsa-implementation-details)
8. [API Endpoints](#api-endpoints)
9. [Security & Moderation](#security--moderation)
10. [Scalability Strategy](#scalability-strategy)

---

## 🎯 System Overview

A live, online movie recommendation website that provides intelligent, similarity-based movie recommendations using:
- **Knowledge Graph**: Represents relationships between movies, genres, actors, directors, and platforms
- **Advanced DSA**: Graph traversal (BFS/DFS), Trie for search, Priority Queue for ranking, HashSet for filtering
- **Content-Based Filtering**: Multi-factor similarity scoring algorithm
- **Real-time Search**: Trie-based autocomplete with fast lookup

---

## 🛠️ Tech Stack

### Frontend
- **Framework**: React 18.2.0
- **Routing**: React Router DOM 6.20.1
- **HTTP Client**: Axios 1.6.2
- **Styling**: CSS Modules

### Backend
- **Runtime**: Node.js
- **Framework**: Express.js 4.18.2
- **Authentication**: JWT (jsonwebtoken 9.0.2)
- **Security**: bcryptjs 2.4.3
- **Validation**: express-validator 7.0.1
- **Rate Limiting**: express-rate-limit 7.1.5

### Database
- **Primary**: SQLite 3 (development)
- **Production Option**: PostgreSQL (compatible schema)
- **Graph Option**: Neo4j (alternative for knowledge graph)

### Data Structures & Algorithms
- **Graph**: Custom implementation using adjacency lists
- **Trie**: Custom implementation for autocomplete
- **Priority Queue**: Max Heap for ranking
- **HashSet**: For bad word filtering

---

## 🏛️ System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Client Layer (React)                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ Landing  │  │  Home    │  │  Movie   │  │  Search  │   │
│  │   Page   │  │  Page    │  │  Detail  │  │   Page   │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└────────────────────┬────────────────────────────────────────┘
                     │ HTTPS/REST API
┌────────────────────▼────────────────────────────────────────┐
│                  API Gateway (Express)                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Middleware Layer                                    │  │
│  │  • Authentication (JWT)                              │  │
│  │  • Rate Limiting                                    │  │
│  │  • Input Validation                                 │  │
│  │  • Bad Word Filtering                               │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                  Service Layer                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Auth       │  │Recommendation│  │    Search    │     │
│  │   Service    │  │   Service    │  │   Service    │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                  DSA Layer                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │  Graph   │  │   Trie   │  │Priority  │  │Bad Word  │  │
│  │ (BFS/DFS)│  │(Autocompl)│ │  Queue   │  │ Filter   │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                  Data Layer                                  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  SQLite Database (Production: PostgreSQL/Neo4j)     │  │
│  │  • Users, Movies, Genres, Actors, Directors         │  │
│  │  • Platforms, Ratings, Comments                     │  │
│  │  • Knowledge Graph (in-memory)                      │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Request Flow

```
User Request
    ↓
React Frontend (Components/Pages)
    ↓
API Service (Axios)
    ↓
Express Backend (Routes)
    ↓
Authentication Middleware (JWT)
    ↓
Service Layer (Business Logic)
    ↓
DSA Layer (Graph/Trie/PriorityQueue)
    ↓
Database Layer (SQLite)
    ↓
Response (JSON)
```

---

## 🗄️ Database Schema

### Entity-Relationship Diagram

```
┌─────────┐         ┌──────────┐         ┌─────────┐
│  Users  │         │  Movies  │         │ Genres  │
│─────────│         │──────────│         │─────────│
│ id (PK) │         │ id (PK)  │         │ id (PK) │
│ username│         │ title    │    ┌───▶│ name    │
│ passhash│         │ desc     │    │    └─────────┘
└────┬────┘         │ year     │    │
     │              │ poster   │    │    ┌─────────┐
     │              │ avg_rate │    │    │ Actors  │
     │              └────┬─────┘    │    │─────────│
     │                   │          └───▶│ id (PK) │
     │              ┌────┴─────┐         │ name    │
     │              │          │         └─────────┘
     │         ┌────▼────┐    │
     │         │ Ratings │    │    ┌──────────┐
     │         │─────────│    │    │Directors │
     │         │user_id  │    └───▶│──────────│
     │         │movie_id │         │ id (PK)  │
     │         │rating   │         │ name     │
     │         └─────────┘         └──────────┘
     │                              
     │         ┌──────────┐         
     │         │ Comments │         
     │         │──────────│         
     └────────▶│user_id   │         
               │movie_id  │    ┌──────────┐
               │content   │    │Platforms │
               └──────────┘    │──────────│
                               │ id (PK)  │
                               │ name     │
                               │ logo_url │
                               └──────────┘
```

### Tables Description

#### 1. Users Table
```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```
**Purpose**: Store user accounts (username-only authentication)

#### 2. Movies Table
```sql
CREATE TABLE movies (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    description TEXT,
    release_year INTEGER,
    poster_url TEXT,
    avg_rating REAL DEFAULT 0.0,
    total_ratings INTEGER DEFAULT 0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```
**Purpose**: Store movie information and aggregated ratings

#### 3. Genres Table
```sql
CREATE TABLE genres (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT UNIQUE NOT NULL
);
```
**Purpose**: Store movie genres

#### 4. Actors Table
```sql
CREATE TABLE actors (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT UNIQUE NOT NULL
);
```
**Purpose**: Store actor information

#### 5. Directors Table
```sql
CREATE TABLE directors (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT UNIQUE NOT NULL
);
```
**Purpose**: Store director information

#### 6. Platforms Table
```sql
CREATE TABLE platforms (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT UNIQUE NOT NULL,
    logo_url TEXT,
    watch_url_template TEXT
);
```
**Purpose**: Store streaming platform information

#### 7. Junction Tables

**movie_genres**: Many-to-many relationship between movies and genres
```sql
CREATE TABLE movie_genres (
    movie_id INTEGER NOT NULL,
    genre_id INTEGER NOT NULL,
    PRIMARY KEY (movie_id, genre_id),
    FOREIGN KEY (movie_id) REFERENCES movies(id),
    FOREIGN KEY (genre_id) REFERENCES genres(id)
);
```

**movie_actors**: Many-to-many relationship between movies and actors
```sql
CREATE TABLE movie_actors (
    movie_id INTEGER NOT NULL,
    actor_id INTEGER NOT NULL,
    PRIMARY KEY (movie_id, actor_id)
);
```

**movie_directors**: Many-to-many relationship between movies and directors
```sql
CREATE TABLE movie_directors (
    movie_id INTEGER NOT NULL,
    director_id INTEGER NOT NULL,
    PRIMARY KEY (movie_id, director_id)
);
```

**movie_platforms**: Many-to-many relationship between movies and platforms
```sql
CREATE TABLE movie_platforms (
    movie_id INTEGER NOT NULL,
    platform_id INTEGER NOT NULL,
    watch_url TEXT,
    PRIMARY KEY (movie_id, platform_id)
);
```

#### 8. Ratings Table
```sql
CREATE TABLE ratings (
    user_id INTEGER NOT NULL,
    movie_id INTEGER NOT NULL,
    rating INTEGER NOT NULL CHECK (rating >= 1 AND rating <= 5),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, movie_id)
);
```
**Purpose**: One rating per user per movie (enforced by composite primary key)

#### 9. Comments Table
```sql
CREATE TABLE comments (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    movie_id INTEGER NOT NULL,
    content TEXT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (movie_id) REFERENCES movies(id)
);
```
**Purpose**: Store user comments (with bad word filtering)

### Indexes for Performance

```sql
CREATE INDEX idx_movies_year ON movies(release_year);
CREATE INDEX idx_movies_rating ON movies(avg_rating DESC);
CREATE INDEX idx_ratings_user ON ratings(user_id);
CREATE INDEX idx_ratings_movie ON ratings(movie_id);
CREATE INDEX idx_comments_movie ON comments(movie_id);
CREATE INDEX idx_movie_genres_genre ON movie_genres(genre_id);
CREATE INDEX idx_movie_actors_actor ON movie_actors(actor_id);
```

### Database Comparison: SQL vs NoSQL vs Graph DB

#### SQL (PostgreSQL/SQLite)
**Pros:**
- ACID compliance
- Strong consistency
- Relational integrity
- Well-suited for structured data
- Good performance with proper indexing

**Cons:**
- Less flexible for complex relationships
- JOIN operations can be expensive

**Best for:** User data, ratings, comments, structured movie metadata

#### NoSQL (MongoDB)
**Pros:**
- Flexible schema
- Easy horizontal scaling
- Good for document-based data

**Cons:**
- Weaker consistency guarantees
- No native relationships
- Would need manual relationship management

**Best for:** Not ideal for this use case due to relationship-heavy data

#### Graph DB (Neo4j)
**Pros:**
- Native graph storage
- Excellent for relationship queries
- Built-in graph algorithms
- Cypher query language optimized for graphs

**Cons:**
- Steeper learning curve
- Higher operational complexity
- More expensive

**Best for:** Knowledge graph storage and complex relationship queries

**Recommendation:** 
- **Development**: SQLite (current)
- **Production**: PostgreSQL for main data + Neo4j for knowledge graph (hybrid approach)
- **Alternative**: PostgreSQL only with custom graph implementation (current approach)

---

## 🧠 Knowledge Graph Design

### Graph Structure

The knowledge graph is implemented using an **adjacency list** representation:

```javascript
Graph Structure:
{
    nodes: Map<nodeId, GraphNode>,
    adjacencyList: Map<nodeId, Array<GraphEdge>>,
    nodeIndex: Map<identifier, nodeId>
}
```

### Node Types

1. **Movie Node**
   - Type: `'movie'`
   - Data: Movie object (title, description, year, rating, etc.)
   - ID: Movie ID from database

2. **Genre Node**
   - Type: `'genre'`
   - Data: Genre object (name)
   - ID: Genre ID from database

3. **Actor Node**
   - Type: `'actor'`
   - Data: Actor object (name)
   - ID: Actor ID from database

4. **Director Node**
   - Type: `'director'`
   - Data: Director object (name)
   - ID: Director ID from database

5. **Platform Node**
   - Type: `'platform'`
   - Data: Platform object (name, logo_url)
   - ID: Platform ID from database

6. **User Node**
   - Type: `'user'`
   - Data: User object (username)
   - ID: User ID from database

### Edge Types (Relationships)

1. **BELONGS_TO**
   - From: Movie → To: Genre
   - Weight: 3
   - Bidirectional: Yes
   - Meaning: Movie belongs to genre

2. **ACTED_IN**
   - From: Movie ↔ To: Actor
   - Weight: 2
   - Bidirectional: Yes
   - Meaning: Actor acted in movie

3. **DIRECTED**
   - From: Movie ↔ To: Director
   - Weight: 2
   - Bidirectional: Yes
   - Meaning: Director directed movie

4. **AVAILABLE_ON**
   - From: Movie ↔ To: Platform
   - Weight: 1
   - Bidirectional: Yes
   - Meaning: Movie available on platform

5. **RATED**
   - From: User → To: Movie
   - Weight: rating / 5 (normalized)
   - Bidirectional: No
   - Meaning: User rated movie

### Graph Visualization Example

```
                    [Genre: Action]
                          ▲
                          │ BELONGS_TO (weight: 3)
                          │
        [Actor: Tom]──────┼──────[Movie: Inception]
         ▲                │                ▲
         │                │                │
    ACTED_IN (2)    ACTED_IN (2)    DIRECTED (2)
         │                │                │
    [Movie: Inception]───┼────────[Director: Nolan]
                          │
                          │ AVAILABLE_ON (1)
                          │
                    [Platform: Netflix]
```

### Graph Construction Algorithm

```javascript
Algorithm: BuildKnowledgeGraph()
1. Load all entities from database:
   - Movies, Genres, Actors, Directors, Platforms, Ratings
   
2. For each entity type:
   - Create nodes in graph
   - Add to node index (type:identifier → nodeId)
   
3. For each relationship:
   - movie_genres: Add BELONGS_TO edges (weight: 3)
   - movie_actors: Add ACTED_IN edges (weight: 2)
   - movie_directors: Add DIRECTED edges (weight: 2)
   - movie_platforms: Add AVAILABLE_ON edges (weight: 1)
   - ratings: Add RATED edges (weight: rating/5)
   
4. Mark graph as built
```

**Time Complexity**: O(V + E) where V = vertices, E = edges
**Space Complexity**: O(V + E)

---

## 🎬 Similar Movie Recommendation Algorithm

### Algorithm Overview

The recommendation algorithm uses **content-based filtering** with multi-factor similarity scoring, leveraging the knowledge graph for efficient traversal.

### Algorithm Steps

#### Step 1: Graph Construction
- Build knowledge graph from database (done once on startup or periodically)
- Each movie becomes a node
- Relationships (genres, actors, directors, platforms) form edges

#### Step 2: Similarity Calculation

For a given movie `M`, find similar movies by calculating similarity scores:

```javascript
SimilarityScore(M1, M2) = 
    Σ(genreMatches × 3) +
    Σ(actorMatches × 2) +
    Σ(directorMatches × 2) +
    Σ(platformMatches × 1) +
    ratingSimilarityScore(M1, M2)
```

**Scoring Breakdown:**

| Criteria | Weight | Calculation |
|----------|--------|-------------|
| Same Genre | +3 per match | Count common genres |
| Same Actor | +2 per match | Count common actors |
| Same Director | +2 per match | Count common directors |
| Same Platform | +1 per match | Count common platforms |
| Rating Similarity | +2 if ≤0.5 diff, +1 if ≤1.0 | Compare avg_rating |

#### Step 3: Graph Traversal (BFS)

Use **Breadth-First Search** to find related movies:

```javascript
Algorithm: FindSimilarMovies(movieId, limit)
1. Initialize:
   - visited = Set()
   - queue = [(movieId, depth=0)]
   - candidates = []
   
2. BFS Traversal (max depth: 2):
   WHILE queue not empty:
       (currentId, depth) = dequeue()
       IF depth >= 2: CONTINUE
       
       FOR each edge from currentId:
           neighborId = edge.to
           IF neighborId not visited:
               visited.add(neighborId)
               neighborNode = graph.getNode(neighborId)
               IF neighborNode.type == 'movie' AND neighborId != movieId:
                   candidates.add(neighborNode)
               IF depth < 2:
                   queue.enqueue((neighborId, depth + 1))
   
3. Calculate similarity scores for all candidates
   
4. Return top K using Priority Queue
```

#### Step 4: Ranking with Priority Queue (Max Heap)

```javascript
Algorithm: RankMovies(candidates, movieId, limit)
1. Initialize Priority Queue (max heap) with comparator: score
2. FOR each candidate in candidates:
       score = calculateSimilarity(movieId, candidate.id)
       IF score > 0:
           pq.enqueue({movie: candidate, score: score})
   
3. Extract top K:
       result = []
       FOR i = 1 to limit:
           IF pq not empty:
               result.append(pq.dequeue())
   
4. RETURN result sorted by score DESC
```

### Example Walkthrough

**Given:**
- Movie A: "Inception"
  - Genres: [Action, Sci-Fi, Thriller]
  - Actors: [Leonardo DiCaprio, Tom Hardy]
  - Director: Christopher Nolan
  - Platform: [Netflix]
  - Rating: 4.5

**Candidate:**
- Movie B: "The Dark Knight"
  - Genres: [Action, Crime, Drama]
  - Actors: [Christian Bale, Heath Ledger]
  - Director: Christopher Nolan
  - Platform: [Netflix, HBO Max]
  - Rating: 4.7

**Similarity Calculation:**

1. **Genre Matching:**
   - Common: [Action] → 1 match × 3 = **3 points**

2. **Actor Matching:**
   - Common: [] → 0 matches × 2 = **0 points**

3. **Director Matching:**
   - Common: [Christopher Nolan] → 1 match × 2 = **2 points**

4. **Platform Matching:**
   - Common: [Netflix] → 1 match × 1 = **1 point**

5. **Rating Similarity:**
   - Diff: |4.5 - 4.7| = 0.2 ≤ 0.5 → **2 points**

**Total Similarity Score: 3 + 0 + 2 + 1 + 2 = 8 points**

### Time Complexity Analysis

- **Graph Construction**: O(V + E) - one-time cost
- **BFS Traversal**: O(V + E) - typically much less (max depth 2)
- **Similarity Calculation**: O(1) per pair (hashmap lookups)
- **Priority Queue Operations**: O(K log N) where K = limit, N = candidates
- **Overall**: O(V + E + K log N) per recommendation request

### Optimization Strategies

1. **Caching**: Cache similarity scores for popular movies
2. **Incremental Updates**: Only rebuild graph when new movies added
3. **Pre-computation**: Pre-calculate top-K similar movies for popular movies
4. **Parallel Processing**: Calculate similarity scores in parallel for independent pairs

---

## 📊 DSA Implementation Details

### 1. Graph (Adjacency List)

**Purpose**: Knowledge graph representation

**Data Structure:**
```javascript
{
    nodes: Map<id, GraphNode>,           // O(1) lookup
    adjacencyList: Map<id, GraphEdge[]>, // O(1) edge access
    nodeIndex: Map<string, id>           // O(1) identifier lookup
}
```

**Operations:**
- `addNode(id, type, data)`: O(1)
- `addEdge(from, to, type, weight)`: O(1)
- `getNode(id)`: O(1)
- `bfs(startId, maxDepth)`: O(V + E)
- `dfs(startId, maxDepth)`: O(V + E)
- `getConnectedNodes(id, type)`: O(V + E)

**Implementation**: `backend/src/dsa/Graph.js`

### 2. Trie (Prefix Tree)

**Purpose**: Fast autocomplete and search

**Data Structure:**
```javascript
TrieNode {
    children: Map<char, TrieNode>,  // HashMap for O(1) char lookup
    isEndOfWord: boolean,
    data: Array                     // Store movie IDs/objects
}
```

**Operations:**
- `insert(word, data)`: O(m) where m = word length
- `search(word)`: O(m)
- `autocomplete(prefix, limit)`: O(m + k) where k = results
- `remove(word)`: O(m)

**Space Complexity**: O(ALPHABET_SIZE × N × M) where N = number of words, M = average length

**Implementation**: `backend/src/dsa/Trie.js`

### 3. Priority Queue (Max Heap)

**Purpose**: Ranking top-K similar movies

**Data Structure:**
```javascript
Binary Heap (Array-based)
Parent: floor((i - 1) / 2)
Left: 2i + 1
Right: 2i + 2
```

**Operations:**
- `enqueue(item)`: O(log n)
- `dequeue()`: O(log n)
- `peek()`: O(1)
- `getTopK(k)`: O(k log n)

**Implementation**: `backend/src/dsa/PriorityQueue.js`

### 4. HashSet (JavaScript Set)

**Purpose**: Bad word filtering (O(1) lookup)

**Data Structure:**
```javascript
Set<string>  // JavaScript native Set
```

**Operations:**
- `add(word)`: O(1) average
- `has(word)`: O(1) average
- `delete(word)`: O(1) average

**Implementation**: Used in `BadWordFilter.js`

### 5. HashMap (JavaScript Map)

**Purpose**: Fast lookups for users, ratings, similarity scores

**Data Structure:**
```javascript
Map<key, value>  // JavaScript native Map
```

**Usage:**
- User lookup: `Map<userId, userObject>`
- Movie index: `Map<movieId, movieObject>`
- Similarity cache: `Map<moviePair, similarityScore>`

---

## 🌐 API Endpoints

### Authentication Endpoints

#### POST `/api/auth/register`
Register a new user (username-only)

**Request:**
```json
{
  "username": "john_doe",
  "password": "password123"
}
```

**Response:**
```json
{
  "message": "User registered successfully",
  "user": {
    "id": 1,
    "username": "john_doe"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

#### POST `/api/auth/login`
Login user

**Request:**
```json
{
  "username": "john_doe",
  "password": "password123"
}
```

**Response:**
```json
{
  "message": "Login successful",
  "user": { "id": 1, "username": "john_doe" },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

#### GET `/api/auth/me`
Get current user (requires authentication)

**Headers:**
```
Authorization: Bearer <token>
```

**Response:**
```json
{
  "user": {
    "id": 1,
    "username": "john_doe",
    "created_at": "2024-01-01T00:00:00.000Z"
  }
}
```

### Movie Endpoints

#### GET `/api/movies`
Get all movies with pagination

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 20)
- `genre` (optional): Filter by genre ID
- `year` (optional): Filter by release year

**Response:**
```json
{
  "movies": [
    {
      "id": 1,
      "title": "Inception",
      "description": "...",
      "release_year": 2010,
      "poster_url": "...",
      "avg_rating": 4.5,
      "total_ratings": 1000,
      "genres": [...],
      "actors": [...],
      "directors": [...],
      "platforms": [...]
    }
  ],
  "page": 1,
  "limit": 20
}
```

#### GET `/api/movies/trending`
Get trending movies

**Query Parameters:**
- `limit` (optional): Number of movies (default: 10)

**Response:**
```json
{
  "movies": [...]
}
```

#### GET `/api/movies/top-decade`
Get top movies of the decade

**Query Parameters:**
- `startYear` (optional): Start year (default: 2010)
- `limit` (optional): Number of movies (default: 20)

**Response:**
```json
{
  "movies": [...],
  "decade": "2010-2019"
}
```

#### GET `/api/movies/genres/:genre`
Get movies by genre

**Response:**
```json
{
  "genre": "Action",
  "movies": [...]
}
```

#### GET `/api/movies/anime`
Get anime movies

**Response:**
```json
{
  "movies": [...]
}
```

#### GET `/api/movies/:id`
Get movie by ID

**Response:**
```json
{
  "movie": {
    "id": 1,
    "title": "Inception",
    "description": "...",
    "release_year": 2010,
    "poster_url": "...",
    "avg_rating": 4.5,
    "total_ratings": 1000,
    "genres": [...],
    "actors": [...],
    "directors": [...],
    "platforms": [...],
    "userRating": 4  // User's rating (if authenticated)
  }
}
```

#### GET `/api/movies/:id/similar`
Get similar movies (IMPORTANT FEATURE)

**Query Parameters:**
- `limit` (optional): Number of similar movies (default: 10)

**Response:**
```json
{
  "movies": [
    {
      "id": 2,
      "title": "The Dark Knight",
      "similarityScore": 8,
      "genres": [...],
      "platforms": [...]
    }
  ]
}
```

### Search Endpoints

#### GET `/api/search`
Search movies by query

**Query Parameters:**
- `q` (required): Search query
- `limit` (optional): Maximum results (default: 20)

**Response:**
```json
{
  "movies": [...]
}
```

#### GET `/api/search/autocomplete`
Get autocomplete suggestions

**Query Parameters:**
- `q` (required): Search prefix (min 2 characters)
- `limit` (optional): Suggestions per category (default: 5)

**Response:**
```json
{
  "movies": ["Inception", "Interstellar"],
  "genres": ["Action", "Adventure"],
  "actors": ["Leonardo DiCaprio"],
  "directors": ["Christopher Nolan"],
  "platforms": ["Netflix"]
}
```

### Rating Endpoints

#### POST `/api/ratings`
Create or update rating (requires authentication)

**Headers:**
```
Authorization: Bearer <token>
```

**Request:**
```json
{
  "movieId": 1,
  "rating": 5
}
```

**Response:**
```json
{
  "message": "Rating added successfully",
  "rating": {
    "userId": 1,
    "movieId": 1,
    "rating": 5,
    "avgRating": 4.5,
    "totalRatings": 1001
  }
}
```

#### GET `/api/ratings/movie/:movieId`
Get ratings for a movie

**Response:**
```json
{
  "ratings": [
    {
      "rating": 5,
      "created_at": "2024-01-01T00:00:00.000Z",
      "username": "john_doe"
    }
  ],
  "summary": {
    "avgRating": 4.5,
    "totalRatings": 1000
  }
}
```

### Comment Endpoints

#### POST `/api/comments`
Create comment (requires authentication, with bad word filtering)

**Headers:**
```
Authorization: Bearer <token>
```

**Request:**
```json
{
  "movieId": 1,
  "content": "Great movie! I loved the plot."
}
```

**Response:**
```json
{
  "message": "Comment added successfully",
  "comment": {
    "id": 1,
    "user_id": 1,
    "movie_id": 1,
    "content": "Great movie! I loved the plot.",
    "created_at": "2024-01-01T00:00:00.000Z",
    "username": "john_doe"
  }
}
```

**If bad words detected:**
```json
{
  "message": "Comment added (bad words filtered)",
  "comment": {...},
  "filtered": true,
  "badWordsFound": ["badword1"]
}
```

#### GET `/api/comments/movie/:movieId`
Get comments for a movie

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 20)

**Response:**
```json
{
  "comments": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 50,
    "totalPages": 3
  }
}
```

#### DELETE `/api/comments/:id`
Delete comment (requires authentication, owner only)

**Headers:**
```
Authorization: Bearer <token>
```

**Response:**
```json
{
  "message": "Comment deleted successfully"
}
```

---

## 🔐 Security & Moderation

### Authentication Security

1. **JWT Tokens**
   - Signed with secret key
   - Expires after 7 days
   - Stored in HTTP-only cookies (recommended) or localStorage

2. **Password Hashing**
   - bcrypt with salt rounds: 10
   - Never store plain passwords

3. **Username Validation**
   - Minimum 3 characters
   - Only letters, numbers, underscores
   - Case-insensitive uniqueness check

### Input Validation

- **express-validator** for all input validation
- SQL injection prevention via parameterized queries
- XSS prevention via content sanitization
- Rate limiting: 100 requests per 15 minutes per IP

### Bad Word Filtering

**Implementation:**
- Trie data structure for efficient matching
- HashSet for O(1) lookup
- Case-insensitive matching
- Word boundary detection

**Strategies:**
1. **Replace Bad Words**: Replace with asterisks (*) and allow comment
2. **Block Comment**: Reject comment entirely (configurable)

**Bad Words Dictionary:**
- Load from file/database (not hardcoded)
- Easily updatable
- Common profanity and spam words

### Rate Limiting

- **Global**: 100 requests / 15 minutes per IP
- **Authentication endpoints**: 5 requests / 15 minutes per IP
- **Rating endpoints**: 10 requests / hour per user
- **Comment endpoints**: 20 requests / hour per user

### Spam Prevention

1. **Rating Spam Prevention:**
   - One rating per user per movie (enforced by DB constraint)
   - Rate limiting on rating endpoints

2. **Comment Spam Prevention:**
   - Bad word filtering
   - Rate limiting
   - Duplicate comment detection (same content within short time)

---

## 📈 Scalability Strategy

### Horizontal Scaling

1. **Load Balancing**
   - Use reverse proxy (Nginx) or cloud load balancer
   - Multiple backend instances behind load balancer
   - Session stickiness not required (JWT stateless)

2. **Database Scaling**
   - **Read Replicas**: Separate read/write databases
   - **Sharding**: Shard by movie ID or genre
   - **Caching**: Redis for frequently accessed data

3. **Caching Strategy**
   - **Redis** for:
     - User sessions (if needed)
     - Popular movie data
     - Trending movies
     - Similar movie recommendations (pre-computed)
   - **CDN** for:
     - Static assets (posters, logos)
     - Frontend bundle

### Vertical Scaling

1. **Database Optimization**
   - Proper indexing (already implemented)
   - Query optimization
   - Connection pooling

2. **Graph Optimization**
   - Build graph incrementally
   - Cache similarity scores
   - Pre-compute top-K similar movies for popular movies
   - Use lazy loading for graph construction

### Performance Optimization

1. **Recommendation Algorithm**
   - **Pre-computation**: Calculate top-10 similar movies for all movies in background
   - **Caching**: Cache results for popular movies
   - **Lazy Loading**: Build graph on-demand if not already built

2. **Search Optimization**
   - **Index Building**: Build Trie indexes once on startup
   - **Caching**: Cache search results for common queries
   - **Elasticsearch**: Migrate to Elasticsearch for advanced search features

3. **API Optimization**
   - **Pagination**: All list endpoints paginated
   - **Field Selection**: Allow clients to specify needed fields
   - **Compression**: Enable gzip compression
   - **HTTP/2**: Use HTTP/2 for multiplexing

### Monitoring & Observability

1. **Logging**
   - Structured logging (Winston/Pino)
   - Log levels: ERROR, WARN, INFO, DEBUG
   - Request/response logging

2. **Metrics**
   - Response times
   - Error rates
   - Database query performance
   - Cache hit rates

3. **Alerting**
   - High error rates
   - Slow response times
   - Database connection issues

### Database Migration Path

**Development → Production:**

1. **SQLite → PostgreSQL**
   - Same schema (PostgreSQL-compatible)
   - Update connection string
   - Use connection pooling (pg-pool)

2. **Add Neo4j for Knowledge Graph**
   - Keep PostgreSQL for transactional data
   - Use Neo4j for graph queries
   - Sync data between databases

3. **Add Redis for Caching**
   - Cache movie data
   - Cache recommendations
   - Session storage (optional)

### Deployment Strategy

1. **Containerization**
   - Docker containers for backend and frontend
   - Docker Compose for local development
   - Kubernetes for production (if needed)

2. **CI/CD**
   - Automated testing
   - Automated deployments
   - Rollback capability

3. **Environment Variables**
   - Database connection strings
   - JWT secrets
   - API keys
   - Environment-specific configs

---

## 📝 Algorithm Example: Step-by-Step

### Example: Finding Similar Movies for "Inception"

**Step 1: Graph Construction**
```
Nodes Created:
- Movie(1, "Inception", {year: 2010, rating: 4.5})
- Genre(10, "Action")
- Genre(11, "Sci-Fi")
- Actor(20, "Leonardo DiCaprio")
- Director(30, "Christopher Nolan")
- Platform(40, "Netflix")

Edges Created:
- Movie(1) --BELONGS_TO(3)--> Genre(10)
- Movie(1) --BELONGS_TO(3)--> Genre(11)
- Movie(1) --ACTED_IN(2)--> Actor(20)
- Movie(1) --DIRECTED(2)--> Director(30)
- Movie(1) --AVAILABLE_ON(1)--> Platform(40)
```

**Step 2: BFS Traversal from Movie(1)**
```
Depth 0: Movie(1)
  ↓
Depth 1: Genre(10), Genre(11), Actor(20), Director(30), Platform(40)
  ↓
Depth 2: Other movies connected to these nodes

Candidates Found:
- Movie(2, "The Dark Knight") via Director(30)
- Movie(3, "Interstellar") via Director(30)
- Movie(4, "Shutter Island") via Actor(20)
```

**Step 3: Similarity Calculation**

For each candidate:
```
Movie(2, "The Dark Knight"):
  - Common genres: 1 (Action) → 3 points
  - Common actors: 0 → 0 points
  - Common directors: 1 (Nolan) → 2 points
  - Common platforms: 1 (Netflix) → 1 point
  - Rating similarity: |4.5 - 4.7| = 0.2 → 2 points
  Total: 8 points

Movie(3, "Interstellar"):
  - Common genres: 1 (Sci-Fi) → 3 points
  - Common actors: 0 → 0 points
  - Common directors: 1 (Nolan) → 2 points
  - Common platforms: 0 → 0 points
  - Rating similarity: |4.5 - 4.6| = 0.1 → 2 points
  Total: 7 points

Movie(4, "Shutter Island"):
  - Common genres: 0 → 0 points
  - Common actors: 1 (DiCaprio) → 2 points
  - Common directors: 0 → 0 points
  - Common platforms: 1 (Netflix) → 1 point
  - Rating similarity: |4.5 - 4.2| = 0.3 → 2 points
  Total: 5 points
```

**Step 4: Priority Queue Ranking**
```
Priority Queue (Max Heap):
  8 → Movie(2, "The Dark Knight")
  7 → Movie(3, "Interstellar")
  5 → Movie(4, "Shutter Island")

Dequeue top 3:
1. Movie(2) - Score: 8
2. Movie(3) - Score: 7
3. Movie(4) - Score: 5
```

**Result:**
```json
{
  "movies": [
    {
      "id": 2,
      "title": "The Dark Knight",
      "similarityScore": 8
    },
    {
      "id": 3,
      "title": "Interstellar",
      "similarityScore": 7
    },
    {
      "id": 4,
      "title": "Shutter Island",
      "similarityScore": 5
    }
  ]
}
```

---

## 🎯 Summary

This system architecture provides:

✅ **Complete authentication** with username-only login  
✅ **Intelligent recommendations** using knowledge graph + DSA  
✅ **Fast search** with Trie-based autocomplete  
✅ **Bad word filtering** for comments  
✅ **Scalable design** ready for production  
✅ **Well-documented** with clear algorithms  
✅ **Security features** (rate limiting, input validation)  

The system is production-ready with room for optimization and scaling as the user base grows.
