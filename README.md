# 🌾 FarmLink

> A direct farm-to-retail marketplace. Farmers list their produce, retailers browse and order it, and both sides track every order in real time, with no middlemen in between.

FarmLink is a full-stack MERN application (MongoDB, Express, React, Node.js) with role-based accounts for farmers and retailers, an order lifecycle with status tracking, reviews, image uploads and live notifications over Socket.io.

---

## Table of contents

- [Features](#features)
- [Tech stack](#tech-stack)
- [Architecture](#architecture)
- [Project structure](#project-structure)
- [Getting started](#getting-started)
- [Environment variables](#environment-variables)
- [API reference](#api-reference)
- [Real-time events](#real-time-events)
- [Data models](#data-models)
- [Deployment](#deployment)
- [Security](#security)
- [Current status and known gaps](#current-status-and-known-gaps)
- [Roadmap](#roadmap)

---

## Features

**For farmers**
- Create, edit and delete crop listings with up to 5 images, pricing, units, minimum order quantity, harvest and expiry dates, grade, organic flag and shipping options
- Dashboard with listing and sales statistics
- Accept, reject, process, ship and deliver incoming orders
- Reply to reviews

**For retailers**
- Search the marketplace and filter by category, city, state, price range, grade, unit and organic
- Sort by newest, price or rating, with pagination
- Place orders with a delivery address, and cancel while an order is pending or accepted
- Dashboard with order and spending statistics
- Review farmers and crops after delivery

**Platform**
- JWT authentication with farmer, retailer and admin roles
- Forgot-password and reset-password by email
- In-app notifications, pushed live over Socket.io
- Order status history with a timestamp, note and author for every change
- Cloudinary image uploads for crops and avatars
- Admin endpoints for platform statistics and deactivating users

---

## Tech stack

| Layer | Technology |
| --- | --- |
| Frontend | React 18, React Router 6, Redux Toolkit, Axios, react-hot-toast, date-fns |
| Real-time | Socket.io (server and client) |
| Backend | Node.js, Express 4 |
| Database | MongoDB with Mongoose 8 |
| Auth | JSON Web Tokens, bcryptjs |
| Validation | express-validator and Mongoose validators |
| Security | Helmet, CORS, express-rate-limit |
| Uploads | Multer, Cloudinary |
| Email | Nodemailer |
| Payments | Stripe SDK (currently mocked, see [known gaps](#current-status-and-known-gaps)) |

---

## Architecture

```
┌──────────────────────────────────────────────────────┐
│                    React client                       │
│   Redux Toolkit · React Router · Socket.io client     │
└───────────────────────┬──────────────────────────────┘
                        │ HTTP (REST) / WebSocket
┌───────────────────────▼──────────────────────────────┐
│               Express + Socket.io server              │
│   Helmet · CORS · rate limit · JWT auth middleware    │
│   Routes → Controllers → Mongoose models              │
└───────────────────────┬──────────────────────────────┘
                        │ Mongoose
┌───────────────────────▼──────────────────────────────┐
│                       MongoDB                         │
│   Users · Crops · Orders · Reviews · Notifications    │
└──────────────────────────────────────────────────────┘

External services:  Cloudinary (images) · SMTP (email) · Stripe (payments, optional)
```

---

## Project structure

```
Farm_Link/
├── backend/
│   ├── config/          # MongoDB and Cloudinary setup
│   ├── controllers/     # auth, crop, order, review, notification, user
│   ├── middleware/      # auth (protect/authorize), validation, uploads, error handler
│   ├── models/          # User, Crop, Order, Review, Notification, Message
│   ├── routes/          # auth, users, crops, orders, reviews, notifications, uploads, payments, messages
│   ├── utils/           # email, logger, helpers, notification helper, socket handlers, seeder
│   ├── server.js        # App entry point
│   └── package.json
└── frontend/
    ├── public/
    └── src/
        ├── components/  # common, auth, farmer and retailer components
        ├── context/     # SocketContext
        ├── hooks/       # useAuth, useCrops, useOrders, useNotifications, useUtils
        ├── pages/       # Home, Marketplace, CropDetail, dashboards, Orders, Profile, auth pages
        ├── services/    # api.js (Axios client)
        ├── utils/       # constants, formatters, validators
        ├── store.js     # Redux Toolkit store and slices
        └── App.js       # Routes and route guards
```

---

## Getting started

### Prerequisites

- Node.js 18 or later
- A MongoDB instance (local or [MongoDB Atlas](https://www.mongodb.com/atlas))
- A [Cloudinary](https://cloudinary.com) account for image uploads (optional for browsing, required for uploading images)
- SMTP credentials for password-reset emails (optional)

### 1. Clone the repository

```bash
git clone https://github.com/mithunm-63/Farm_Link.git
cd Farm_Link
```

### 2. Set up the backend

```bash
cd backend
npm install
```

Create a `backend/.env` file (see [Environment variables](#environment-variables)). At minimum:

```env
MONGO_URI=mongodb://localhost:27017/farmlink
JWT_SECRET=replace-with-a-long-random-string
JWT_EXPIRE=7d
CLIENT_URL=http://localhost:3000
```

Optionally seed sample data, then start the server:

```bash
npm run seed   # WARNING: clears existing data first
npm run dev    # nodemon, auto-restarts on changes
```

### 3. Set up the frontend

```bash
cd frontend
npm install
```

Create a `frontend/.env` file:

```env
REACT_APP_API_URL=http://localhost:5000/api
REACT_APP_SOCKET_URL=http://localhost:5000
```

```bash
npm start
```

### 4. Open the app

| Service | URL |
| --- | --- |
| Frontend | http://localhost:3000 |
| Backend API | http://localhost:5000/api |
| Health check | http://localhost:5000/health |

### 5. Demo accounts (after seeding)

| Role | Email | Password |
| --- | --- | --- |
| Farmer | `farmer1@farmlink.com` | `password123` |
| Retailer | `retailer1@farmlink.com` | `password123` |

The seeder also creates `farmer2`, `farmer3` and `retailer2` accounts with the same password. These are for local development only.

> ⚠️ `npm run seed` deletes existing data before inserting the samples. Do not run it against a database you care about.

---

## Environment variables

### Backend (`backend/.env`)

| Variable | Required | Description |
| --- | --- | --- |
| `MONGO_URI` | Yes | MongoDB connection string (defaults to `mongodb://localhost:27017/farmlink`) |
| `JWT_SECRET` | Yes | Secret used to sign tokens. Use a long random value |
| `JWT_EXPIRE` | Yes | Token lifetime, e.g. `7d` |
| `CLIENT_URL` | Yes | Frontend origin, used for CORS, Socket.io and reset-password links (defaults to `http://localhost:3000`) |
| `PORT` | No | Server port (defaults to `5000`) |
| `NODE_ENV` | No | `development` enables request logging |
| `CLOUDINARY_CLOUD_NAME` | For uploads | Cloudinary cloud name |
| `CLOUDINARY_API_KEY` | For uploads | Cloudinary API key |
| `CLOUDINARY_API_SECRET` | For uploads | Cloudinary API secret |
| `SMTP_HOST` | For email | SMTP server host |
| `SMTP_PORT` | For email | SMTP server port |
| `SMTP_EMAIL` | For email | SMTP username |
| `SMTP_PASSWORD` | For email | SMTP password |
| `FROM_NAME` | No | Sender display name |
| `FROM_EMAIL` | No | Sender address (falls back to `SMTP_EMAIL`) |
| `STRIPE_SECRET_KEY` | No | Only needed once the real Stripe flow is enabled |

### Frontend (`frontend/.env`)

| Variable | Description |
| --- | --- |
| `REACT_APP_API_URL` | Backend API base URL, e.g. `http://localhost:5000/api` |
| `REACT_APP_SOCKET_URL` | Backend origin for Socket.io, e.g. `http://localhost:5000` |

Never commit `.env` files or real secrets.

---

## API reference

Base URL: `/api`. Send the token as `Authorization: Bearer <token>` for private routes. Responses use the shape `{ success, ... }`.

### Auth: `/api/auth`

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| POST | `/register` | Public | Register a farmer or retailer |
| POST | `/login` | Public | Log in and receive a JWT |
| GET | `/me` | Private | Current user |
| PUT | `/profile` | Private | Update profile |
| PUT | `/change-password` | Private | Change password |
| POST | `/forgot-password` | Public | Send a reset email |
| PUT | `/reset-password/:token` | Public | Reset password |

### Crops: `/api/crops`

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| GET | `/` | Public | List crops with filters, search and pagination |
| GET | `/featured` | Public | Featured crops |
| GET | `/my-listings` | Farmer | Your own listings |
| GET | `/farmer-stats` | Farmer | Dashboard statistics |
| GET | `/:id` | Public | Single crop |
| POST | `/` | Farmer | Create a listing |
| PUT | `/:id` | Farmer, Admin | Update a listing |
| DELETE | `/:id` | Farmer, Admin | Delete a listing |

Query parameters for `GET /api/crops`:

```
?search=tomato&category=vegetables&city=Mumbai&state=Maharashtra
&minPrice=10&maxPrice=500&isOrganic=true&unit=kg&grade=A
&page=1&limit=12&sort=-createdAt | pricePerUnit | -pricePerUnit | -rating
```

### Orders: `/api/orders`

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| POST | `/` | Retailer | Place an order |
| GET | `/` | Private | Your orders |
| GET | `/retailer-stats` | Retailer | Dashboard statistics |
| GET | `/:id` | Private | Order detail (involved parties only) |
| PUT | `/:id/status` | Private | Update order status |

Order status flow:

- Farmer: `pending → accepted | rejected`, `accepted → processing | shipped`, `shipped → delivered`
- Retailer: `pending | accepted → cancelled`

### Reviews: `/api/reviews`

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| GET | `/` | Public | List reviews (`?cropId=` or `?userId=`) |
| POST | `/` | Private | Create a review (delivered orders only) |
| PUT | `/:id/reply` | Private | Reply to a review |
| PUT | `/:id/helpful` | Private | Mark a review as helpful |
| DELETE | `/:id` | Private | Delete a review |

### Notifications: `/api/notifications`

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| GET | `/` | Private | List notifications |
| GET | `/unread-count` | Private | Unread count |
| PUT | `/read-all` | Private | Mark all as read |
| PUT | `/:id/read` | Private | Mark one as read |
| DELETE | `/clear-read` | Private | Delete all read notifications |
| DELETE | `/:id` | Private | Delete one notification |

### Users: `/api/users`

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| GET | `/farmers` | Public | List farmers |
| GET | `/search` | Public | Search users |
| GET | `/:id` | Public | Public profile |
| GET | `/retailers` | Farmer, Admin | List retailers |
| PUT | `/avatar` | Private | Update avatar |
| GET | `/admin/stats` | Admin | Platform statistics |
| PUT | `/:id/deactivate` | Admin | Deactivate a user |

### Uploads: `/api/uploads`

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| POST | `/image` | Private | Upload one image (field `image`) |
| POST | `/images` | Private | Upload up to 5 images (field `images`) |

### Payments: `/api/payments`

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| POST | `/create-intent` | Retailer | Create a payment intent (mock, see below) |
| POST | `/confirm` | Private | Mark an order's payment as paid |
| POST | `/webhook` | Public | Stripe webhook placeholder |

### Health

`GET /health` returns the API status, timestamp and environment.

---

## Real-time events

Socket.io shares the HTTP server and uses the same CORS origin (`CLIENT_URL`).

**Client → server**

```javascript
socket.emit('join', userId)                    // join your private room
socket.emit('typing', { roomId, userName })
socket.emit('stop_typing', { roomId })
```

**Server → client**

```javascript
socket.on('notification', (notification) => {})
socket.on('order_update', ({ orderNumber, status }) => {})
socket.on('user_typing', ({ userName }) => {})
socket.on('user_stop_typing', () => {})
```

---

## Data models

| Model | Key fields |
| --- | --- |
| **User** | `name`, `email`, hashed `password`, `role` (`farmer`, `retailer`, `admin`), `phone`, `avatar`, `location`; farmer fields (`farmName`, `farmSize`, `cropTypes`, `farmingType`); retailer fields (`businessName`, `businessType`, `gstNumber`, `preferredCrops`); `rating`, `isVerified`, `isActive` |
| **Crop** | `farmer`, `name`, `category`, `description`, `images`, `pricePerUnit`, `unit`, `minimumOrderQuantity`, `availableQuantity`, `harvestDate`, `expiryDate`, `grade`, `isOrganic`, `location`, `status` (`active`, `inactive`, `sold_out`, `expired`), `rating`, shipping fields |
| **Order** | `orderNumber`, `retailer`, `farmer`, `crop`, `cropSnapshot`, `quantity`, `subtotal`, `shippingCost`, `totalAmount`, `status`, `statusHistory`, `deliveryAddress`, `payment`, review flags |
| **Review** | `order`, `crop`, `reviewer`, `reviewee`, `rating` (1 to 5), `title`, `comment`, quality, packaging and communication ratings, `reply`, `helpfulVotes` |
| **Notification** | In-app notifications delivered per user |
| **Message** | Chat message schema (see [known gaps](#current-status-and-known-gaps)) |

Orders store a snapshot of the crop at order time, so later price changes do not alter past orders.

---

## Deployment

The app deploys as two pieces: the API (with MongoDB) and the static React build.

**Backend (for example Render or Railway)**
1. Create a web service with root directory `backend`.
2. Build command `npm install`, start command `npm start`.
3. Add the backend environment variables. Set `CLIENT_URL` to your deployed frontend URL.
4. Use a hosted MongoDB such as Atlas for `MONGO_URI`.

**Frontend (for example Vercel or Netlify)**
1. Set the root directory to `frontend`, with build command `npm run build` and output directory `build`.
2. Set `REACT_APP_API_URL` and `REACT_APP_SOCKET_URL` to the deployed backend URL.

Because the server uses WebSockets, pick a host that supports long-lived connections for the backend.

---

## Security

- Passwords hashed with bcrypt
- Stateless JWT authentication with configurable expiry
- Role-based access control (farmer, retailer, admin) on protected routes
- Owner-only crop editing and order access limited to the farmer and retailer involved
- Helmet security headers
- CORS restricted to `CLIENT_URL`
- Rate limiting on `/api/*`: 100 requests per 15 minutes per IP
- Request validation with express-validator and Mongoose schemas
- Secrets are read from environment variables

---

## Current status and known gaps

- **Payments are mocked.** `POST /api/payments/create-intent` returns a demo client secret. The real Stripe call is present only as commented-out code, and `/webhook` is a placeholder. `/confirm` marks an order as paid without verifying with Stripe. Do not use this for real money until the Stripe flow is implemented and verified.
- **Chat is not wired up yet.** A `Message` model, a `messages` route file and a chat widget exist, but the messages route is not mounted in `server.js`, so there are no `/api/messages` endpoints yet.
- **No automated tests or CI** are included in the repository yet.
- **No Docker setup or `.env.example` files** are included. Use the variable tables above.

---

## Roadmap

- [ ] Real Stripe payments with verified webhooks
- [ ] Mount and finish farmer-retailer chat
- [ ] Geospatial search for nearby farmers
- [ ] Analytics dashboards (revenue, top crops, seasonality)
- [ ] Automated tests and a CI pipeline
- [ ] Dockerfile and Docker Compose for local development
- [ ] Push notifications
- [ ] Multi-language support

---

## Contributing

Issues and pull requests are welcome. Please open an issue first for larger changes.

## License

No license file is currently included. The previous README stated MIT, so if that is your intent, add an MIT `LICENSE` file to make it effective.
