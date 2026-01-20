# 📚 API Documentation

Complete API reference for the Movie Recommendation System.

## Base URL

```
Development: http://localhost:5000/api
Production: https://api.movierec.com/api
```

## Authentication

Most endpoints require authentication via JWT token in the `Authorization` header:

```
Authorization: Bearer <token>
```

---

## Authentication Endpoints

### Register User

**POST** `/api/auth/register`

Register a new user with username and password.

**Request Body:**
```json
{
  "username": "john_doe",
  "password": "password123"
}
```

**Validation:**
- `username`: Required, 3-30 characters, alphanumeric + underscores only
- `password`: Required, minimum 6 characters

**Response 201 (Success):**
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

**Response 400 (Error):**
```json
{
  "errors": [
    {
      "msg": "Username must be between 3 and 30 characters",
      "param": "username"
    }
  ]
}
```

**Response 400 (Duplicate Username):**
```json
{
  "error": "Username already exists"
}
```

---

### Login

**POST** `/api/auth/login`

Login with username and password.

**Request Body:**
```json
{
  "username": "john_doe",
  "password": "password123"
}
```

**Response 200 (Success):**
```json
{
  "message": "Login successful",
  "user": {
    "id": 1,
    "username": "john_doe"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response 401 (Error):**
```json
{
  "error": "Invalid username or password"
}
```

---

### Get Current User

**GET** `/api/auth/me`

Get current authenticated user information.

**Headers:**
```
Authorization: Bearer <token>
```

**Response 200 (Success):**
```json
{
  "user": {
    "id": 1,
    "username": "john_doe",
    "created_at": "2024-01-01T00:00:00.000Z"
  }
}
```

**Response 401 (Unauthorized):**
```json
{
  "error": "No token provided"
}
```

---

## Movie Endpoints

### Get All Movies

**GET** `/api/movies`

Get paginated list of movies with optional filters.

**Query Parameters:**
- `page` (optional, default: 1): Page number
- `limit` (optional, default: 20): Items per page
- `genre` (optional): Filter by genre ID
- `year` (optional): Filter by release year

**Headers:**
```
Authorization: Bearer <token>  (optional)
```

**Example Request:**
```
GET /api/movies?page=1&limit=20&genre=1&year=2010
```

**Response 200 (Success):**
```json
{
  "movies": [
    {
      "id": 1,
      "title": "Inception",
      "description": "A thief who steals corporate secrets through dream-sharing technology.",
      "release_year": 2010,
      "poster_url": "https://example.com/poster.jpg",
      "avg_rating": 4.5,
      "total_ratings": 1000,
      "genres": [
        { "id": 1, "name": "Action" },
        { "id": 2, "name": "Sci-Fi" }
      ],
      "actors": [
        { "id": 1, "name": "Leonardo DiCaprio" }
      ],
      "directors": [
        { "id": 1, "name": "Christopher Nolan" }
      ],
      "platforms": [
        {
          "id": 1,
          "name": "Netflix",
          "logo_url": "https://example.com/netflix.png",
          "watch_url": "https://netflix.com/watch/1"
        }
      ]
    }
  ],
  "page": 1,
  "limit": 20
}
```

---

### Get Trending Movies

**GET** `/api/movies/trending`

Get trending movies based on ratings and popularity.

**Query Parameters:**
- `limit` (optional, default: 10): Number of movies to return

**Example Request:**
```
GET /api/movies/trending?limit=10
```

**Response 200 (Success):**
```json
{
  "movies": [
    {
      "id": 1,
      "title": "Inception",
      "avg_rating": 4.5,
      "total_ratings": 1000,
      "genres": [...],
      "platforms": [...]
    }
  ]
}
```

---

### Get Top Movies of Decade

**GET** `/api/movies/top-decade`

Get top-rated movies from a specific decade.

**Query Parameters:**
- `startYear` (optional, default: 2010): Start year of decade
- `limit` (optional, default: 20): Number of movies to return

**Example Request:**
```
GET /api/movies/top-decade?startYear=2010&limit=20
```

**Response 200 (Success):**
```json
{
  "movies": [...],
  "decade": "2010-2019"
}
```

---

### Get Movies by Genre

**GET** `/api/movies/genres/:genre`

Get all movies in a specific genre.

**URL Parameters:**
- `genre`: Genre name (e.g., "Action", "Drama")

**Example Request:**
```
GET /api/movies/genres/Action
```

**Response 200 (Success):**
```json
{
  "genre": "Action",
  "movies": [...]
}
```

**Response 404 (Not Found):**
```json
{
  "error": "Genre not found"
}
```

---

### Get Anime Movies

**GET** `/api/movies/anime`

Get all anime movies.

**Example Request:**
```
GET /api/movies/anime
```

**Response 200 (Success):**
```json
{
  "movies": [
    {
      "id": 10,
      "title": "Spirited Away",
      "genres": [
        { "id": 5, "name": "Anime" }
      ],
      ...
    }
  ]
}
```

---

### Get Movie by ID

**GET** `/api/movies/:id`

Get detailed information about a specific movie.

**URL Parameters:**
- `id`: Movie ID

**Headers:**
```
Authorization: Bearer <token>  (optional - for userRating)
```

**Example Request:**
```
GET /api/movies/1
```

**Response 200 (Success):**
```json
{
  "movie": {
    "id": 1,
    "title": "Inception",
    "description": "A thief who steals corporate secrets...",
    "release_year": 2010,
    "poster_url": "https://example.com/poster.jpg",
    "avg_rating": 4.5,
    "total_ratings": 1000,
    "genres": [...],
    "actors": [...],
    "directors": [...],
    "platforms": [...],
    "userRating": 5  // Only if authenticated
  }
}
```

**Response 404 (Not Found):**
```json
{
  "error": "Movie not found"
}
```

---

### Get Similar Movies

**GET** `/api/movies/:id/similar`

Get similar movies using knowledge graph and similarity algorithm.

**URL Parameters:**
- `id`: Movie ID

**Query Parameters:**
- `limit` (optional, default: 10): Number of similar movies to return

**Example Request:**
```
GET /api/movies/1/similar?limit=10
```

**Response 200 (Success):**
```json
{
  "movies": [
    {
      "id": 2,
      "title": "The Dark Knight",
      "similarityScore": 8,
      "genres": [...],
      "platforms": [...],
      "avg_rating": 4.7,
      "total_ratings": 1500
    },
    {
      "id": 3,
      "title": "Interstellar",
      "similarityScore": 7,
      "genres": [...],
      "platforms": [...],
      "avg_rating": 4.6,
      "total_ratings": 1200
    }
  ]
}
```

**Algorithm Explanation:**
- Similarity score calculated based on:
  - Same genres: +3 per match
  - Same actors: +2 per match
  - Same directors: +2 per match
  - Same platforms: +1 per match
  - Rating similarity: +2 if ≤0.5 diff, +1 if ≤1.0
- Results ranked using Priority Queue (Max Heap)
- Top-K movies returned

---

## Search Endpoints

### Search Movies

**GET** `/api/search`

Search movies by name, genre, actor, director, or platform.

**Query Parameters:**
- `q` (required): Search query
- `limit` (optional, default: 20): Maximum results

**Headers:**
```
Authorization: Bearer <token>  (optional)
```

**Example Request:**
```
GET /api/search?q=inception&limit=20
```

**Response 200 (Success):**
```json
{
  "movies": [
    {
      "id": 1,
      "title": "Inception",
      "description": "...",
      "avg_rating": 4.5,
      ...
    }
  ],
  "query": "inception"
}
```

**Search Algorithm:**
- Uses Trie data structure for fast autocomplete
- Searches across:
  - Movie titles
  - Genres
  - Actors
  - Directors
  - Platforms
- Case-insensitive matching

---

### Autocomplete

**GET** `/api/search/autocomplete`

Get autocomplete suggestions for search query.

**Query Parameters:**
- `prefix` (required, min 2 characters): Search prefix
- `limit` (optional, default: 5): Suggestions per category

**Example Request:**
```
GET /api/search/autocomplete?prefix=inc&limit=5
```

**Response 200 (Success):**
```json
{
  "suggestions": {
    "movies": ["Inception", "Incredibles"],
    "genres": ["Action", "Adventure"],
    "actors": ["Leonardo DiCaprio"],
    "directors": ["Christopher Nolan"],
    "platforms": ["Netflix"]
  }
}
```

**Response 400 (Too Short):**
```json
{
  "suggestions": {
    "movies": [],
    "genres": [],
    "actors": [],
    "directors": [],
    "platforms": []
  }
}
```

---

## Rating Endpoints

### Create or Update Rating

**POST** `/api/ratings`

Create a new rating or update existing rating for a movie.

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "movieId": 1,
  "rating": 5
}
```

**Validation:**
- `movieId`: Required, must be valid integer
- `rating`: Required, integer between 1 and 5

**Response 200 (Success):**
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

**Response 400 (Invalid Rating):**
```json
{
  "errors": [
    {
      "msg": "Rating must be between 1 and 5",
      "param": "rating"
    }
  ]
}
```

**Response 404 (Movie Not Found):**
```json
{
  "error": "Movie not found"
}
```

**Note:** Only one rating per user per movie (enforced by database constraint)

---

### Get Movie Ratings

**GET** `/api/ratings/movie/:movieId`

Get all ratings for a specific movie.

**URL Parameters:**
- `movieId`: Movie ID

**Example Request:**
```
GET /api/ratings/movie/1
```

**Response 200 (Success):**
```json
{
  "ratings": [
    {
      "rating": 5,
      "created_at": "2024-01-01T00:00:00.000Z",
      "username": "john_doe"
    },
    {
      "rating": 4,
      "created_at": "2024-01-02T00:00:00.000Z",
      "username": "jane_smith"
    }
  ],
  "summary": {
    "avgRating": 4.5,
    "totalRatings": 1000
  }
}
```

---

## Comment Endpoints

### Create Comment

**POST** `/api/comments`

Create a comment for a movie (with bad word filtering).

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "movieId": 1,
  "content": "Great movie! I loved the plot and cinematography."
}
```

**Validation:**
- `movieId`: Required, must be valid integer
- `content`: Required, 1-1000 characters

**Response 201 (Success):**
```json
{
  "message": "Comment added successfully",
  "comment": {
    "id": 1,
    "user_id": 1,
    "movie_id": 1,
    "content": "Great movie! I loved the plot and cinematography.",
    "created_at": "2024-01-01T00:00:00.000Z",
    "username": "john_doe"
  }
}
```

**Response 201 (With Bad Words Filtered):**
```json
{
  "message": "Comment added (bad words filtered)",
  "comment": {
    "id": 1,
    "user_id": 1,
    "movie_id": 1,
    "content": "Great movie! I loved the plot. *****",
    "created_at": "2024-01-01T00:00:00.000Z",
    "username": "john_doe"
  },
  "filtered": true,
  "badWordsFound": ["spam"]
}
```

**Response 400 (Validation Error):**
```json
{
  "errors": [
    {
      "msg": "Comment must be between 1 and 1000 characters",
      "param": "content"
    }
  ]
}
```

**Bad Word Filtering:**
- Uses Trie and HashSet for fast detection
- Bad words replaced with asterisks (*)
- Option to block comment entirely (configurable)

---

### Get Movie Comments

**GET** `/api/comments/movie/:movieId`

Get all comments for a specific movie with pagination.

**URL Parameters:**
- `movieId`: Movie ID

**Query Parameters:**
- `page` (optional, default: 1): Page number
- `limit` (optional, default: 20): Items per page

**Example Request:**
```
GET /api/comments/movie/1?page=1&limit=20
```

**Response 200 (Success):**
```json
{
  "comments": [
    {
      "id": 1,
      "user_id": 1,
      "movie_id": 1,
      "content": "Great movie!",
      "created_at": "2024-01-01T00:00:00.000Z",
      "username": "john_doe"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 50,
    "totalPages": 3
  }
}
```

---

### Delete Comment

**DELETE** `/api/comments/:id`

Delete a comment (only by owner).

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
- `id`: Comment ID

**Example Request:**
```
DELETE /api/comments/1
```

**Response 200 (Success):**
```json
{
  "message": "Comment deleted successfully"
}
```

**Response 403 (Forbidden):**
```json
{
  "error": "You can only delete your own comments"
}
```

**Response 404 (Not Found):**
```json
{
  "error": "Comment not found"
}
```

---

## Error Responses

### Standard Error Format

```json
{
  "error": "Error message"
}
```

### Validation Error Format

```json
{
  "errors": [
    {
      "msg": "Validation error message",
      "param": "fieldName"
    }
  ]
}
```

### HTTP Status Codes

- `200 OK`: Request successful
- `201 Created`: Resource created successfully
- `400 Bad Request`: Invalid request data
- `401 Unauthorized`: Authentication required or invalid token
- `403 Forbidden`: Insufficient permissions
- `404 Not Found`: Resource not found
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Server error

---

## Rate Limiting

### Global Rate Limit
- **Limit**: 100 requests per 15 minutes per IP
- **Header**: `X-RateLimit-Limit: 100`
- **Remaining**: `X-RateLimit-Remaining: 95`

### Endpoint-Specific Limits
- **Authentication**: 5 requests / 15 minutes per IP
- **Ratings**: 10 requests / hour per user
- **Comments**: 20 requests / hour per user

### Rate Limit Exceeded Response

**Response 429:**
```json
{
  "error": "Too many requests, please try again later"
}
```

---

## Pagination

All list endpoints support pagination:

**Query Parameters:**
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20)

**Response Format:**
```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

---

## Examples

### Complete Flow: Get Movie and Similar Movies

```bash
# 1. Login
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"john_doe","password":"password123"}'

# Response: {"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."}

# 2. Get Movie Details
curl -X GET http://localhost:5000/api/movies/1 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

# 3. Get Similar Movies
curl -X GET http://localhost:5000/api/movies/1/similar?limit=10 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

# 4. Rate Movie
curl -X POST http://localhost:5000/api/ratings \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{"movieId":1,"rating":5}'

# 5. Add Comment
curl -X POST http://localhost:5000/api/comments \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{"movieId":1,"content":"Great movie!"}'
```

---

## SDK/Client Libraries

### JavaScript/React Example

```javascript
// api.js
import axios from 'axios';

const API_BASE_URL = 'http://localhost:5000/api';

const api = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Content-Type': 'application/json'
  }
});

// Add token to requests
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Get similar movies
export const getSimilarMovies = async (movieId, limit = 10) => {
  const response = await api.get(`/movies/${movieId}/similar`, {
    params: { limit }
  });
  return response.data.movies;
};

// Search movies
export const searchMovies = async (query, limit = 20) => {
  const response = await api.get('/search', {
    params: { q: query, limit }
  });
  return response.data.movies;
};

// Rate movie
export const rateMovie = async (movieId, rating) => {
  const response = await api.post('/ratings', {
    movieId,
    rating
  });
  return response.data;
};
```

---

## Notes

1. **Authentication**: Most endpoints require JWT token in `Authorization` header
2. **Rate Limiting**: Be mindful of rate limits, especially for search and recommendation endpoints
3. **Pagination**: Always use pagination for large datasets
4. **Caching**: Similar movie results are cached for better performance
5. **Bad Word Filtering**: Comments are automatically filtered for inappropriate content

---

## Support

For issues or questions:
- GitHub Issues: [Link to repository]
- Email: support@movierec.com
