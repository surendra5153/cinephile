# 🧠 Similar Movie Recommendation Algorithm - Detailed Explanation

## Overview

The similar movie recommendation system uses **content-based filtering** with a **knowledge graph** and advanced **Data Structures & Algorithms** to provide intelligent movie recommendations.

---

## Algorithm Flow

```
User searches/opens movie
        ↓
Build Knowledge Graph (if not built)
        ↓
Find related movies using BFS traversal
        ↓
Calculate similarity scores for each candidate
        ↓
Rank using Priority Queue (Max Heap)
        ↓
Return top-K similar movies
```

---

## Step 1: Knowledge Graph Construction

### Graph Structure

The knowledge graph is an **adjacency list** representation:

```javascript
Graph {
    nodes: Map<nodeId, GraphNode>,
    adjacencyList: Map<nodeId, Array<GraphEdge>>,
    nodeIndex: Map<identifier, nodeId>
}
```

### Node Types

1. **Movie Node**: Contains movie data (title, year, rating, etc.)
2. **Genre Node**: Represents a genre
3. **Actor Node**: Represents an actor
4. **Director Node**: Represents a director
5. **Platform Node**: Represents a streaming platform
6. **User Node**: Represents a user (for ratings)

### Edge Types (Relationships)

| Relationship | From → To | Weight | Direction |
|-------------|-----------|--------|-----------|
| BELONGS_TO | Movie → Genre | 3 | Bidirectional |
| ACTED_IN | Movie ↔ Actor | 2 | Bidirectional |
| DIRECTED | Movie ↔ Director | 2 | Bidirectional |
| AVAILABLE_ON | Movie ↔ Platform | 1 | Bidirectional |
| RATED | User → Movie | rating/5 | Unidirectional |

### Construction Algorithm

```javascript
Algorithm: BuildKnowledgeGraph()
1. Load all entities from database:
   - Movies, Genres, Actors, Directors, Platforms, Ratings
   
2. Create nodes for each entity:
   FOR each movie:
       graph.addNode(movieId, 'movie', movieData, movieTitle)
   
   FOR each genre:
       graph.addNode(genreId, 'genre', genreData, genreName)
   
   (Repeat for actors, directors, platforms)
   
3. Create edges for relationships:
   FOR each movie_genre relationship:
       graph.addEdge(movieId, genreId, 'BELONGS_TO', weight=3)
       graph.addEdge(genreId, movieId, 'BELONGS_TO', weight=3)  // Bidirectional
   
   FOR each movie_actor relationship:
       graph.addEdge(movieId, actorId, 'ACTED_IN', weight=2)
       graph.addEdge(actorId, movieId, 'ACTED_IN', weight=2)  // Bidirectional
   
   (Repeat for directors, platforms, ratings)
   
4. Mark graph as built
```

**Time Complexity**: O(V + E) where V = vertices, E = edges
**Space Complexity**: O(V + E)

---

## Step 2: BFS Traversal for Related Movies

### Why BFS?

BFS (Breadth-First Search) explores nodes level by level, making it ideal for finding related movies within a limited distance (max depth 2).

### Algorithm

```javascript
Algorithm: FindRelatedMovies(movieId, maxDepth)
1. Initialize:
   - visited = Set()
   - queue = [(movieId, depth=0)]
   - candidates = []
   
2. BFS Traversal:
   WHILE queue not empty:
       (currentId, depth) = dequeue()
       
       IF depth >= maxDepth:
           CONTINUE
       
       node = graph.getNode(currentId)
       IF node AND node.type == 'movie' AND currentId != movieId:
           candidates.add(node)
       
       edges = graph.getNeighbors(currentId)
       FOR each edge in edges:
           neighborId = edge.to
           IF neighborId not in visited:
               visited.add(neighborId)
               IF depth < maxDepth:
                   queue.enqueue((neighborId, depth + 1))
   
3. RETURN candidates
```

### Example: Finding Related Movies for "Inception"

```
Starting: Movie(1, "Inception")

Depth 0:
  - Movie(1)

Depth 1 (direct connections):
  - Genre(10, "Action") via BELONGS_TO
  - Genre(11, "Sci-Fi") via BELONGS_TO
  - Actor(20, "Leonardo DiCaprio") via ACTED_IN
  - Director(30, "Christopher Nolan") via DIRECTED
  - Platform(40, "Netflix") via AVAILABLE_ON

Depth 2 (movies connected to depth 1 nodes):
  - Movie(2, "The Dark Knight") via Director(30)
  - Movie(3, "Interstellar") via Director(30)
  - Movie(4, "Shutter Island") via Actor(20)
  - Movie(5, "The Revenant") via Actor(20)
  - Movie(6, "Dunkirk") via Director(30)

Candidates: [Movie(2), Movie(3), Movie(4), Movie(5), Movie(6)]
```

**Time Complexity**: O(V + E) but typically much less (max depth 2)
**Space Complexity**: O(V) for visited set and queue

---

## Step 3: Similarity Score Calculation

### Scoring Formula

```javascript
SimilarityScore(M1, M2) = 
    Σ(commonGenres × 3) +
    Σ(commonActors × 2) +
    Σ(commonDirectors × 2) +
    Σ(commonPlatforms × 1) +
    ratingSimilarityScore(M1, M2)
```

### Detailed Scoring Breakdown

| Factor | Weight | Calculation |
|--------|--------|-------------|
| **Same Genre** | +3 per match | Count common genres between M1 and M2 |
| **Same Actor** | +2 per match | Count common actors between M1 and M2 |
| **Same Director** | +2 per match | Count common directors between M1 and M2 |
| **Same Platform** | +1 per match | Count common platforms between M1 and M2 |
| **Rating Similarity** | +2 if ≤0.5, +1 if ≤1.0 | Compare avg_rating: `|rating1 - rating2|` |

### Algorithm

```javascript
Algorithm: CalculateSimilarity(movieId1, movieId2)
1. IF movieId1 == movieId2:
       RETURN 0
   
2. Get movie nodes:
   movie1 = graph.getNode(movieId1)
   movie2 = graph.getNode(movieId2)
   
   IF not movie1 OR not movie2:
       RETURN 0
   
3. Get connected nodes:
   genres1 = graph.getConnectedNodes(movieId1, 'genre', 'BELONGS_TO')
   genres2 = graph.getConnectedNodes(movieId2, 'genre', 'BELONGS_TO')
   
   actors1 = graph.getConnectedNodes(movieId1, 'actor', 'ACTED_IN')
   actors2 = graph.getConnectedNodes(movieId2, 'actor', 'ACTED_IN')
   
   directors1 = graph.getConnectedNodes(movieId1, 'director', 'DIRECTED')
   directors2 = graph.getConnectedNodes(movieId2, 'director', 'DIRECTED')
   
   platforms1 = graph.getConnectedNodes(movieId1, 'platform', 'AVAILABLE_ON')
   platforms2 = graph.getConnectedNodes(movieId2, 'platform', 'AVAILABLE_ON')
   
4. Calculate genre overlap:
   genreSet1 = Set(genres1.map(g => g.id))
   genreSet2 = Set(genres2.map(g => g.id))
   commonGenres = intersection(genreSet1, genreSet2)
   score += commonGenres.length × 3
   
5. Calculate actor overlap:
   actorSet1 = Set(actors1.map(a => a.id))
   actorSet2 = Set(actors2.map(a => a.id))
   commonActors = intersection(actorSet1, actorSet2)
   score += commonActors.length × 2
   
6. Calculate director overlap:
   directorSet1 = Set(directors1.map(d => d.id))
   directorSet2 = Set(directors2.map(d => d.id))
   commonDirectors = intersection(directorSet1, directorSet2)
   score += commonDirectors.length × 2
   
7. Calculate platform overlap:
   platformSet1 = Set(platforms1.map(p => p.id))
   platformSet2 = Set(platforms2.map(p => p.id))
   commonPlatforms = intersection(platformSet1, platformSet2)
   score += commonPlatforms.length × 1
   
8. Calculate rating similarity:
   rating1 = movie1.data.avg_rating
   rating2 = movie2.data.avg_rating
   ratingDiff = |rating1 - rating2|
   
   IF ratingDiff <= 0.5:
       score += 2
   ELSE IF ratingDiff <= 1.0:
       score += 1
   
9. RETURN score
```

**Time Complexity**: O(1) - all operations are hashmap lookups
**Space Complexity**: O(1) - temporary sets for intersection

---

## Step 4: Ranking with Priority Queue (Max Heap)

### Why Priority Queue?

A **Max Heap** (Priority Queue) efficiently maintains the top-K highest scores without sorting all candidates, which is more efficient for large datasets.

### Max Heap Structure

```
          Max Heap
             8
           /   \
          7     5
         / \
        3   2
```

Root always contains the maximum value. Insertion and extraction are O(log n).

### Algorithm

```javascript
Algorithm: RankMovies(candidates, movieId, limit)
1. Initialize Priority Queue (max heap):
   pq = new PriorityQueue(comparator: (a, b) => a.score > b.score)
   
2. Calculate similarity scores and enqueue:
   FOR each candidate in candidates:
       IF candidate.id != movieId:
           score = calculateSimilarity(movieId, candidate.id)
           IF score > 0:
               pq.enqueue({
                   movie: candidate.data,
                   score: score
               })
   
3. Extract top-K:
   result = []
   FOR i = 1 to limit:
       IF pq not empty:
           item = pq.dequeue()  // Gets maximum score
           result.append({
               ...item.movie,
               similarityScore: item.score
           })
   
4. RETURN result
```

### Priority Queue Operations

**Enqueue (Insert)**:
```javascript
enqueue(item) {
    heap.push(item)           // Add to end
    heapifyUp(heap.length-1)  // Bubble up to maintain heap property
}
// Time: O(log n)
```

**Dequeue (Extract Max)**:
```javascript
dequeue() {
    max = heap[0]             // Root is maximum
    heap[0] = heap.pop()      // Replace root with last element
    heapifyDown(0)            // Bubble down to maintain heap property
    return max
}
// Time: O(log n)
```

**Time Complexity**: O(K log N) where K = limit, N = candidates
**Space Complexity**: O(N) for heap

---

## Complete Example Walkthrough

### Input

**Source Movie**: "Inception" (ID: 1)
- Genres: [Action, Sci-Fi, Thriller]
- Actors: [Leonardo DiCaprio, Tom Hardy]
- Director: Christopher Nolan
- Platform: [Netflix]
- Rating: 4.5

**Candidate Movies**:
1. "The Dark Knight" (ID: 2)
   - Genres: [Action, Crime, Drama]
   - Actors: [Christian Bale, Heath Ledger]
   - Director: Christopher Nolan
   - Platform: [Netflix, HBO Max]
   - Rating: 4.7

2. "Interstellar" (ID: 3)
   - Genres: [Adventure, Drama, Sci-Fi]
   - Actors: [Matthew McConaughey, Anne Hathaway]
   - Director: Christopher Nolan
   - Platform: [Netflix, Paramount+]
   - Rating: 4.6

3. "Shutter Island" (ID: 4)
   - Genres: [Mystery, Thriller]
   - Actors: [Leonardo DiCaprio, Mark Ruffalo]
   - Director: Martin Scorsese
   - Platform: [Netflix]
   - Rating: 4.2

### Step-by-Step Calculation

#### For Movie 2: "The Dark Knight"

1. **Genre Matching**:
   - Inception: {Action, Sci-Fi, Thriller}
   - The Dark Knight: {Action, Crime, Drama}
   - Common: {Action}
   - Score: 1 × 3 = **3 points**

2. **Actor Matching**:
   - Inception: {Leonardo DiCaprio, Tom Hardy}
   - The Dark Knight: {Christian Bale, Heath Ledger}
   - Common: {}
   - Score: 0 × 2 = **0 points**

3. **Director Matching**:
   - Inception: {Christopher Nolan}
   - The Dark Knight: {Christopher Nolan}
   - Common: {Christopher Nolan}
   - Score: 1 × 2 = **2 points**

4. **Platform Matching**:
   - Inception: {Netflix}
   - The Dark Knight: {Netflix, HBO Max}
   - Common: {Netflix}
   - Score: 1 × 1 = **1 point**

5. **Rating Similarity**:
   - Inception: 4.5
   - The Dark Knight: 4.7
   - Difference: |4.5 - 4.7| = 0.2 ≤ 0.5
   - Score: **2 points**

**Total Score: 3 + 0 + 2 + 1 + 2 = 8 points**

#### For Movie 3: "Interstellar"

1. **Genre Matching**:
   - Common: {Sci-Fi}
   - Score: 1 × 3 = **3 points**

2. **Actor Matching**:
   - Common: {}
   - Score: **0 points**

3. **Director Matching**:
   - Common: {Christopher Nolan}
   - Score: 1 × 2 = **2 points**

4. **Platform Matching**:
   - Common: {Netflix}
   - Score: 1 × 1 = **1 point**

5. **Rating Similarity**:
   - Difference: |4.5 - 4.6| = 0.1 ≤ 0.5
   - Score: **2 points**

**Total Score: 3 + 0 + 2 + 1 + 2 = 8 points**

#### For Movie 4: "Shutter Island"

1. **Genre Matching**:
   - Common: {Thriller}
   - Score: 1 × 3 = **3 points**

2. **Actor Matching**:
   - Common: {Leonardo DiCaprio}
   - Score: 1 × 2 = **2 points**

3. **Director Matching**:
   - Common: {}
   - Score: **0 points**

4. **Platform Matching**:
   - Common: {Netflix}
   - Score: 1 × 1 = **1 point**

5. **Rating Similarity**:
   - Difference: |4.5 - 4.2| = 0.3 ≤ 0.5
   - Score: **2 points**

**Total Score: 3 + 2 + 0 + 1 + 2 = 8 points**

### Priority Queue Ranking

```
Initial Heap (after enqueue):
        8 (The Dark Knight)
       / \
      8   8 (Interstellar, Shutter Island)

After dequeue 1:
        Result: [The Dark Knight (8)]
        Heap: 
              8 (Interstellar)
             /
            8 (Shutter Island)

After dequeue 2:
        Result: [The Dark Knight (8), Interstellar (8)]
        Heap: 
              8 (Shutter Island)

After dequeue 3:
        Result: [The Dark Knight (8), Interstellar (8), Shutter Island (8)]
        Heap: []
```

### Final Result

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
      "similarityScore": 8
    },
    {
      "id": 4,
      "title": "Shutter Island",
      "similarityScore": 8
    }
  ]
}
```

**Note**: In case of ties (same score), the order may vary. For deterministic results, add a secondary sort (e.g., by title or rating).

---

## Time Complexity Analysis

### Overall Complexity

1. **Graph Construction**: O(V + E) - One-time cost on startup
2. **BFS Traversal**: O(V + E) - Typically much less (max depth 2)
3. **Similarity Calculation**: O(C) where C = candidates, but each is O(1)
4. **Priority Queue Operations**: O(K log N) where K = limit, N = candidates

**Total per recommendation request**: O(V + E + K log N)

For a database with:
- 100 movies (V ≈ 100)
- 500 edges (E ≈ 500)
- 10 candidates (N ≈ 10)
- Top 10 results (K = 10)

**Approximate complexity**: O(610 + 10 log 10) ≈ **O(630)** operations

---

## Optimization Strategies

### 1. Caching

**Cache similarity scores** for popular movies:
```javascript
similarityCache = Map<`${movieId1}-${movieId2}`, score>
```

**Benefits**: 
- O(1) lookup for cached pairs
- Reduces computation for frequently requested movies

### 2. Pre-computation

**Pre-calculate top-K similar movies** for all movies:
```javascript
// Background job
FOR each movie:
    similarMovies = getSimilarMovies(movie.id, limit=10)
    cache.set(movie.id, similarMovies)
```

**Benefits**:
- Instant results for recommendations
- Reduced server load during peak times

### 3. Incremental Graph Updates

**Update graph incrementally** instead of rebuilding:
```javascript
// When new movie added
graph.addNode(newMovieId, 'movie', movieData)
graph.addEdges(newMovieId, relatedNodes)

// When new rating added
graph.addEdge(userId, movieId, 'RATED', rating/5)
```

**Benefits**:
- No need to rebuild entire graph
- Faster updates

### 4. Parallel Processing

**Calculate similarity scores in parallel**:
```javascript
// Use worker threads or async operations
const scores = await Promise.all(
    candidates.map(c => calculateSimilarity(movieId, c.id))
)
```

**Benefits**:
- Faster computation for large candidate sets
- Better CPU utilization

---

## Alternative Approaches

### 1. Collaborative Filtering

**User-based**: Find users with similar ratings, recommend movies they liked
- Pros: Can discover hidden patterns
- Cons: Requires many user ratings (cold start problem)

**Item-based**: Similar to content-based but uses user ratings
- Pros: More accurate for users with sparse ratings
- Cons: Computationally expensive

### 2. Hybrid Approach

**Combine content-based + collaborative filtering**:
```javascript
score = α × contentBasedScore + (1-α) × collaborativeScore
```

### 3. Machine Learning

**Use embeddings** (Word2Vec, Node2Vec) to represent movies as vectors:
```javascript
// Represent movie as vector
movieVector = [
    genreFeatures, actorFeatures, directorFeatures, ...
]

// Use cosine similarity
similarity = cosineSimilarity(movie1Vector, movie2Vector)
```

**Benefits**:
- Can capture complex patterns
- More accurate recommendations

**Cons**:
- Requires training data
- More complex implementation

---

## Summary

The similar movie recommendation algorithm uses:

1. **Knowledge Graph** for efficient relationship traversal
2. **BFS** to find related movies within 2 degrees of separation
3. **Multi-factor Similarity Scoring** based on genres, actors, directors, platforms, and ratings
4. **Priority Queue (Max Heap)** for efficient top-K ranking

**Time Complexity**: O(V + E + K log N) per request
**Space Complexity**: O(V + E) for graph + O(N) for priority queue

This approach provides accurate, fast recommendations while remaining interpretable and maintainable.
