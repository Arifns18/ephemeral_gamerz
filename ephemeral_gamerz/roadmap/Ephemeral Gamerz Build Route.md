Django build plan · MVP

# Ephemeral Gamerz Build Route

The order to build the site in. Each phase relies on the ones above it, and ends with a check you can try in the browser before moving on. App names refer to the Django apps you create with `startapp`.

Build phase Decision gate After launch

Start: empty repo with README

PHASE 0

## Project setup

config/

- Python virtualenv, `pip install django pillow django-environ`, `requirements.txt`
- `django-admin startproject config .`, add a `.gitignore`
- Keep secrets in `.env` (SECRET_KEY, DEBUG, DATABASE_URL). Use SQLite for now; Postgres can wait until deploy
- Set up `templates/base.html` with a nav bar, a static folder, and `MEDIA_ROOT` for uploaded images

**Done when:** `runserver` shows your base page and the project is committed.

PHASE 1

## User accounts

accounts

**Do this first:** create a custom `User(AbstractUser)` and set `AUTH_USER_MODEL` before your first `migrate`. Changing the user model later is painful.

- Login and logout with Django's built-in `LoginView` / `LogoutView`
- A sign-up view using `UserCreationForm`
- `Profile` fields (avatar, bio) on the custom user, or as a one-to-one model
- An "Edit profile" page with an avatar upload

**Done when:** you can sign up, log in, change your avatar, and see the account in `/admin`.

PHASE 2

## Game catalog

games

- `Game` model: title, platform, genre, release_year, cover_image
- One shared row per game: add a `UniqueConstraint(title, platform)` so users don't create duplicates
- Register it in the admin with search and filters
- A basic game detail page at `/games/<slug>/`

**Done when:** games added in the admin show up on their own pages with a cover image.

PHASE 3

## Personal playlist

library

- `PlaylistEntry`: user → game, added_at, unique per (user, game)
- "Add game" flow: pick an existing game, or create a new one if it isn't there yet
- Edit or remove entries. Only the owner can, enforced with `LoginRequiredMixin` and owner checks
- A "My library" page with a grid of covers

**Done when:** two test users each have their own library, and adding the same game twice is blocked.

PHASE 4

## Reviews & ratings

reviews

- `Review`: user, game, rating (1–10 with `MinValueValidator` / `MaxValueValidator`), body, created / updated
- One review per user per game. Add a "Write review" button on each playlist entry
- Edit or delete your own reviews only, using `UserPassesTestMixin`
- Show the average rating with `annotate(avg=Avg("reviews__rating"))` and list every review on the game page

**Done when:** a game page shows the correct average and all reviews, and user B can't edit user A's review.

PHASE 5

## Discovery & core pages

games · accounts

- Home page: recent reviews and popular games, sorted by review count and average
- Browse and search games that have reviews (`title__icontains` to start), with pagination
- Public profile page at `/u/<username>/` showing that user's playlist and reviews
- Use `select_related` / `prefetch_related` on list pages so they don't run a query per item

**Done when:** a logged-out visitor can find a game, read its reviews, and open a reviewer's profile.

PHASE 6

## Polish & tests

all apps

- Tests for permissions, unique constraints, and average ratings (`TestCase` or pytest-django)
- Styling: a classless CSS framework like Pico.css, or Tailwind if you want full control
- Flash messages ("Review saved"), custom 404 and 403 pages, empty states
- A seed command (`manage.py seed`) that fills the database with demo data

**Done when:** `manage.py test` passes and every page looks right on a phone.

**NO ↺**Go back to the phase that owns the failing item

Every MVP item in the README works?

**YES ↓**Go on to deploy

- Sign up / log in / log out
- Profile with avatar and bio
- Add, edit, and remove playlist games
- Rate and review, then edit or delete
- All reviews and the average on the game page
- Browse and search reviewed games
- Home, game, and profile pages

PHASE 7

## Deploy

config/

- Switch to Postgres (Neon, Render, or Railway) and run migrations there
- `gunicorn` as the web server, `whitenoise` for static files, `DEBUG=False`, `ALLOWED_HOSTS` set
- Store uploaded images in the cloud (Cloudinary or S3 through `django-storages`). Most hosts delete local files on every redeploy
- Run `manage.py check --deploy` and fix what it reports

**Done when:** the site is live at a public URL and avatars are still there after a redeploy.

After launch · any order

### Likes & comments

`ReviewLike` and `Comment` models that link to a review.

### Follow users

A `Follow` model (who follows whom) and a feed of reviews from people you follow.

### Tags & filters

Genre and platform as their own tables; filtering with `django-filter`.

### Game API

Fill in game details from RAWG or IGDB and store their ID on `Game`.

Rule of thumb: commit at the end of every phase, and don't start the next one until its "Done when" check passes.