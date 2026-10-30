
I'm building Spendly — a personal expense tracker web app built with Flask + SQLite + Jinja2.
 
Here is my exact project structure:
 
expense-tracker/
├── .claude/
│   └── specs/
│       ├── 01-database-setup.md   ← complete
│       └── 02-registration.md     ← complete
├── database/
│   ├── __init__.py
│   └── db.py                      ← get_db(), init_db(), seed_db(), create_user() implemented
├── static/
│   ├── css/
│   └── js/
├── templates/
│   ├── base.html
│   ├── landing.html
│   ├── login.html                 ← stub template, no form action or method yet
│   └── register.html              ← complete
├── .gitignore
├── app.py                         ← GET /login and GET /logout exist as stubs only
├── README.md
└── requirements.txt
 
Steps 01 and 02 are complete. Now write a spec for Step 03: Implement login and logout.
 
Save the output as:
  .claude/specs/03-login_logout.md
 
Create the folder if it does not exist.
 
Rules for the spec itself:
- Output only the Markdown spec document
- No code, no prose outside the spec sections
- Use bullet lists throughout
- Use a checkbox list for Definition of Done
- Use the exact section names listed below — no numbers, just headings
 
Include these sections in order:
 
Overview
  — This step converts the /login stub into a functional POST handler that:
      — verifies credentials against the users table
      — stores the authenticated user's id in session["user_id"]
      — also stores the user's name in session["user_name"] so the navbar can show it
      — redirects to the landing page (until a dashboard route exists)
  — Also implements /logout, which clears the session and redirects to /
  — After this step the app can distinguish logged-in users from guests
  — This is a prerequisite for all expense features
 
Depends on
  — Step 01 — Database Setup (users table must exist)
  — Step 02 — Registration (create_user and password hashing must be in place;
    a user must exist to log in against)
 
Routes
  — GET /login — render login form — public
  — POST /login — validate credentials, set session, redirect — public
  — GET /logout — clear session, redirect to / — public (no login required)
 
Database changes
  — No database changes
  — The users table from Step 01 already stores email and password_hash
  — One new DB helper to add to database/db.py:
      get_user_by_email(email)
      — queries the users table by email
      — returns the matching user row or None if not found
      — must live in database/db.py, not inline in the route
 
Templates
  — Modify templates/login.html:
      — add a POST form with email and password fields
      — set form action to url_for('login') with method="post"
      — add a block to display flashed error messages
      — add a link to /register for new users
      — the flash block must render both success and error messages, so the
        "registered successfully" message flashed by Step 02 is visible here
  — Modify templates/base.html:
      — when session["user_id"] is set, the navbar shows the user's name
        (from session["user_name"]) and a Logout link
      — when not set, the navbar shows the Login and Register links
      — Step 04 relies on this navbar logged-in state
 
Files to change
  — app.py — implement login() as GET+POST handler; implement logout()
  — database/db.py — add get_user_by_email() helper
  — templates/login.html — add POST form and flash display
  — templates/base.html — add the logged-in / guest navbar states
 
Files to create
  — None
 
New dependencies
  — None — werkzeug.security.check_password_hash is already available
 
Rules for implementation
  — No SQLAlchemy or ORMs — use raw sqlite3 via get_db()
  — Parameterised queries only — never use f-strings in SQL
  — Verify passwords with werkzeug.security.check_password_hash
  — Session key for the logged-in user must be session["user_id"] (integer)
  — Also set session["user_name"] to the user's name on login
  — Call session.clear() before setting the session keys on login
    (prevents carrying over a previous session)
  — Strip whitespace and lowercase the email before the lookup
    (Step 02 stores emails lowercased)
  — Use flask.session — do not roll a custom session mechanism
  — On failed login show a generic flash error: "Invalid email or password."
    — do not reveal which field was wrong
  — An empty email or password also gets the same generic error
  — On failed login re-render the form with the flashed error — do not redirect
  — After successful login redirect to url_for("landing") until a dashboard exists
  — logout() must call session.clear() then redirect to url_for("landing")
  — All templates must extend base.html
  — Use CSS variables — never hardcode hex values
  — Use url_for() for every internal link — never hardcode paths
 
Definition of done
  — Checkbox list:
    - GET /login renders the login form with email and password fields
    - Submitting with valid credentials (demo@spendly.com / demo123) sets
      session["user_id"] and redirects to /
    - Submitting with a wrong password shows "Invalid email or password." flash
      and stays on the login page
    - Submitting with an unregistered email shows the same generic error flash
    - GET /logout clears the session and redirects to /
    - After logout, session["user_id"] is no longer present
    - The /logout route no longer returns the raw stub string
    - Logging in as "Demo@Spendly.com" (mixed case) with demo123 succeeds
    - After login the navbar shows the user's name and a Logout link; after
      logout it shows Login and Register again
    - Registering a new user then lands on /login showing the success message
 