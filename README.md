# DevTinder – Backend

REST API for **DevTinder**, a "Tinder for developers" app. Developers create profiles, browse a feed of other developers, send *interested* / *ignored* signals, and match when interest is mutual. Premium memberships are sold through Razorpay.

## Tech Stack

- **Runtime:** Node.js, Express 5
- **Database:** MongoDB with Mongoose
- **Auth:** JWT stored in an HTTP-only cookie, passwords hashed with bcrypt
- **Payments:** Razorpay (orders + webhook signature verification)
- **Email:** AWS SES
- **Scheduling:** node-cron (daily notification job)
- **Tooling:** ESLint, nodemon

## Getting Started

### Prerequisites

- Node.js 18+
- A MongoDB instance (local or Atlas)
- Razorpay account (test keys are fine)
- AWS account with SES configured (only needed for email notifications)

### Installation

```bash
git clone <repo-url>
cd devTinder-BE
npm install
```

### Environment variables

Create a `.env` file in the project root:

```env
PORT=7777
MONGO_URI=mongodb://localhost:27017/devTinder
JWT_SECRET=your_jwt_secret

RAZORPAY_KEY_ID=your_key_id
RAZORPAY_KEY_SECRET=your_key_secret
RAZORPAY_WEBHOOK_KEY_SECRET=your_webhook_secret

AWS_REGION=ap-south-1
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
```

`PORT` defaults to `7777` if omitted.

### Run

```bash
npm run dev     # start with nodemon (auto-reload)
npm start       # start with node
npm run lint    # lint src/
npm run lint:fix
```

The server starts only after the MongoDB connection succeeds.

## Project Structure

```
src/
├── app.js            # Express app, middleware, route mounting, server bootstrap
├── config/           # MongoDB, Razorpay and AWS SES clients
├── middleware/
│   └── userAuth.js   # JWT cookie authentication
├── model/            # Mongoose models: user, connectionRequest, payment
├── routes/           # auth, profile, request, user (feed), payment
└── utils/            # validation, email sender, cron job, membership plans
```

## API Reference

Authenticated routes require the JWT cookie set by `/login`. CORS is configured for the frontend at `http://localhost:5173` with credentials enabled.

### Auth

| Method | Endpoint  | Description                       |
| ------ | --------- | --------------------------------- |
| POST   | `/signup` | Register a new user               |
| POST   | `/login`  | Log in and receive the auth cookie |
| POST   | `/logout` | Clear the auth cookie             |

### Profile

| Method | Endpoint         | Description                  |
| ------ | ---------------- | ---------------------------- |
| GET    | `/profile/view`  | Get the logged-in user's profile |
| PATCH  | `/profile/edit`  | Update profile fields        |

### Connections

| Method | Endpoint                          | Description                                                                 |
| ------ | --------------------------------- | --------------------------------------------------------------------------- |
| POST   | `/request/send/:status/:userId`   | Send a request; `status` is `interested` or `ignored`. Mutual interest becomes `accepted` ("It's a match!") |
| POST   | `/request/respond/:status/:requestId` | Respond to a received request; `status` is `accepted` or `rejected`     |

### Feed

| Method | Endpoint       | Description                                                                 |
| ------ | -------------- | --------------------------------------------------------------------------- |
| GET    | `/users/feed`  | Paginated list of users you haven't connected with (`?page=&limit=`, default limit 20) |

### Payments

| Method | Endpoint                    | Description                                                    |
| ------ | --------------------------- | -------------------------------------------------------------- |
| POST   | `/payment/create-order`     | Create a Razorpay order for a membership plan                  |
| POST   | `/payment/verify-payment`   | Razorpay webhook endpoint; verifies `X-Razorpay-Signature`     |
| GET    | `/payment/premium`          | Check whether the current user is a premium member             |

**Membership plans** (defined in [src/utils/contants.js](src/utils/contants.js)): Basic (₹599), Premium (₹999), Gold (₹1999).

## Background Jobs

A cron job in [src/utils/cronjob.js](src/utils/cronjob.js) runs daily at **08:00** and emails users about connection requests that are still pending from the previous day.

## License

ISC © Bharat Dahiya
