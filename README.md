# ROEF Social Planner

Content planner for ROEF's Instagram and TikTok (@byroef), built for the build-up to the Kickstarter launch. Sam and Kim log in; nobody else can see anything.

## What it does

- **Calendar** with an idea bank. Drag ideas onto a day, or drop your own photos and videos on a day to start a post.
- **Feed preview** of how the Instagram or TikTok profile will look. Drafts are shown faded.
- **List** with search and a status filter. **Shotlist** of what's still to film, by week, with "Copy for WhatsApp".
- **Playbook** with what works, openings that fit ROEF, a content rhythm up to launch, and a search (SEO) checklist.
- **Claude features:** write a script from an idea, a link or your own footage; optimise for search; ask for changes.
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
| Claude | Edge Function `claude`, calls the Anthropic API with the secret `ANTHROPIC_API_KEY` (optional `ANTHROPIC_MODEL`, default `claude-sonnet-5-5`) |

The Supabase URL and publishable key in `index.html` are meant to be public: access is protected by login and row-level security. The Anthropic key only lives in Supabase secrets.

## One-time setup (Sam)

1. **Add the login emails** to `allowed_users` (Claude can do this in a chat).
2. **Create both logins:** Supabase dashboard → Authentication → Users → Add user → Create new user. Enter email and password, tick "Auto Confirm User".
3. **Turn off public sign-ups:** Authentication → Sign In / Providers → turn off "Allow new users to sign up".
4. **Add the Claude key:** Edge Functions → Secrets → add `ANTHROPIC_API_KEY` with a key from console.anthropic.com.
5. **Hosting:** GitHub Pages from the `main` branch: https://torremanss-del.github.io/roef-planner-web/

One shared login (by.roef@gmail.com). Change the password with "Change password" at the top of the planner.

Forgotten password: reset it in Authentication → Users.
