# Render deployment runbook

The public source is [hcthakur2004/interview-studio](https://github.com/hcthakur2004/interview-studio). This application needs a long-lived Node web service; a static-site deployment will not serve its Express API or job worker.

Current Render service: **interview-studio**, Free tier, Singapore, at [https://interview-studio-sxvl.onrender.com](https://interview-studio-sxvl.onrender.com). Render assigned the `-sxvl` suffix. The application's production origin is set to this exact URL. Atlas allows the service's two documented outbound ranges; the database user has application write permissions rather than Atlas administrator access. The public health endpoint returned `status: ok` with MongoDB and both provider flags. A disposable account generated a real PostHog kit on the public host; question editing, pinning, regeneration and reload succeeded. The existing production database is selected explicitly through `MONGODB_DATABASE`; changing the code's default does not migrate or rename that data.

1. In Render, create a **Web Service** from the GitHub repository and select the **Free** instance. Set build command to `npm ci && npm run build`, start command to `npm start`, health path to `/api/health`, and use one instance. The project uses Node 22+.
2. Set server-side environment variables (never expose them in build logs or client variables):

   | Name | Value |
   | --- | --- |
   | `MONGODB_URI` | Atlas application-user URI, copied into Render's secret field |
   | `MONGODB_DATABASE` | `interview_studio` for a new installation; preserve the existing value on a live service unless migrating data |
   | `GEMINI_API_KEY` | Current Gemini key, ideally rotated before publishing |
   | `GEMINI_MODEL` | `gemini-3.5-flash-lite` |
   | `TAVILY_API_KEY` | Current Tavily key, ideally rotated before publishing |
   | `APP_ORIGIN` | Exact Render-assigned HTTPS origin, including any generated suffix, with no trailing slash |
   | `TRUST_PROXY` | `1` only if Render is the sole trusted reverse proxy in front of the app |

   The hosting provider supplies `PORT`; leave it unset unless the service requires an explicit value. Do not set `EVALUATE_ALLOW_LOCAL` on the public service.
3. Render lists a service's outbound IP ranges under **Connect → Outbound**. Add only those ranges to the Atlas project's network access list. Do not use `0.0.0.0/0`. Keep the local test IP only while local verification is needed. Prefer a MongoDB user limited to the application database rather than the initial project-wide administration user.
4. Deploy and visit the Render-assigned origin plus `/api/health`. Expect `status: ok`, `storage: mongodb`, and both provider flags true. Then register a disposable user on the public origin, generate one small real kit, edit and pin a question, regenerate its category, review a flashcard and reload. Verify saved edits and confidence survive. Check Render logs for errors without pasting logs containing secrets into public issues.
5. Keep the public URL in `README.md` current, and recheck the live origin after each release.

Render's [web service guide](https://render.com/docs/web-services) supports Node/Express Git repositories and health checks. Its [Free plan](https://render.com/docs/free) spins down after inactivity and has usage limits; a cold start is therefore expected during review. Render publishes [outbound IP range instructions](https://render.com/docs/outbound-ip-addresses), which allow Atlas access without opening the database to every address.
