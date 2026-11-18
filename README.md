# Minimal URL Shortener

A small URL shortener using HTML, CSS, JavaScript, Node.js, Express, and SQLite. Express serves the browser interface from `public/` and exposes the URL API.

For the AWS EC2, SSH, Node.js, Nginx, PM2, DNS, and HTTPS walkthrough, see [DEPLOYMENT.md](DEPLOYMENT.md).

## Run locally

Requirements: Node.js 22 or newer and npm.

```sh
cp .env.example .env
npm install
npm run dev
```

`cp` creates your private local settings file. `.env` is ignored by Git. `npm install` installs the dependencies recorded in `package.json` and `package-lock.json`. `npm run dev` starts Node's watch mode, which restarts the server when source files change. Use `npm start` for a normal server process.

The server listens on `http://localhost:3000`. SQLite creates `data/urls.sqlite` the first time the server starts. To make short links use a public hostname later, set `PUBLIC_BASE_URL` to that hostname's origin, for example `https://sho.rt`.

## API

Create a short link:

```sh
curl -X POST http://localhost:3000/api/urls \
  -H 'Content-Type: application/json' \
  -d '{"url":"https://example.com/a/long/page"}'
```

The response contains `code`, `originalUrl`, `shortUrl`, `clicks`, and `createdAt`.

Read a link and its click count:

```sh
curl http://localhost:3000/api/urls/CODE
```

Visiting `http://localhost:3000/CODE` returns an HTTP 302 redirect and adds one click. `GET /api/health` reports whether the server is responding.

URL creation accepts HTTP and HTTPS addresses up to 2048 characters and is limited to 10 requests per IP every 15 minutes. The limiter uses in-memory state, which is suitable for the single-process starter deployment; multiple app instances need a shared rate-limit store.

## Browser interface

Open `http://localhost:3000` to create a short link. The result includes a copy button and a click count. After someone visits the short link, select **Refresh stats** to fetch its current count.

## Project layout

```text
public/
  index.html                Shortening form and result view
  styles.css                Responsive layout and visual styles
  app.js                    Form submission, copy, and stats refresh
src/
  app.js                    Express middleware and route mounting
  server.js                 Environment loading and server lifecycle
  config.js                 Port, public URL, and database path
  controllers/               HTTP request and response logic
  db/database.js             SQLite connection and schema
  db/urlRepository.js        Storage operations used by controllers
  middleware/                Request limits and centralized errors
  routes/                    URL API routes
```

The controllers depend on `urlRepository.js`, not on SQL. A PostgreSQL version can replace that adapter while keeping the route and response shape stable.
# urlShorter
