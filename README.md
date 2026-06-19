# Lost & Found Community App (Faisalabad)

Full-stack MERN application (MongoDB, Express, React, Node.js) for a community
Lost & Found platform. Includes a User Panel (post/browse/message) and an
Admin Panel (stats, pie charts, user/item management), rule-based AI matching,
and free GPS location pinning on an OpenStreetMap map (no API key or billing
account needed).

## Tech Stack
- **Backend:** Node.js, Express, MongoDB (Mongoose), JWT auth, Multer (image uploads)
- **Frontend:** React (Vite), Tailwind CSS, React Router, Recharts (charts), Leaflet + OpenStreetMap (map)
- **Matching engine:** Rule-based scoring (category + text similarity + GPS distance + date proximity) in `backend/utils/matching.js`

## Folder Structure
```
lost-and-found-app/
├── backend/        Express API (port 5000)
└── frontend/       React app (port 5173)
```

## Prerequisites
- Node.js 18+ and npm
- MongoDB running locally (`mongodb://localhost:27017`) OR a free MongoDB Atlas cluster
- No map API key needed — the map uses free OpenStreetMap tiles via Leaflet
- Recommended editor: **VS Code** (Visual Studio the full IDE does not run Node/React projects directly — use VS Code, or run the npm commands below from any terminal)

## 1. Backend Setup
```bash
cd backend
npm install
cp .env.example .env
# Edit .env: set MONGO_URI, JWT_SECRET, ADMIN_EMAIL, ADMIN_PASSWORD

npm run seed         # creates the admin account from .env
npm run seed:items   # (optional) inserts sample Faisalabad lost/found items + demo user

npm run dev          # starts API on http://localhost:5000
```

Default admin login (from `.env.example`, change before real use):
- Email: `admin@lostfound.com`
- Password: `Admin@123`

Demo user created by `npm run seed:items`:
- Email: `demo.user@lostfound.com`
- Password: `Demo@123`

## 2. Frontend Setup
```bash
cd frontend
npm install
cp .env.example .env
# Edit .env: set VITE_API_URL if your backend isn't on localhost:5000

npm run dev          # starts React app on http://localhost:5173
```

Open `http://localhost:5173` — you'll land on the home page with **User Login**
and **Admin Login** buttons, exactly as requested.

## Map / GPS Location
The Post Item form's map uses **Leaflet + OpenStreetMap** — completely free,
no API key, no billing account, no signup. Click anywhere on the map to drop
a pin, or use the "Use My Current Location" button to auto-pin via the
browser's GPS/location API. This works out of the box with zero configuration.

## How the AI Matching Works
When a new "lost" item is posted, the backend automatically compares it
against all open "found" items (and vice versa) using:
1. **Category match** (required, 25 pts)
2. **Title/description text similarity** via Dice coefficient (up to 35 pts)
3. **GPS distance** between pinned coordinates, or location text similarity if no GPS (up to 25 pts)
4. **Date proximity** between lost/found dates (up to 15 pts)

Matches scoring 30+ are shown to the user immediately after posting, and also
on each item's detail page.

## Key Features Implemented
- Landing page → choice of User Login or Admin Login
- User registration/login (JWT-based)
- User dashboard: stats (lost/found/reunited counts), recent listings
- Browse page with search, category/type/status/location filters, pagination
- Post Item form: Lost/Found toggle, category, date, text location, GPS map pin, photo upload (up to 5 images)
- My Posts page with All/Lost/Found/Reunited tabs, mark-as-reunited, delete
- Item detail page showing AI-suggested matches + direct messaging to the poster
- Messaging system (conversations + threaded messages)
- Admin dashboard: total users, lost/found/reunited counts, pie chart by category, pie chart lost-vs-found, bar chart by status, recent items table
- Admin Users page: view all users, enable/disable account, delete account
- Admin Items page: view/filter/delete any item platform-wide
- Fully responsive layout (collapsible sidebar on mobile, usable on Android phone browsers)

## Notes
- Uploaded images are stored in `backend/uploads/` and served at `/uploads/<filename>`. For production, switch to a cloud storage service (S3, Cloudinary, etc.) instead of local disk.
- Change `JWT_SECRET` and the seeded admin password before any real deployment.
- This is a responsive **web app** (works great in Android Chrome). It is not a packaged native Android app — that would require a separate React Native build, which is a different codebase.

## Troubleshooting: MongoDB Atlas connection times out (ETIMEOUT / querySrv)
Some networks (certain ISPs, routers, or campus/office WiFi) block the DNS
SRV lookup that `mongodb+srv://` connection strings rely on. If `npm run seed`
or `npm run dev` fails with an error like `querySrv ETIMEOUT`, switch to the
standard (non-SRV) connection string instead:

1. Run `nslookup -type=SRV _mongodb._tcp.<your-cluster>.mongodb.net` to get the three shard hostnames.
2. Run `nslookup -type=TXT <your-cluster>.mongodb.net` to get the replica set name.
3. Build the connection string manually:
   ```
   MONGO_URI=mongodb://<user>:<password>@<shard-00-00-host>:27017,<shard-00-01-host>:27017,<shard-00-02-host>:27017/lostfound?ssl=true&replicaSet=<replica-set-name>&authSource=admin&retryWrites=true&w=majority
   ```
4. If your password contains special characters (`@`, `#`, `%`, `/`, `:`), URL-encode them (e.g. `@` → `%40`) or the connection string won't parse correctly.
