# URL-Shortner
Full-stack URL shortener with custom aliases, click analytics, and QR code generation. Features Redis caching, rate limiting, and consistent hashing for scalability. Built with React, Node.js, Express, and MongoDB.

# 🔗 SnapURL — Full-Stack URL Shortener

A production-ready URL shortener with custom aliases, click analytics, QR code generation, and Redis caching. Built with a focus on scalability using consistent hashing and rate limiting.

![Tech Stack](https://img.shields.io/badge/Frontend-React-61DAFB?style=flat&logo=react)
![Tech Stack](https://img.shields.io/badge/Backend-Node.js-339933?style=flat&logo=node.js)
![Tech Stack](https://img.shields.io/badge/Database-MongoDB-47A248?style=flat&logo=mongodb)
![Tech Stack](https://img.shields.io/badge/Cache-Redis-DC382D?style=flat&logo=redis)

---

## ✨ Features

- **URL Shortening** — Generate short, shareable links from long URLs instantly
- **Custom Aliases** — Choose your own slug (e.g. `snapurl.io/my-link`)
- **Click Analytics** — Track total clicks, unique visitors, referrers, devices, and geographic data
- **QR Code Generation** — Auto-generate downloadable QR codes for every short URL
- **Redis Caching** — Frequently accessed URLs served from cache for sub-millisecond redirects
- **Rate Limiting** — Prevents abuse with IP-based rate limiting (100 requests/hour per IP)
- **Link Expiry** — Optional TTL for links that auto-expire after a set time
- **Dashboard** — Visual analytics with charts for clicks over time, top referrers, and device breakdown

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React, Recharts (analytics charts), TailwindCSS |
| Backend | Node.js, Express |
| Database | MongoDB (persistent storage) |
| Caching | Redis (redirect cache + rate limit counters) |
| QR Codes | `qrcode` npm package |
| Short ID | NanoID (collision-resistant unique IDs) |

---

## 📁 Project Structure

```
snapurl/
├── client/                         # React frontend
│   ├── src/
│   │   ├── components/
│   │   │   ├── UrlShortener.jsx     # Main shortening form
│   │   │   ├── Dashboard.jsx        # Analytics dashboard
│   │   │   ├── AnalyticsChart.jsx   # Recharts click graph
│   │   │   ├── QRModal.jsx          # QR code display & download
│   │   │   └── LinkCard.jsx         # Individual link display
│   │   ├── pages/
│   │   │   ├── Home.jsx
│   │   │   └── Stats.jsx            # Per-link stats page
│   │   └── App.jsx
│   └── package.json
│
├── server/                         # Node.js backend
│   ├── src/
│   │   ├── routes/
│   │   │   ├── url.js              # Shorten, redirect, delete routes
│   │   │   └── analytics.js        # Stats & analytics routes
│   │   ├── models/
│   │   │   └── Url.js              # MongoDB URL schema
│   │   ├── middleware/
│   │   │   └── rateLimiter.js      # Redis-based rate limiter
│   │   ├── services/
│   │   │   ├── cache.js            # Redis get/set helpers
│   │   │   └── qrGenerator.js      # QR code generation service
│   │   ├── utils/
│   │   │   └── generateId.js       # NanoID short code generator
│   │   └── index.js
│   └── package.json
│
└── docker-compose.yml              # MongoDB + Redis setup
```

---

## ⚙️ How It Works

### Shortening Flow

```
User submits long URL
        ↓
Check if custom alias is taken (MongoDB lookup)
        ↓
Generate NanoID (6 chars) if no alias provided
        ↓
Save to MongoDB { shortCode, longUrl, createdAt, clicks: [] }
        ↓
Return short URL to user + QR code
```

### Redirect Flow (Optimized)

```
User hits short URL → Check Redis cache
                           ↓
              Cache HIT → Redirect instantly (< 1ms)
              Cache MISS → Query MongoDB → Cache result → Redirect
                                ↓
                      Log click (async) → { timestamp, ip, userAgent, referrer }
```

### Redis Caching Strategy

- **Cache on read** — Short code → long URL mapping cached on first access
- **TTL** — Cache entries expire after 24 hours (configurable)
- **Invalidation** — Cache entry deleted on URL update or deletion
- **Rate limiting** — Redis `INCR` + `EXPIRE` used for sliding window rate limiting per IP

### Consistent Hashing Concept

For horizontal scaling across multiple Redis nodes, consistent hashing ensures:
- Each short code maps to the same Redis node deterministically
- Adding/removing nodes only remaps a small fraction of keys
- No thundering herd on cache misses

---

## 🚀 Getting Started

### Prerequisites

- Node.js v18+
- MongoDB
- Redis

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/snapurl.git
cd snapurl

# Start MongoDB and Redis using Docker
docker-compose up -d

# Install server dependencies
cd server
npm install

# Install client dependencies
cd ../client
npm install
```

### Environment Variables

Create a `.env` in `/server`:

```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/snapurl
REDIS_URL=redis://localhost:6379
BASE_URL=http://localhost:5000
CLIENT_URL=http://localhost:5173
RATE_LIMIT_MAX=100
RATE_LIMIT_WINDOW=3600
```

### Running the App

```bash
# Start backend
cd server
npm run dev

# Start frontend (new terminal)
cd client
npm run dev
```

Open `http://localhost:5173` to use the app.

---

## 🔌 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/url/shorten` | Shorten a URL |
| `GET` | `/:shortCode` | Redirect to original URL |
| `GET` | `/api/url/:shortCode` | Get URL metadata |
| `DELETE` | `/api/url/:shortCode` | Delete a short URL |
| `GET` | `/api/analytics/:shortCode` | Get click analytics |
| `GET` | `/api/url/:shortCode/qr` | Get QR code as PNG |

### Request / Response Example

**POST** `/api/url/shorten`
```json
{
  "longUrl": "https://www.example.com/very/long/path?param=value",
  "customAlias": "my-link",    // optional
  "expiresIn": 7               // optional, days
}
```

**Response:**
```json
{
  "shortUrl": "http://localhost:5000/my-link",
  "shortCode": "my-link",
  "qrCode": "data:image/png;base64,...",
  "expiresAt": "2024-08-01T00:00:00.000Z"
}
```

---

## 📊 Analytics Schema

Each click on a short URL logs:

```json
{
  "timestamp": "2024-07-01T10:30:00Z",
  "ip": "192.168.1.1",
  "country": "India",
  "device": "mobile",
  "browser": "Chrome",
  "referrer": "https://twitter.com"
}
```

---

## 🧠 Key Concepts Implemented

- **Redis caching** — Redirect latency reduced from ~50ms (DB query) to < 1ms (cache hit)
- **Rate limiting** — Sliding window algorithm using Redis atomic `INCR` operations
- **Consistent hashing** — Theoretical basis for distributing cache across multiple Redis nodes
- **Async click logging** — Redirect happens first, analytics logged in background to minimize latency
- **NanoID** — URL-safe, collision-resistant 6-character IDs (64^6 = ~68 billion combinations)
- **TTL-based expiry** — MongoDB TTL indexes auto-delete expired documents

---

## 🛣️ Future Improvements

- [ ] User authentication and personal link dashboard
- [ ] Bulk URL shortening via CSV upload
- [ ] API keys for programmatic access
- [ ] Custom domain support
- [ ] A/B testing with multiple destination URLs

---

## 📄 License

MIT License — feel free to use, modify, and distribute.
