# Job Hunter – Instant Career-Page Alerts (n8n + Telegram)

Get a Telegram ping **within 30 minutes** of a matching job being posted on a company's own careers page — often hours or days before it shows up on LinkedIn or Naukri. Built for job seekers targeting Global Capability Centres (GCCs) and other large employers, and easy to adapt to any role or country.

Free and open source (MIT). Runs on your own n8n. No scraping logins, no paid APIs.

---

## What it does

- Every 30 minutes it checks the careers pages of the companies on your list.
- It finds each company's job board automatically (Greenhouse, Lever, Ashby). You can add Workday sites by URL.
- It keeps only jobs that match **your roles** (in the title *or* the job description) and **your locations**.
- It sends new matches to your phone via your own Telegram bot. Nothing goes to your inbox.

## What you need

- **n8n**, self-hosted (free) or n8n Cloud (paid, has a free trial). Tested on n8n 2.41. If you don't have it yet, Step 0 below sets it up.
- **A Telegram account.**
- **About 30 minutes** if you're installing n8n for the first time, or 15 if you already have it.

---

## Setup

### 0. Install n8n (skip if you already have it)

**Option A: n8n Cloud (easiest, no install)**

1. Sign up at **n8n.io**. There's a free trial, then a monthly fee.
2. You're done. It runs 24/7, so your computer doesn't need to stay on. Go to Step 1.

**Option B: Free, on your own computer with Docker**

1. Install **Docker Desktop** from **docker.com** (Mac, Windows or Linux) and open it. Wait until it says Docker is running.
2. Open **Terminal** (Mac/Linux) or **PowerShell** (Windows) and run these two commands one after another:

   ```bash
   docker volume create n8n_data
   ```

   ```bash
   docker run -d --name n8n --restart unless-stopped -p 127.0.0.1:5678:5678 -e GENERIC_TIMEZONE="Asia/Kolkata" -e TZ="Asia/Kolkata" -e N8N_RUNNERS_TASK_TIMEOUT=300 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
   ```

   Change `Asia/Kolkata` to your time zone if you're outside India.

3. Open **http://localhost:5678** in your browser.
4. Create your n8n owner account (email and password). This account stays on your computer.

> **Your computer must be on and awake** for alerts to arrive. To run 24/7, you can run the same Docker commands on a small cloud server (for example Oracle Cloud's Always Free tier, or a ₹600/month VPS).
>
> **On a cloud server, keep n8n private.** The command above already limits n8n to `127.0.0.1`, so it isn't open to the internet. Reach it from your laptop through an SSH tunnel: `ssh -L 5678:localhost:5678 user@SERVER_IP`, then open http://localhost:5678. Don't change it to `-p 5678:5678` on a server unless you put n8n behind HTTPS and a strong password.

**Updating n8n later (Docker):**

```bash
docker pull docker.n8n.io/n8nio/n8n
docker stop n8n && docker rm n8n
```

Then run the `docker run …` command from step 2 again. Your workflows are kept in `n8n_data`.

### 1. Create your Telegram bot

1. In Telegram, message **@BotFather** and send `/newbot`. Follow the prompts and copy the **token** it gives you.
2. Open your new bot and tap **Start** (it can't message you until you do).
3. Message **@userinfobot**. It replies with your **Id**, a number. That's your chat ID.
4. Test it by opening this link in a browser, with your values and no brackets:
   `https://api.telegram.org/botYOUR_TOKEN/sendMessage?chat_id=YOUR_ID&text=test`
   If "test" arrives in Telegram, you're good.

### 2. Import the workflow

1. Download [`job-hunter-instant-alerts.json`](job-hunter-instant-alerts.json) (open it, then click the **Download raw file** button at the top right).
2. In n8n, click **Create workflow** (top right of the home page) or the **+** in the left sidebar. A blank canvas opens.
3. Click **⋯** (top right) → **Import from file…** and pick the file.
   *Or* open the file in a text editor, copy everything, click the empty canvas and paste.

### 3. Add your settings

Double-click the **Check career pages** node and edit the top section:

| Setting | What to put |
|---|---|
| `TELEGRAM_TOKEN` | Your bot token |
| `TELEGRAM_CHAT_ID` | Your chat ID number |
| `ROLES` | The roles you want (already set for Product Manager, Business Analyst, Systems Analyst, Delivery roles and similar) |
| `EXCLUDE` | Titles you never want (engineer, sales, intern and similar) |
| `LOC` | Your locations (default: India, Bengaluru, Hyderabad, Pune, Remote) |
| `COMPANIES` | **Replace the example list with your own target companies**, separated by `\|`. The included list is the author's (mostly GCCs hiring in India) |
| `WORKDAY_URLS` | Optional, see below. Two examples are included, switched off with `//` |

Keep the quotes and semicolons as they are.

> **Tip:** use each company's name as it appears on its careers page (e.g. `Atlassian`, `Datadog`). You can add or remove companies at any time; removed ones are forgotten automatically on the next run.

### 4. Turn it on

Click **Save**, then **Publish** (or switch it to **Active** on older n8n versions).

> Don't test by clicking "Execute" by hand: manual runs don't save memory, so you'd get alerts for old jobs.

---

## What to expect

- **First few hours:** no alerts. It is finding each company's job board (20 companies per run) and recording the jobs already posted, so you aren't flooded with old ones.
- **After that:** one Telegram message per run, listing only **new** matching jobs. Jobs matched through the description are tagged `[matched in description]`.
- **To see progress:** go to **Executions** → latest run → **Check career pages**. The output shows:
  - `telegram_error`: empty when alerts are being delivered; otherwise the reason (wrong token, chat not found…)
  - `on_public_boards`: companies it is watching
  - `not_found_add_manually`: companies it couldn't find. These are re-checked automatically once a week, in case a company moves to a supported job board
  - `jobs_remembered`: jobs already seen (anything older than 60 days is forgotten automatically, so the workflow stays fast)

## Adding Workday companies

Many large companies use Workday, which isn't found automatically.

1. Open the company's careers site and click any job.
2. If the address contains **myworkdayjobs.com**, copy the address up to the first part after `.com/`. Example:
   `https://astrazeneca.wd3.myworkdayjobs.com/Careers`
3. Add it to `WORKDAY_URLS`, in quotes, followed by a comma. Two examples (AstraZeneca, Amgen) are already there; remove the `//` in front of a line to switch it on.

Companies using other systems (SuccessFactors, Taleo, Oracle, iCIMS, their own sites) aren't supported. Follow those on LinkedIn and turn on job alerts there.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| "Task execution timed out after 60 seconds" | Use the latest version of the workflow. Or set `N8N_RUNNERS_TASK_TIMEOUT=300` in your n8n environment. |
| Telegram test says "chat not found" | Wrong chat ID. Use your own ID from @userinfobot, not the bot's number. |
| Telegram says "bot can't initiate conversation" | Open your bot and tap **Start**. |
| Telegram says "Unauthorized" | Wrong token. Copy it again from @BotFather. |
| `new_jobs: 0` for hours | Normal at first. Alerts only come for jobs posted after the first check. |
| No Telegram messages, but `new_jobs` is above 0 | Look at `telegram_error` in the run's output; it tells you exactly what Telegram rejected. |
| A company stays in `not_found_add_manually` | Check its careers page. If it's on Workday, add it to `WORKDAY_URLS`; other systems aren't supported. |
| Want to start completely fresh | Duplicate the workflow (⋯ → Duplicate) and publish the copy; its memory starts empty. |

## Privacy

Your bot token gives full control of your bot. Never post it in screenshots or share the workflow with your token filled in. If it leaks, send `/revoke` to @BotFather.

---

## License

[MIT](LICENSE): free to use, change and share, including commercially. Keep the copyright notice.

*Built with n8n.*
