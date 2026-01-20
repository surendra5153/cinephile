# 🎬 Movie Recommendation System

A live, online movie recommendation website with intelligent similarity-based recommendations using knowledge graphs and advanced DSA algorithms.

## 📋 System Architecture

### Tech Stack
- **Frontend**: React with modern UI components
- **Backend**: Node.js + Express
- **Database**: SQLite (with PostgreSQL-compatible schema for production)
- **Knowledge Graph**: Custom implementation using adjacency lists
- **Search**: Trie-based autocomplete
- **Authentication**: JWT (username-only)

### Architecture Overview

```
┌─────────────────┐
│   React Client  │
└────────┬────────┘
         │ HTTP/REST
┌────────▼────────────────────┐
│   Express Backend API       │
│  ┌──────────────────────┐   │
│  │ Authentication (JWT) │   │
│  │ Search (Trie)        │   │
│  │ Recommendation Engine│   │
│  │ Knowledge Graph      │   │
│  └──────────────────────┘   │
└────────┬────────────────────┘
         │
┌────────▼────────┐
│  SQLite Database│
│  + Graph Store  │
└─────────────────┘
```

## 🏗️ Database Schema

### Tables
- **users**: username, password_hash, created_at
- **movies**: id, title, description, release_year, poster_url, avg_rating
- **genres**: id, name
- **actors**: id, name
- **directors**: id, name
- **platforms**: id, name, logo_url
- **movie_genres**: movie_id, genre_id
- **movie_actors**: movie_id, actor_id
- **movie_directors**: movie_id, director_id
- **movie_platforms**: movie_id, platform_id
- **ratings**: user_id, movie_id, rating (1-5), created_at
- **comments**: id, user_id, movie_id, content, created_at

## 🧠 Knowledge Graph

### Nodes
- Movie
- Genre
- Actor
- Director
- Platform
- User

### Relationships
- MOVIE → BELONGS_TO → GENRE
- ACTOR → ACTED_IN → MOVIE
- DIRECTOR → DIRECTED → MOVIE
- MOVIE → AVAILABLE_ON → PLATFORM
- USER → RATED → MOVIE

## 🔍 Similar Movie Recommendation Algorithm

### Algorithm Steps:

1. **Graph Construction**: Build adjacency list where each movie is a node
2. **Similarity Calculation**: 
   - Same genre → +3 points
   - Same actor → +2 points
   - Same director → +2 points
   - Same platform → +1 point
   - Similar rating (within 0.5) → +2 points
3. **Graph Traversal**: Use BFS to explore related movies
4. **Scoring**: Calculate weighted similarity scores
5. **Ranking**: Use Max Heap (Priority Queue) to get top 5-10 similar movies

### Example:
```
Movie A: [Genre: Action, Thriller], [Actor: Tom], [Director: John], [Platform: Netflix], Rating: 4.5
Movie B: [Genre: Action], [Actor: Tom], [Director: Mike], [Platform: Netflix], Rating: 4.3

Similarity Score: 
- Same Genre (Action) = +3
- Same Actor (Tom) = +2
- Same Platform (Netflix) = +1
- Similar Rating = +2
Total = 8 points
```

## 📊 DSA Usage

- **Graph (Adjacency List)**: Knowledge graph for recommendations
- **Trie**: Fast search autocomplete
- **HashMap**: User lookup, rating cache, similarity scores
- **Priority Queue (Max Heap)**: Top-K similar movies, trending movies
- **HashSet**: Bad word filtering
- **Sorting**: Top movies of decade

## 🚀 Getting Started

### Prerequisites
- Node.js (v16 or higher)
- npm or yarn

### Backend Setup
```bash
cd backend
npm install

# Initialize database (first time only)
npm run init-db

# Seed database with 100+ movies
npm run seed

# Start development server
npm run dev
```

The backend server will run on `http://localhost:5000`

### Frontend Setup
```bash
cd frontend
npm install

# Start development server
npm start
```

The frontend will run on `http://localhost:3000`

### Environment Variables
Create a `.env` file in the `backend` directory:
```
PORT=5000
JWT_SECRET=your-secret-key-change-in-production
NODE_ENV=development
```

## 📁 Project Structure

```
movie-rec-system/
├── backend/
│   ├── src/
│   │   ├── models/          # Database models
│   │   ├── dsa/             # DSA implementations
│   │   │   ├── Trie.js
│   │   │   ├── Graph.js
│   │   │   ├── PriorityQueue.js
│   │   │   └── BadWordFilter.js
│   │   ├── services/        # Business logic
│   │   │   ├── authService.js
│   │   │   ├── recommendationService.js
│   │   │   └── searchService.js
│   │   ├── routes/          # API routes
│   │   ├── middleware/      # Auth, validation
│   │   └── app.js
│   └── database/
│       └── init.sql
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/
│   │   └── App.js
│   └── public/
└── README.md
```

## 🔐 Security Features

- JWT authentication (username-only)
- Password hashing (bcrypt)
- Input validation (express-validator)
- Rate limiting (100 requests / 15 minutes per IP)
- Bad word filtering for comments (Trie + HashSet)
- SQL injection prevention (parameterized queries)
- XSS prevention (content sanitization)

## 📖 Documentation

- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Complete system architecture, database schema, and algorithm explanations
- **[API_DOCUMENTATION.md](./API_DOCUMENTATION.md)** - Complete API reference with examples

## 🎯 Key Features

### ✅ Authentication
- Username-only login (no email/phone)
- JWT token-based authentication
- Session management

### ✅ Home Page
- Top Movies of the Decade
- Trending Movies
- Genre-based categorized sections
- Separate Anime section
- Netflix-style UI

### ✅ Search Functionality
- Search by movie name, genre, actor, director, platform
- Trie-based autocomplete
- Fast lookup using Trie/HashMap/Inverted Index

### ✅ Movie Details
- Complete movie information
- Cast & crew
- Genres and release year
- Streaming platforms with watch links
- User ratings (1-5 stars)

### ✅ Comments
- User comments with bad-word filtering
- Trie/HashSet/Regex for filtering
- Replace abusive words or block comments

### ✅ Ratings
- One rating per user per movie
- Efficient storage
- Average rating and total count

### ✅ Similar Movie Recommendation (IMPORTANT)
- Content-based filtering
- Knowledge graph traversal (BFS/DFS)
- Multi-factor similarity scoring:
  - Same genre: +3
  - Same actor: +2
  - Same director: +2
  - Same platform: +1
  - Rating similarity: +2
- Priority Queue (Max Heap) for ranking
- Top 5-10 similar movies

## 🧠 Knowledge Graph

The system uses a knowledge graph to represent relationships:
- **Nodes**: Movies, Genres, Actors, Directors, Platforms, Users
- **Relationships**: BELONGS_TO, ACTED_IN, DIRECTED, AVAILABLE_ON, RATED
- **Implementation**: Adjacency list with weights

## 📊 DSA Usage

- **Graph (Adjacency List)**: Knowledge graph for recommendations
- **Trie**: Fast search autocomplete
- **HashMap**: Users, ratings, similarity scores
- **Priority Queue (Max Heap)**: Trending & similar movies
- **HashSet**: Bad word filtering
- **Sorting Algorithms**: Top movies of decade

## 📝 Algorithm Explanation

See [ARCHITECTURE.md](./ARCHITECTURE.md) for detailed step-by-step algorithm explanations with examples.

## 🔧 Development

### Running Tests
```bash
# Backend tests (if implemented)
cd backend
npm test

# Frontend tests
cd frontend
npm test
```

### Database Management
```bash
# Reset database
cd backend
rm database/movies.db
npm run init-db
npm run seed
```

## 📈 Production Deployment

### Recommendations
1. **Database**: Use PostgreSQL instead of SQLite
2. **Caching**: Add Redis for caching
3. **Graph DB**: Consider Neo4j for knowledge graph
4. **Search**: Migrate to Elasticsearch for advanced search
5. **CDN**: Use CDN for static assets
6. **Monitoring**: Add logging and monitoring (Winston, Prometheus)

See [ARCHITECTURE.md](./ARCHITECTURE.md) for detailed scalability strategy.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 📄 License

ISC

## 🙏 Acknowledgments

Built with:
- React
- Node.js & Express
- SQLite/PostgreSQL
- Custom DSA implementations