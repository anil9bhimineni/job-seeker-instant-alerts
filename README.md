# job-seeker-instant-alerts
This will help you setup the code in your N8N for people searching for different jobs , across different GCC so that you would get alerts almost instantly when the job is posted.
Job Hunter – Instant Career-Page Alerts (n8n + Telegram)

Get a Telegram ping within 30 minutes of a matching job being posted on a company's own careers page — often hours or days before it shows up on LinkedIn or Naukri.

Free. Runs on your own n8n. No scraping logins, no paid APIs.

What it does
Every 30 minutes it checks the careers pages of the companies on your list.
It finds each company's job board automatically (Greenhouse, Lever, Ashby). You can add Workday sites by URL.
It keeps only jobs that match your roles (in the title or the job description) and your locations.
It sends new matches to your phone via your own Telegram bot. Nothing goes to your inbox.
What you need
n8n (self-hosted or n8n Cloud). Tested on n8n 2.41.
A Telegram account.
About 15 minutes.
Setup
1. Create your Telegram bot
In Telegram, message @BotFather and send /newbot. Follow the prompts and copy the token it gives you.
Open your new bot and tap Start (it can't message you until you do).
Message @userinfobot. It replies with your Id, a number. That's your chat ID.
Test it by opening this link in a browser, with your values and no brackets: https://api.telegram.org/botYOUR_TOKEN/sendMessage?chat_id=YOUR_ID&text=test If "test" arrives in Telegram, you're good.
2. Import the workflow
Download job-hunter-instant-alerts.json.
In n8n, create a new blank workflow.
Click ⋯ (top right) → Import from file… and pick the file. Or open the file in a text editor, copy everything, click the empty canvas and paste.
3. Add your settings

Double-click the Check career pages node and edit the top section:

Setting	What to put
TELEGRAM_TOKEN	Your bot token
TELEGRAM_CHAT_ID	Your chat ID number
ROLES	The roles you want (already set for Product Manager, Business Analyst, Systems Analyst, Delivery roles and similar)
EXCLUDE	Titles you never want (engineer, sales, intern and similar)
LOC	Your locations (default: India, Bengaluru, Hyderabad, Pune, Remote)
COMPANIES	Your target companies, separated by |
WORKDAY_URLS	Optional, see below

Keep the quotes and semicolons as they are.

4. Turn it on

Click Save, then Publish (or switch it to Active on older n8n versions).

Don't test by clicking "Execute" by hand: manual runs don't save memory, so you'd get alerts for old jobs.

What to expect
First few hours: no alerts. It is finding each company's job board (20 companies per run) and recording the jobs already posted, so you aren't flooded with old ones.
After that: one Telegram message per run, listing only new matching jobs. Jobs matched through the description are tagged [matched in description].
To see progress: go to Executions → latest run → Check career pages. The output shows:
on_public_boards: companies it is watching
not_found_add_manually: companies it couldn't find
Adding Workday companies

Many large companies use Workday, which isn't found automatically.

Open the company's careers site and click any job.
If the address contains myworkdayjobs.com, copy the address up to the first part after .com/. Example: https://astrazeneca.wd3.myworkdayjobs.com/Careers
Add it to WORKDAY_URLS, in quotes, followed by a comma.

Companies using other systems (SuccessFactors, Taleo, Oracle, iCIMS, their own sites) aren't supported. Follow those on LinkedIn and turn on job alerts there.

Troubleshooting
Problem	Fix
"Task execution timed out after 60 seconds"	Use the latest version of the workflow. Or set N8N_RUNNERS_TASK_TIMEOUT=300 in your n8n environment.
Telegram test says "chat not found"	Wrong chat ID. Use your own ID from @userinfobot, not the bot's number.
Telegram says "bot can't initiate conversation"	Open your bot and tap Start.
Telegram says "Unauthorized"	Wrong token. Copy it again from @BotFather.
new_jobs: 0 for hours	Normal at first. Alerts only come for jobs posted after the first check.
Privacy

Your bot token gives full control of your bot. Never post it in screenshots or share the workflow with your token filled in. If it leaks, send /revoke to @BotFather.
