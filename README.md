# Cinema E-Booking System

A full-stack movie ticketing platform with customer booking and a complete administrator
portal — built as a 5-person team project over three Agile sprints.

<!-- TODO: add a screenshot or GIF here. This is the single highest-value addition to
     this README. A shot of the admin portal or the seat-selection screen. -->
![Admin portal](docs/screenshot-admin.png)

## Team & My Role

Team project (5 developers), delivered across three sprints.

I served as **Backend Developer**, with primary ownership of the Django REST Framework
backend — the data model, the admin portal API, and the authentication system —
along with React interfaces on the front end. I also coordinated sprint planning,
version control, and peer code review across the frontend and backend teams.

<!-- TODO: Make this precise before publishing. List what you personally built vs. what
     teammates built. Accuracy here costs nothing and protects you in an interview. -->

## Features

**Customer**
- Registration with email verification, login, logout, password reset
- JWT-based sessions with refresh tokens
- Profile management with multiple addresses and saved payment cards
- Movie browsing by genre, status, and showtime
- Seat selection, ticket purchasing, promo code redemption

**Administrator**
- Dashboard with live counts of movies, promotions, users, and showtimes
- Movie management with full attribute validation (title, description, age rating,
  poster/trailer URLs, status)
- Showtime scheduling across multiple showrooms with **conflict prevention** — no two
  movies can occupy the same showroom at overlapping times
- Promotion creation with validated promo codes, date ranges, and discount percentages
- Targeted promotional email delivery to opted-in subscribers only

## Tech Stack

**Backend:** Python · Django · Django REST Framework · SimpleJWT · MySQL
**Frontend:** React · JavaScript
**Other:** Git · Agile/Scrum

## Architecture Notes

**Event-driven design.** Admin and auth operations dispatch through a custom pub-sub
event dispatcher rather than calling side effects inline. Creating a promotion emits an
event; the email-notification handler subscribes to it. This keeps notification logic
decoupled from the views and makes new listeners cheap to add.

**Showtime conflict detection.** New showings are validated against existing bookings for
the same showroom using Django `Q` objects to test time-range overlap, covering all four
overlap cases (new-starts-inside, new-ends-inside, new-contains-existing,
existing-contains-new). Conflicts return a descriptive JSON error rather than a generic 400.

**Two-layer admin authorization.** `IsAuthenticated` validates the JWT; `IsAdminUser`
checks the `is_staff` flag. Non-admin requests to admin endpoints return 403.

**Payment card encryption.** Card data is encrypted at rest with Fernet (symmetric
authenticated encryption) rather than stored in plaintext.

**Resilient email dispatch.** Each promotional send is individually wrapped, so one
failed recipient doesn't abort the batch; failures are logged and the response reports
the successful send count.

**API conventions.** Consistent JSON envelopes (`message` / `error` / `data`), validation
concentrated in serializers rather than views, and docstrings with request/response
examples on each endpoint.

## Setup

```bash
git clone https://github.com/YOUR-USERNAME/cinema-ebooking-system.git
cd cinema-ebooking-system

python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt

cp .env.example .env             # configure DB credentials and email settings
python manage.py migrate
python manage.py loaddata showrooms.json   # seeds 3 test showrooms
python manage.py createsuperuser
python manage.py runserver
```

Frontend:

```bash
cd frontend
npm install
npm start
```

<!-- TODO: verify these commands actually work from a clean clone. A broken setup
     section is worse than none — it's the one part a reviewer might actually run. -->

## Project Structure

```
├── cinema/
│   ├── models.py           # Movie, Genre, Showroom, MovieShowtime, Promotion,
│   │                       # Profile, Address, PaymentCard, tokens
│   ├── serializers.py      # validation layer
│   ├── views.py            # public movie endpoints
│   ├── views_admin.py      # admin portal endpoints
│   ├── views_auth.py       # registration, login, password reset, profile
│   ├── events.py           # event dispatcher (pub-sub)
│   ├── event_handlers.py   # subscribers (email, logging)
│   └── urls.py
├── frontend/               # React application
└── requirements.txt
```

## Sprints

| Sprint | Delivered |
|---|---|
| 1 | Registration, email verification, login/logout, password reset, movie browsing |
| 2 | Profile editing, addresses, payment cards, seat selection, booking |
| 3 | Admin portal: dashboard, movie management, showtime scheduling with conflict prevention, promotions with targeted email |

## License

MIT
