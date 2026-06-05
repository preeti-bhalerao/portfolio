/**
 * Project structure and architecture documentation
 */

# Project Structure

```
face-photo-finder/
├── public/
│   └── models/                    # Face-api.js ML models
├── src/
│   ├── app/
│   │   ├── (pages)
│   │   │   ├── page.tsx          # Home page with upload/camera
│   │   │   ├── results/
│   │   │   │   └── page.tsx      # Results gallery
│   │   │   └── admin/
│   │   │       └── page.tsx      # Admin dashboard
│   │   ├── api/
│   │   │   ├── search/
│   │   │   │   └── route.ts      # Search endpoint
│   │   │   ├── download/
│   │   │   │   └── route.ts      # Batch download
│   │   │   └── admin/
│   │   │       ├── login/
│   │   │       │   └── route.ts  # Admin login
│   │   │       └── sources/
│   │   │           ├── route.ts  # List/add sources
│   │   │           └── [id]/
│   │   │               ├── reindex/route.ts
│   │   │               └── delete/route.ts
│   │   ├── globals.css           # Global styles
│   │   └── layout.tsx            # Root layout
│   ├── lib/
│   │   ├── db.ts                 # Prisma client
│   │   ├── adminAuth.ts          # Auth utilities
│   │   ├── faceRecognition.ts    # Face detection & embeddings
│   │   └── googleDrive.ts        # Drive API integration
│   └── components/               # (Reusable components)
├── prisma/
│   ├── schema.prisma             # Database schema
│   └── seed.ts                   # Initial data (optional)
├── .env.example                  # Environment template
├── .eslintrc.js                  # ESLint config
├── .gitignore                    # Git ignore rules
├── next.config.ts               # Next.js config
├── tailwind.config.ts           # Tailwind config
├── tsconfig.json                # TypeScript config
├── postcss.config.js            # PostCSS config
├── package.json                 # Dependencies
├── README.md                    # Documentation
└── DEPLOYMENT.md               # Deployment guide
```

## Architecture Overview

### Frontend Architecture

**Client-Side Flow:**
1. User uploads photo or captures with camera
2. Image sent to browser (not server)
3. Face-api.js detects faces locally
4. Embedding extracted (128-dimensional vector)
5. Embedding sent to backend for matching
6. Results displayed in gallery

**State Management:**
- React hooks for local state
- sessionStorage for search results
- localStorage for admin auth token

### Backend Architecture

**API Layer:**
- Next.js API Routes (serverless)
- RESTful endpoints
- JWT authentication for admin

**Database Layer:**
- PostgreSQL with Prisma ORM
- Vector embeddings stored as base64
- Indexed queries for fast search

**Google Drive Integration:**
- OAuth 2.0 for authentication
- Bulk folder indexing
- Metadata caching

### Face Recognition Pipeline

```
User Photo
    ↓
[Face Detection] → TinyFaceDetector
    ↓
[Face Alignment] → Landmarks
    ↓
[Feature Extraction] → 128-D Embedding
    ↓
[Similarity Matching] → Cosine Distance
    ↓
[Ranking] → Sort by Score
    ↓
[Results] → Return Top Matches
```

## Data Flow

### Indexing Flow (Admin)

```
Admin adds Drive link
    ↓
Verify folder access
    ↓
List all images in folder
    ↓
For each image (batch):
    ├─ Download image
    ├─ Detect faces
    ├─ Extract embeddings
    ├─ Store in database
    ├─ Calculate duplicate hash
    └─ Store metadata
    ↓
Update indexing status
    ↓
Ready for search
```

### Search Flow (User)

```
User uploads selfie
    ↓
Detect face in uploaded image
    ↓
Extract embedding
    ↓
Query database for similar embeddings
    ↓
Calculate cosine similarity
    ↓
Filter by threshold (0.6)
    ↓
Sort by match score
    ↓
Return top 100 results
    ↓
Display gallery
```

## Database Schema Relationships

```
PhotoSource (1) ── (N) Photo
PhotoSource (1) ── (N) IndexJob

Photo (1) ── (N) Face
Photo (1) ── (N) Download

SearchResult (references Photo and Face)
SystemConfig (singleton)
```

## API Endpoints

### Public Endpoints

```
POST   /api/search                  # Search by face
POST   /api/download               # Download photos as ZIP
```

### Admin Endpoints

```
POST   /api/admin/login            # Admin authentication
GET    /api/admin/sources          # List photo sources
POST   /api/admin/sources          # Add new source
POST   /api/admin/sources/[id]/reindex   # Re-index source
DELETE /api/admin/sources/[id]     # Delete source
```

## Performance Considerations

### Embedding Storage
- **Size**: ~512 bytes per embedding (128 floats × 4 bytes)
- **10,000 photos × 1 face**: ~5 MB
- **Compression**: Base64 encoding vs binary storage

### Search Speed
- **Query time**: O(N) where N = number of embeddings
- **Optimization**: Vector indexing (pgvector extension)
- **Target**: <3 seconds for 10,000+ embeddings

### Indexing Speed
- **Batch processing**: 10 images at a time
- **Speed**: ~10 photos/second on CPU
- **Background jobs**: Prevent blocking UI

## Security Architecture

### Authentication
- Admin password → JWT token
- Token validates all admin requests
- Token expiration: 24 hours

### Authorization
- Public: Search and download endpoints
- Protected: Admin endpoints require JWT

### Data Protection
- Embeddings stored as base64 (not reversible to face)
- No raw face images stored
- Metadata only

### Best Practices
- Environment variables for secrets
- SQL injection prevention (Prisma)
- Input validation on all endpoints
- Rate limiting recommended

## Scalability Strategy

### Current Capacity
- **Photos**: 10,000+
- **Faces per photo**: Multiple
- **Search latency**: <3 seconds
- **Concurrent users**: Limited by database

### Scaling Path
1. **Vector indexing** → pgvector + indexes
2. **Caching** → Redis for hot embeddings
3. **CDN** → CloudFlare for images
4. **Sharding** → Split by source/date
5. **Async jobs** → Background indexing

## Testing Strategy

### Unit Tests
- Face recognition utilities
- Similarity calculations
- API response formatting

### Integration Tests
- Database operations
- Google Drive API
- Search pipeline

### E2E Tests
- Upload → Search → Download flow
- Admin → Add Source → Index flow

## Monitoring

### Metrics to Track
- Average search latency
- Indexing speed
- API error rates
- Database query performance
- Disk usage

### Logging
- Search queries
- Download logs
- Indexing progress
- API errors

---

For detailed API documentation, see endpoint comments in route handlers.
For deployment details, see DEPLOYMENT.md
