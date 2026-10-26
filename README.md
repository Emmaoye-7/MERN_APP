# MERN Taskboard on Azure

A small React + Express + MongoDB task app configured to build and run as a single Azure App Service web app. The Express server serves the production React build and exposes the task API.

## Run locally

1. Install Node.js 20 or newer.
2. Create a MongoDB Atlas cluster and database user. In Atlas Network Access, allow the IP address used by your app; for initial local testing, add your current IP address.
3. Copy `.env.example` to `.env` and replace the placeholder connection string with the Atlas connection string. URL-encode special characters in the database username/password.
4. From this folder, run:

   ```powershell
   npm install
   npm run dev
   ```

5. Open the Vite URL printed in the terminal (normally `http://localhost:5173`). The API is available at `http://localhost:8080/api/health`.

The `.env` file is ignored by Git. Never commit database credentials.

## Production build

```powershell
npm install
npm run build
npm start
```

Set `MONGODB_URI` before running the production server. Azure supplies `PORT` automatically. The app exits with a clear error if it cannot connect to MongoDB.

## Deploy to Azure App Service

1. Create an Azure account if needed, then create a **Web App** in Azure Portal. Choose **Code**, **Node 20 LTS** (or newer supported Node.js), and a Linux App Service plan.
2. In the app's **Configuration > Application settings**, add:
   - `MONGODB_URI`: the Atlas connection string for the app's database user.
   - Do not set a fixed `PORT`; App Service provides it.
3. In MongoDB Atlas, add the App Service outbound IP addresses to **Network Access**. Find these under the web app's **Properties** (`Outbound IP addresses` and `Additional outbound IP addresses`). For a classroom/demo deployment only, Atlas `0.0.0.0/0` permits connections from anywhere and is less secure; use it only if you understand the exposure and protect the database with a strong unique password.
4. Deploy the repository containing this folder using **Deployment Center** and GitHub Actions. Set the deployment source to the repository/branch where `MERN_APP` is located. The included workflow builds and deploys this subfolder.
5. Ensure deployment installs dependencies and runs `npm run build` from this folder. Set the App Service startup command to `npm start` if it is not detected automatically. The root `start` script starts the Express server, and the root `build` script builds the React frontend.
6. After deployment, open `https://<your-app-name>.azurewebsites.net/api/health`; confirm the JSON reports `"database": "connected"`, then open the site and try adding, completing, and deleting a task.

The workflow expects these GitHub Actions repository secrets:

- `AZURE_WEBAPP_NAME`: the Azure Web App name.
- `AZURE_WEBAPP_PUBLISH_PROFILE`: the publish profile downloaded from the Azure Web App's **Overview** page.

For Azure's built-in GitHub Actions workflow, set the app's `MONGODB_URI` in App Service Configuration as described above. If you choose another deployment method, its build commands must run in `MERN_APP` and include the generated `client/dist` in the deployed package. Avoid committing generated `node_modules`.

## API

- `GET /api/health` — process and MongoDB connectivity status.
- `GET /api/tasks` — list tasks.
- `POST /api/tasks` — create a task with `{ "title": "..." }`.
- `PATCH /api/tasks/:id` — update completion with `{ "completed": true }`.
- `DELETE /api/tasks/:id` — delete a task.
