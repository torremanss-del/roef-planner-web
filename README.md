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
| Instagram and Facebook stats | Edge Function `meta-sync` with the secret `META_TOKEN` (system user key from ROEF's own Meta app, read-only). Raw numbers in table `social_stats` |
| Claude | Edge Function `claude`, calls the Anthropic API with the secret `ANTHROPIC_API_KEY` (optional `ANTHROPIC_MODEL`, default `claude-sonnet-5-5`) |

The Supabase URL and publishable key in `index.html` are meant to be public: access is protected by login and row-level security. The Anthropic key only lives in Supabase secrets.

## One-time setup (Sam)

1. **Add the login emails** to `allowed_users` (Claude can do this in a chat).
2. **Create both logins:** Supabase dashboard → Authentication → Users → Add user → Create new user. Enter email and password, tick "Auto Confirm User".
3. **Turn off public sign-ups:** Authentication → Sign In / Providers → turn off "Allow new users to sign up".
4. **Add the Claude key:** Edge Functions → Secrets → add `ANTHROPIC_API_KEY` with a key from console.anthropic.com.
5. **Meta stats (optional):** in Meta Business Settings, create a system user with read access to the ROEF Facebook page and Instagram, generate a key for the ROEF app (expiry: never; permissions `pages_show_list`, `pages_read_engagement`, `read_insights`, `instagram_basic`, `instagram_manage_insights`, `business_management`) and add it under Edge Functions → Secrets as `META_TOKEN`.
6. **Hosting:** GitHub Pages from the `main` branch: https://torremanss-del.github.io/roef-planner-web/

One shared login (by.roef@gmail.com). Change the password with "Change password" at the top of the planner.

Forgotten password: reset it in Authentication → Users.
