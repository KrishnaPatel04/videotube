# VideoTube

A backend service for a YouTube-style video platform, built with Node.js, Express, and MongoDB. It handles authentication, video publishing, engagement (likes, comments, tweets), subscriptions, playlists, and channel analytics, with media storage on Cloudinary.

## Features

- **Authentication** — JWT-based access/refresh tokens, HTTP-only cookies, secure login/logout flow, password change
- **User management** — registration with avatar/cover image upload, profile updates, channel profile with subscriber counts (via aggregation pipelines), watch history
- **Videos** — publish, update, delete, toggle publish status, paginated listing with search/sort/filter
- **Engagement** — like/unlike videos, comments, and tweets; paginated comments per video
- **Tweets** — short-form posts tied to a user (create, update, delete, list by user)
- **Subscriptions** — subscribe/unsubscribe to channels, list subscribers and subscribed channels
- **Playlists** — create, update, delete playlists; add/remove videos
- **Dashboard** — channel stats (views, subscribers, video count, likes) and channel video list
- **Health check** — simple uptime/status endpoint
- **Media handling** — temporary local storage via Multer, uploaded to Cloudinary, temp files cleaned up after upload

## Tech Stack

| Layer | Technology |
|---|---|
| Runtime | Node.js |
| Framework | Express.js |
| Database | MongoDB with Mongoose |
| Auth | JWT (jsonwebtoken), bcrypt |
| File uploads | Multer (local temp) → Cloudinary (permanent storage) |
| Pagination | mongoose-aggregate-paginate-v2 |
| Middleware | cors, cookie-parser |

## Project Structure

```
src/
├── controllers/
│   ├── user.controller.js
│   ├── video.controller.js
│   ├── comment.controller.js
│   ├── like.controller.js
│   ├── tweet.controller.js
│   ├── subscription.controller.js
│   ├── playlist.controller.js
│   ├── dashboard.controller.js
│   └── healthcheck.controller.js
├── models/
│   ├── user.model.js
│   ├── video.model.js
│   ├── comment.model.js
│   ├── like.model.js
│   ├── tweet.model.js
│   ├── subscription.model.js
│   └── playlist.model.js
├── routes/
│   ├── user.routes.js
│   ├── video.routes.js
│   ├── comment.routes.js
│   ├── like.routes.js
│   ├── tweet.routes.js
│   ├── subscription.routes.js
│   ├── playlist.routes.js
│   ├── dashboard.routes.js
│   └── healthcheck.routes.js
├── middlewares/
│   ├── auth.middleware.js       # verifyJWT
│   ├── multer.middleware.js     # local temp storage
│   └── error.middleware.js      # centralized error handler
├── utils/
│   ├── ApiError.js
│   ├── ApiResponse.js
│   ├── asyncHandler.js
│   └── cloudinary.js
├── db/
│   └── index.js                 # MongoDB connection
├── app.js                       # Express app + middleware + routes
└── index.js                     # Entry point, loads env, starts server
```

## Prerequisites

- Node.js (v18+ recommended)
- MongoDB instance (local or Atlas)
- A Cloudinary account (cloud name, API key, API secret)

## Environment Variables

Create a `.env` file in the project root:

```env
PORT=8000
MONGODB_URI=mongodb+srv://<user>:<password>@cluster.mongodb.net
CORS_ORIGIN=http://localhost:5173

ACCESS_TOKEN_SECRET=your_access_token_secret
ACCESS_TOKEN_EXPIRY=1d
REFRESH_TOKEN_SECRET=your_refresh_token_secret
REFRESH_TOKEN_EXPIRY=10d

CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
```

## Installation

```bash
git clone <repository-url>
cd videotube
npm install
npm run dev
```

The server starts at `http://localhost:8000` (or the port set in `.env`).

## API Reference

All routes are prefixed with `/api/v1`. Routes marked 🔒 require a valid access token (cookie or `Authorization: Bearer <token>` header).

### Healthcheck
| Method | Endpoint | Description |
|---|---|---|
| GET | `/healthcheck` | Returns service status |

### Users — `/users`
| Method | Endpoint | Description |
|---|---|---|
| POST | `/register` | Register user (multipart: `avatar`, `coverImage`) |
| POST | `/login` | Log in, sets access/refresh token cookies |
| POST | `/logout` 🔒 | Log out, clears cookies |
| POST | `/refresh-token` | Issue new access token from refresh token |
| POST | `/change-password` 🔒 | Change current password |
| GET | `/current-user` 🔒 | Get logged-in user's profile |
| PATCH | `/update-account` 🔒 | Update `fullName`/`email` |
| PATCH | `/avatar` 🔒 | Update avatar (multipart: `avatar`) |
| PATCH | `/cover-image` 🔒 | Update cover image (multipart: `coverImage`) |
| GET | `/c/:username` 🔒 | Get channel profile with subscriber counts |
| GET | `/history` 🔒 | Get watch history |

### Videos — `/videos`
| Method | Endpoint | Description |
|---|---|---|
| GET | `/` 🔒 | List videos (paginated, searchable, sortable) |
| POST | `/` 🔒 | Publish a video (multipart: `videoFile`, `thumbnail`) |
| GET | `/:videoId` 🔒 | Get video by ID |
| PATCH | `/:videoId` 🔒 | Update video details (multipart: `thumbnail`) |
| DELETE | `/:videoId` 🔒 | Delete a video |
| PATCH | `/toggle/publish/:videoId` 🔒 | Toggle publish status |

### Comments — `/comments`
| Method | Endpoint | Description |
|---|---|---|
| GET | `/:videoId` 🔒 | Get paginated comments for a video |
| POST | `/:videoId` 🔒 | Add a comment |
| PATCH | `/c/:commentId` 🔒 | Update a comment |
| DELETE | `/c/:commentId` 🔒 | Delete a comment |

### Likes — `/likes`
| Method | Endpoint | Description |
|---|---|---|
| POST | `/toggle/v/:videoId` 🔒 | Toggle like on a video |
| POST | `/toggle/c/:commentId` 🔒 | Toggle like on a comment |
| POST | `/toggle/t/:tweetId` 🔒 | Toggle like on a tweet |
| GET | `/videos` 🔒 | Get all videos liked by current user |

### Tweets — `/tweets`
| Method | Endpoint | Description |
|---|---|---|
| POST | `/` 🔒 | Create a tweet |
| GET | `/user/:userId` 🔒 | Get tweets by user |
| PATCH | `/:tweetId` 🔒 | Update a tweet |
| DELETE | `/:tweetId` 🔒 | Delete a tweet |

### Subscriptions — `/subscriptions`
| Method | Endpoint | Description |
|---|---|---|
| POST | `/c/:channelId` 🔒 | Toggle subscription to a channel |
| GET | `/c/:channelId` 🔒 | Get subscribed channels list |
| GET | `/u/:subscriberId` 🔒 | Get a channel's subscribers |

### Playlists — `/playlists`
| Method | Endpoint | Description |
|---|---|---|
| POST | `/` 🔒 | Create a playlist |
| GET | `/:playlistId` 🔒 | Get playlist by ID |
| PATCH | `/:playlistId` 🔒 | Update playlist |
| DELETE | `/:playlistId` 🔒 | Delete playlist |
| PATCH | `/add/:videoId/:playlistId` 🔒 | Add video to playlist |
| PATCH | `/remove/:videoId/:playlistId` 🔒 | Remove video from playlist |
| GET | `/user/:userId` 🔒 | Get all playlists of a user |

### Dashboard — `/dashboard`
| Method | Endpoint | Description |
|---|---|---|
| GET | `/stats` 🔒 | Channel stats — views, subscribers, videos, likes |
| GET | `/videos` 🔒 | All videos uploaded by the channel |

## Authentication Flow

1. `POST /users/register` creates a user with a hashed password (bcrypt, via a Mongoose `pre("save")` hook) and uploads avatar/cover image to Cloudinary.
2. `POST /users/login` verifies credentials and returns an access token (short-lived) and a refresh token (long-lived), both set as HTTP-only cookies.
3. Protected routes run through `verifyJWT`, which reads the access token from the cookie or `Authorization` header, verifies it, and attaches the user to `req.user`.
4. `POST /users/refresh-token` issues a new access token using a valid refresh token, without requiring the user to log in again.

## File Upload Flow

1. Multer saves incoming files to a local temp directory.
2. The controller passes the local path to `uploadOnCloudinary`, which uploads the file and returns the Cloudinary URL.
3. The local temp file is deleted after the upload (success or failure).

## Error Handling

All controllers are wrapped in `asyncHandler` to forward errors to Express's error pipeline. Errors are normalized into a consistent `ApiError` shape (`statusCode`, `message`, `errors`, `success: false`) by a centralized error-handling middleware, and successful responses use a consistent `ApiResponse` shape (`statusCode`, `data`, `message`, `success`).

## Scripts

```bash
npm run dev     # start with nodemon
npm start       # start in production
```

