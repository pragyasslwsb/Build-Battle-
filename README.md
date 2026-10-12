# InsightFlow

A responsive customer-feedback decision dashboard for the Build Battle 2026 brief. It loads the supplied 7,000-row feedback dataset, derives sentiment and issue priorities from the actual review text, and supports CSV imports, feedback management, analytics, issue tracking, and filter-matched reports.

## Start the application

From the existing `insightflow` project directory:

```sh
npm run dev
```

Open the local URL printed by Vite. Production build and lint checks:

```sh
npm run build
npm run lint
npm test
```

## Deploy

For Vercel, import the existing `insightflow` directory as a Vite project. Set the project root to `insightflow`, the build command to `npm run build`, and the output directory to `dist`. The production build includes `dist/data/feedback.csv`; deploy the complete `dist` output so the sample dataset remains available. No environment variables or client-side API keys are required. The same `dist` folder can be hosted by any static host that serves Vite's `index.html` and public assets.

The supplied source dataset is kept at `public/data/feedback.csv` so the app can load it both in development and from a production build. Additions and status changes are saved in this browser's local storage; they are not sent to a server.

## Feedback CSV format

Import a `.csv` file from the Feedback or Issues view. It must include `review` and `category` headers; other columns are ignored. Quoted values, commas, escaped quotes, and newlines inside reviews are supported. Add feedback individually from the **Add feedback** dialog. Open any review to inspect its sentiment, detected topics, priority score, prioritization reason, and editable status. Search covers review text, category, and detected topics; category, topic, sentiment, status, priority, date, and sorting controls are available in the feedback and issue views. Feedback export respects the active search and filters.

Feedback imports may optionally include a `date`, `review_date`, `created_at`, `created_date`, `date_created`, `submitted_at`, `date_submitted`, or `timestamp` column, and a `severity`, `impact`, `urgency`, or `priority` column. Common space-separated versions of the date headers are also accepted. Supported dates are validated and normalized to `YYYY-MM-DD`; invalid supplied dates reject the import with the offending row number. Additional columns are ignored. New feedback submitted through the app is stamped with its actual local submission date. Missing imported dates remain missing: date-filtered results omit undated records, and trend charts clearly report when no real dated data is available. The provided 7,000-row dataset contains only `review` and `category`, so it does not support a truthful historical trend.

Sentiment is calculated locally using an explainable word-and-negation classifier, not a hosted AI model. Topic detection matches transparent, reviewable theme lexicons. Reviews can match multiple topics; topics, chart totals, percentages, category counts, and recommendations are recalculated from the loaded and edited records. Open issues are prioritized using unresolved negative feedback and urgent fraud or safety language. Recommendations link directly to the matching topic's open issues and show the counts that justify their ranking. Treat sentiment and topic labels as triage signals rather than ground truth.

The Analytics and Reports views share the same search, category, topic, sentiment, status, priority, and date filters. Report summaries, evidence, recommendations, CSV exports, and print-to-PDF output all use the same selected feedback. Recommendation priority scores are bounded from 0 to 100 and combine unresolved negative sentiment (40 points), topic frequency within the selected feedback (20), urgent language (30), and explicitly supplied high severity (10). Individual record scores similarly expose their negative, urgent, severity, and frequency contributions. Evidence excerpts are taken directly from the selected customer reviews.

## Adaption workflow

InsightFlow's Adaption integration follows the vendor's documented **Adaptive Data CSV import** workflow: no undocumented browser API, credentials, or secret is embedded in this frontend. **Adaption** exports a CSV with:

- `prompt` — an instruction to classify sentiment and identify the feedback category.
- `context` — the original customer review.
- `completion` — calculated sentiment, source category, and detected topics as JSON.

In Adaption, create a dataset from a local file, upload `insightflow-adaption-training.csv`, and map those columns to Prompt, Context, and Completion. Review the configured recipes, run estimate, and output in Adaption before using adapted data. InsightFlow does not start or monitor a remote Adaption job.

This export uses the documented data roles and workflow described in [Adaption's Adaptive Data walkthrough](https://docs.adaptionlabs.ai/tutorials/adaptive-data-app-walkthrough/) and [column-mapping guide](https://docs.adaptionlabs.ai/adaptive-data/select-columns). The supplied Adaption reference PDF and current official documentation describe an authenticated Adaption workspace for importing the CSV and running Adaptive Data; they do not provide project-specific credentials. A live remote SDK integration is therefore not enabled: it needs an Adaption account, an API key from Adaption settings, and a secure server-side environment configured with the official `ADAPTION_API_KEY`. Those credentials/server setup were not provided, and a browser bundle is not a safe place for that secret. The CSV preparation and export workflow is fully functional without credentials; an Adaption adaptation run must be started and reviewed in the authenticated Adaption workspace. See [Adaption's SDK setup](https://docs.adaptionlabs.ai/introduction/getting-started) and [dataset upload guide](https://docs.adaptionlabs.ai/adaptive-data/create-a-dataset) for the current official options.
