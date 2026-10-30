# Deploy PayOps on Render (fast path)

This repository contains three services: the Angular frontend, the Spring Boot API, and a private fake-bank service. `render.yaml` defines all three. The existing Supabase PostgreSQL database is reused; this deployment does not create or reset a database.

## 1. Prepare the repository

1. Extract this ZIP.
2. Confirm `.env` is not in the repository. It is intentionally excluded from this deployment bundle.
3. Push the project to a **private GitHub repository**. Keep `.env` out of Git.
4. In Render, choose **New + → Blueprint**, connect the repository, and select `render.yaml`.

## 2. Configure the secrets

During Blueprint setup, Render asks for the three `sync: false` database values twice (once for `payops-api`, once for `fake-bank`). Enter the same values from your existing local `.env`:

- `PAYOPS_DB_URL` — the complete JDBC URL beginning `jdbc:postgresql://...`
- `PAYOPS_DB_USERNAME`
- `PAYOPS_DB_PASSWORD`

Render generates `PAYOPS_JWT_SECRET` automatically. Do not reuse a weak or committed JWT secret.

The database URL should use your Supabase **session pooler** endpoint/port if the database is IPv4-only. Ensure the Supabase database allows connections from Render; for a quick first deployment, review Supabase network restrictions rather than opening access more broadly than necessary.

## 3. Database caution

Do **not** run `sql/fresh-install.sql` against a database that already contains PayOps tables or data. It is for a brand-new, empty database only. If this is a new Supabase project, inspect the SQL first and run it once in the Supabase SQL Editor. If your schema already exists, do not rerun it; verify that the roles, admin user, and tables expected by this code are present.

## 4. Deploy and verify

Render builds and deploys the API, private fake-bank service, and frontend. Open the `payops-frontend` URL when its deploy is live. The UI routes `/api` calls through Nginx to the API, so you do not need to hardcode a public API URL in Angular.

Useful checks:

- Frontend: `https://<your-frontend>.onrender.com/`
- API OpenAPI: `https://<your-frontend>.onrender.com/swagger-ui/index.html`
- API health-ish check: `https://<your-api>.onrender.com/v3/api-docs`

If the API cannot connect to the bank, confirm `FAKEBANK_URL=http://fake-bank:8081` and that both services are in the same Render region. If the API cannot connect to PostgreSQL, recheck the JDBC URL, username, password, Supabase pooler port, and Supabase network restrictions.

## 5. Before sharing the URL

- Change the seeded demo passwords immediately; do not expose default demo credentials publicly.
- Keep fake-bank private. It is a simulator, not a real payment processor.
- Use only fictional test cards and test data. This build is not certified for real-money processing or production payment handling.
- Rotate the database password and JWT secret if the uploaded `.env` was ever committed, shared, or exposed. Never upload `.env` to GitHub.

## Notes

The Angular Docker build uses Node 24, which satisfies the project's declared Node engine. The Java services use Java 21. Render's Starter plans are selected in `render.yaml`; review the current cost shown by Render before confirming resource creation.
