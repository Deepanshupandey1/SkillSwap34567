# Deploy PeerLearn on Render (single web service)

This setup serves the Vite frontend, Express API, and Socket.IO signaling from the same Render web service. This avoids hard-coded URLs pointing to an unrelated/old deployment. MongoDB must be MongoDB Atlas (transactions are required for credit settlement).

## 1. Push the project to GitHub

Upload the contents of this `peerlearn` folder to a GitHub repository. Keep the root `package.json`, `render.yaml`, `client/`, and `server/` at the repository root.

## 2. Create the Render service

1. In Render, choose **New + → Blueprint** and select the repository containing `render.yaml`. Alternatively create a **Web Service** with the settings below.
2. Runtime: **Node**.
3. Build command: `npm install && npm run build`
4. Start command: `npm start`
5. Health check path: `/api/health`

If using a regular Web Service instead of Blueprint, set the root directory to the repository root and add the environment variables in step 4.

## 3. Configure MongoDB Atlas

Create an Atlas cluster and database user. In Atlas Network Access, allow Render access (for a quick demo you may use `0.0.0.0/0`, but restrict it when possible). Copy the **SRV connection string** and replace the username/password placeholders. Use a replica-set-capable Atlas deployment because PeerLearn performs MongoDB transactions.

## 4. Set Render environment variables

Required:

- `NODE_ENV` = `production`
- `JWT_SECRET` = a long random secret (at least 32 characters; Render can generate one)
- `MONGODB_URI` = your MongoDB Atlas connection string
- `CLIENT_URL` = your exact deployed service URL, e.g. `https://peerlearn-xxxx.onrender.com` (no trailing slash). This is used by Express and Socket.IO CORS.
- `ENABLE_DEMO_AUTH` = `false` for a public deployment

Optional:

- `OPENAI_API_KEY` = only if you want AI-generated recommendations
- `OPENAI_MODEL` = e.g. `gpt-4.1-mini`
- `TURN_URLS`, `TURN_USERNAME`, `TURN_CREDENTIAL` = credentials from a TURN provider. TURN is recommended for reliable WebRTC calls across restrictive networks. Never commit secrets to GitHub.

Do not set `VITE_API_URL` or `VITE_SOCKET_URL` for this single-service deployment; the frontend automatically uses its own origin in production.

## 5. Deploy and verify

After deployment, open the Render URL and check `https://YOUR-SERVICE.onrender.com/api/health`. It should return `ok: true` and `database: connected`. Sign in from two separate browser profiles/devices and join the same accepted session to test chat, Socket.IO, and video. Camera access requires HTTPS, which Render provides.

## WebRTC note

STUN-only WebRTC works on many networks but is not guaranteed to work across all NATs/firewalls. If the mentor still cannot see the peer, configure a TURN provider using the optional variables above and inspect browser console logs. A successful app deployment does not by itself guarantee every network can establish a peer-to-peer media path.

## Free-instance note

Render free web services can spin down when idle; the first request after idle may take longer. Do not use local disk for persistent user data; MongoDB Atlas stores application data.
