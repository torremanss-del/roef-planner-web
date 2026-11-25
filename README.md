# ROEF Social Planner

Content planner and creator for ROEF's Instagram, TikTok and Facebook (@byroef), built for the build-up to the Kickstarter launch. Only the people in `allowed_users` can log in.

## What it does

Five screens, built to work like Asana for social media.

- **Home:** your top 3 tasks, your open tasks (or everyone's), posts ready to post today with one-tap copy, posts waiting for Kim's check, what's coming up, where you were tagged, posts that still need TikTok numbers, and the newest ideas.
- **Ideas:** "Get 6 new ideas" asks Claude for ideas built on your results, this week's trends and sounds, and the hashtags and search phrases that worked, each with the reason. Ideas from the Monday trend scout and from the team sit next to them. "Plan it" opens the idea as a post; "Not for us" removes it. The trend brief, business-safe sounds and the hashtag/search phrase scores are on this screen too.
- **Plan:** Calendar (drag ideas onto days), Board (every post moves Idea → Script → Film → Edit → Review → Ready → Posted, by drag or a button), Tasks (everyone's open tasks by person, quick add), Creators (dog accounts, shops and partners to contact, with status, owner and next step) and Media (every photo and video attached to a post, footage not posted yet first).
- **Results:** Kickstarter tracker (tracked links for the bio and story stickers, click counts by place and post, weekly sign-up count), CSV import, the Instagram + Facebook sync, results by pillar and format, Claude's learnings, and the hashtag/search phrase scores.
- **Playbook:** the brand book (copy rules, do/don't, colours, type), the writing style sources, formats you repeat (templates), Claude's lessons from your edits, and tips.
- **Posts:** each post has an owner, a stage, a date, platforms (Instagram, TikTok, Facebook), Claude writing (from an idea, a link, a screenshot or your own footage, or a template), photos and videos (upload or Google Drive), script and sound, caption and search, tasks ("Add the standard tasks" makes script, film, edit, check, post and results tasks with due dates counted back from the post date), results and a tracked link, "Make versions" for other formats, and comments with @tags.
- **Tasks** (table `tasks`, members only, live): per post or on their own, with a person, a due date and notes.
- **Email alerts:** an email when someone tags you or gives you a task, and a Monday overview (your tasks this week, what's planned, last week's best post, clicks and new ideas). Everyone can turn their own emails off in Settings. Needs the free Google mailer, see setup.
- **Writing in ROEF's voice:** every Claude request includes the brand book (`settings.brandBook`), the brand brain (facts and the never-mention list, which wins when the two disagree), Kim's voice samples, the captions you actually posted (from the sync), and lessons from the team's edits. English only.
- **Automatic Instagram and Facebook stats:** every day at 06:15 UTC, `pg_cron` calls the `meta-sync` Edge Function. It reads views, reach, likes, comments, shares and saves for @byroef and the ROEF Facebook page through Meta's free Graph API, matches each post to the planner (by post link, or by date and caption) and fills in Results. TikTok numbers are typed in or imported from a CSV.
- **Weekly trend scout:** every Monday 07:45 UTC the `trends` Edge Function uses Claude with web search to research trends and business-safe sounds and adds 3 trend ideas (marked Trend, expire after 14 days).
- **Learning from edits:** changed fields are logged in `edit_log`; every Monday 07:15 UTC the `learn-edits` function turns them into style lessons used in every Claude request.
- **Tracked links:** `https://vxrwoibtoobyksdkldod.supabase.co/functions/v1/go?p=<post or bio>&c=<place>` counts a click in table `clicks` and forwards to the Kickstarter pre-launch page (Settings) or ridewithroef.com. Kickstarter links get a `ref` tag so Kickstarter's own referrer report shows them too.
- **Deep links:** `#post=<id>` opens a post and `#task=<id>` a task (used in the emails). Old links such as `#calendar` or `#today` still work.
- **New version banner:** an open tab checks every 15 minutes whether a newer version is live and asks you to reload.

## Files

| File | What it is |
|---|---|
| `index.html` | **The website.** Login, data and storage through Supabase; Claude through a Supabase Edge Function. |

This repo is public. Keep private details (brand rules, plans, prices) in the database's brand brain, never in the code.

## How it's set up

| Piece | Where |
|---|---|
| Database | Supabase project `roef-planner` (ref `vxrwoibtoobyksdkldod`): `posts` and `settings` (JSON per row), `tasks`, `creators`, `comments`, `edit_log`, `social_stats`, `clicks` |
| Who can use it | Table `allowed_users` (login emails). Every table, file and the Claude function checks it |
| Photos and videos | Private storage bucket `media`, 50 MB per file |
| Live sync | Supabase Realtime on `posts`, `settings`, `comments`, `tasks` and `creators` |
| Email alerts | Database triggers on `comments` and `tasks` call the `notify` Edge Function; cron `roef-monday-digest` (Monday 08:30 UTC) sends the overview. Mail goes out through a Google Apps Script web app (secrets `MAIL_HOOK_URL`, `MAIL_HOOK_SECRET`) |
| Tracked links | Edge Function `go` (public), clicks in table `clicks` |
| Search suggestions | Edge Function `suggest`: Google and YouTube autocomplete (free, unofficial, so it can change or slow down) |
| Instagram and Facebook stats | Edge Function `meta-sync` with the secret `META_TOKEN` (a ROEF Page key from ROEF's own Meta app with read-only permissions; a system user key also works). Raw numbers in table `social_stats` |
| Claude | Edge Function `claude`, calls the Anthropic API with the secret `ANTHROPIC_API_KEY` (optional `ANTHROPIC_MODEL`, default `claude-sonnet-5-5`) |

The Supabase URL and publishable key in `index.html` are meant to be public: access is protected by login and row-level security. The Anthropic key only lives in Supabase secrets.

## One-time setup (Sam)

1. **Add the login emails** to `allowed_users` (Claude can do this in a chat).
2. **Create both logins:** Supabase dashboard → Authentication → Users → Add user → Create new user. Enter email and password, tick "Auto Confirm User".
3. **Turn off public sign-ups:** Authentication → Sign In / Providers → turn off "Allow new users to sign up".
4. **Add the Claude key:** Edge Functions → Secrets → add `ANTHROPIC_API_KEY` with a key from console.anthropic.com.
5. **Meta stats (optional):** the ROEF Facebook page (linked to @byroef) is managed by the Facebook profile Kim Jantzen. The key is a Page key that doesn't expire:
   1. Meta for Developers (logged in as Kim Jantzen) → app **ROEF Planner Stats** → Tools → **Graph API Explorer**.
   2. Pick the app, choose "Get User Access Token", tick `pages_show_list`, `pages_read_engagement`, `pages_read_user_content` (needed to list Facebook posts), `read_insights`, `instagram_basic`, `instagram_manage_insights`, `business_management`, click Generate Access Token and allow the ROEF page, the ROEF business and @byroef.
   3. Click the blue (i) next to the key → **Open in Access Token Tool** → **Extend Access Token** (this makes a 60-day key). Copy that key back into the Explorer.
   4. Keep "User or Page" on User Token and run `me/accounts?fields=name,access_token` in the Explorer. The `access_token` next to "ROEF" is the Page key. Check it with the (i) button: Expiration must say "Never".
   5. Supabase → Edge Functions → Secrets → add it as `META_TOKEN`. Then press "Sync now" on Results.
   The Page key stops working if Kim Jantzen's password changes, the profile is removed as page admin, or the app is deleted. The planner then shows "The Meta key has expired or was revoked": repeat steps 2 to 5.
   Why not a system user: the ROEF business portfolio was made with the Instagram login, and Meta blocks Instagram-only users from Business Settings once the account is linked to a Facebook page, so no one can add a system user there right now. A system user key still works if that's sorted out later.
6. **Email alerts (free, about 5 minutes):** logged in to Google as by.roef@gmail.com:
   1. Go to script.google.com → New project, name it "ROEF mailer", replace the code with:
      ```js
      function doPost(e) {
        var d = JSON.parse((e.postData && e.postData.contents) || '{}');
        var secret = PropertiesService.getScriptProperties().getProperty('SECRET');
        if (!secret || d.secret !== secret) return ContentService.createTextOutput('denied');
        if (!/^[^@\s]+@[^@\s]+\.[^@\s]+$/.test(d.to || '')) return ContentService.createTextOutput('bad address');
        MailApp.sendEmail({ to: d.to, subject: String(d.subject || 'ROEF planner').slice(0, 200), htmlBody: String(d.html || ''), name: 'ROEF planner' });
        return ContentService.createTextOutput('ok');
      }
      ```
   2. Project Settings (gear) → Script properties → add `SECRET` with a long random password you make up.
   3. Deploy → New deployment → type **Web app** → Execute as **Me** → Who has access **Anyone** → Deploy → allow access. Copy the Web app URL.
   4. Supabase → Edge Functions → Secrets: `MAIL_HOOK_URL` = that URL, `MAIL_HOOK_SECRET` = the same password as step 2.
   5. In the planner: Settings → "Send me a test email".
   Gmail lets a free account send about 100 emails a day, plenty for three people. If the script is ever changed, deploy a new version and keep the same URL (Deploy → Manage deployments → Edit).
7. **Hosting:** GitHub Pages from the `main` branch: https://torremanss-del.github.io/roef-planner-web/

Everyone has their own login (Sam, Kim, Noémie, plus the shared by.roef@gmail.com). Change your password with "Change password" at the top of the planner.

Forgotten password: reset it in Authentication → Users.
