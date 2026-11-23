# ROEF Social Planner

Content planner for ROEF's Instagram and TikTok (@byroef), built for the build-up to the Kickstarter launch. Sam and Kim log in; nobody else can see anything.

## What it does

- **Calendar** with an idea bank. Drag ideas onto a day, or drop your own photos and videos on a day to start a post.
- **Feed preview** of how the Instagram or TikTok profile will look. Drafts are shown faded.
- **List** with search and a status filter. **Shotlist** of what's still to film, by week, with "Copy for WhatsApp".
- **Playbook** with what works, openings that fit ROEF, a content rhythm up to launch, and a search (SEO) checklist.
- **Claude features:** write a script from an idea, a link or your own footage; optimise for search; ask for changes; **make it sound human** in Kim's voice (from the voice samples in Brand & launch).
- **Post today:** approved posts for today with one-tap copy (captions, hashtags, alt text), a Google Calendar reminder and "Mark as posted". Drafts for the next 3 days can be approved there. Kim posts in the apps herself, so she can pick the sound.
- **Results:** fill in views, likes, comments, shares, saves, followers and watch time per platform 2 days after posting. The Results screen compares pillars and formats, and "Ask Claude what's working" saves learnings that feed every script.
- **Automatic Instagram and Facebook stats:** every day at 06:15 UTC, `pg_cron` calls the `meta-sync` Edge Function. It reads views, reach, likes, comments, shares and saves for @byroef and the ROEF Facebook page through Meta's free Graph API, matches each post to the planner (by post link, or by date and caption) and fills in Results. "Sync now" on the Results screen runs it by hand (at most every 10 minutes). Posts it can't match are listed under the sync card. TikTok numbers are still typed in by hand.
- **Import numbers from a file:** on the Results screen, drop a CSV export (Meta Business Suite → Insights → Content → Export data, or a TikTok CSV). The planner recognises English and Dutch column names, matches each row to a planner post (by post link, then caption, then same day), shows a check screen where you can change or skip matches, then saves the numbers, marks those posts as posted and fills in empty post links. The file is read in the browser and not stored.
- **Comments and @tags:** every saved post has a comment thread (table `comments`, members only, live updates). Type @ and a name to tag someone (`@all` tags everyone). Tagged people see a red count on **For you** and on the post in the calendar; opening the post marks it read. Names come from the `name` column in `allowed_users`, so everyone needs their own login for this to work.
- **Sounds:** each post has a Sound block (sound or beat, what to search in the app, how to use it) with a "Suggest a sound with Claude" button. @byroef is a business account, so suggestions stick to TikTok's Commercial Music Library, Instagram's Sound Collection and no-licence audio (voiceover, real sound). The weekly trend scout also lists business-safe sounds (`settings.trendSounds`), shown in the Playbook and as tap-to-use chips in each post.
- **Google Drive:** "Add from Google Drive" in a post opens Google's file picker (scope `drive.file`, so the planner only sees files you pick). Files stay in Drive; the planner stores a small preview and a link. Needs a Google Cloud OAuth client ID, a browser API key and the project number, entered under Brand & launch (stored in the members-only settings row, not in this repo).
- **Learning from edits:** the planner keeps Claude's version of every field it writes (`aiDraft` on the post). When a post is saved, each changed field is logged in table `edit_log` (members only): Claude's draft and the final text, older text and the new text, or text typed from scratch, plus change requests given to Claude. Every Monday at 07:15 UTC (and on "Update now" in the Playbook) the `learn-edits` Edge Function asks Claude for short style lessons and before/after examples (`settings.editLessons`, `settings.editExamples`). These go into every Claude request. The team can edit or delete lessons in the Playbook. It needs at least 3 logged changes and gets useful after about 10 edited posts.
- **New version banner:** an open planner tab checks every 15 minutes, and when you switch back to it, whether a newer version is live, and asks you to reload.
- **Weekly trend scout:** every Monday 07:45 UTC, `pg_cron` calls the `trends` Edge Function. It uses Claude with web search to research TikTok and Instagram trends, adds 3 trend ideas to the idea bank (marked Trend, expire after 14 days) and updates the trend brief in the Playbook. "Refresh trends now" in the Playbook runs it by hand (at most once an hour). Needs web search enabled for the org in the Claude Console (about $10 per 1,000 searches, 5 searches per run).
- **Free search phrases:** "Find real search phrases" shows what people actually type into Google and YouTube (English and Dutch), plus a field for what Instagram and TikTok suggest. Claude uses both when optimising.
- **Brand & launch:** launch date, content pillars, and the brand brain Claude reads before every script. The brand brain lives in the database, not in the code.

## Files

| File | What it is |
|---|---|
| `index.html` | **The website.** Login, data and storage through Supabase; Claude through a Supabase Edge Function. |

This repo is public. Keep private details (brand rules, plans, prices) in the database's brand brain, never in the code.

## How it's set up

| Piece | Where |
|---|---|
| Database (posts, settings) | Supabase project `roef-planner` (ref `vxrwoibtoobyksdkldod`), tables `posts` and `settings`, JSON per row |
| Who can use it | Table `allowed_users` (login emails). Every table, file and the Claude function checks it |
| Photos and videos | Private storage bucket `media`, 50 MB per file |
| Live sync between Sam and Kim | Supabase Realtime on `posts` and `settings` |
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
6. **Hosting:** GitHub Pages from the `main` branch: https://torremanss-del.github.io/roef-planner-web/

One shared login (by.roef@gmail.com). Change the password with "Change password" at the top of the planner.

Forgotten password: reset it in Authentication → Users.
