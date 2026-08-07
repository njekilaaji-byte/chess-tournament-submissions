# Chess Tournament Management System

A Svelte + JavaScript implementation of the Bytelogik Chess Tournament Management System assignment.

## Features

- Player CRUD: create, read, update and delete players.
- Tournament CRUD: create, read, update and delete tournaments.
- Add/remove players from a tournament.
- Select a tournament from the management and match screens.
- Randomly pair tournament participants.
- Randomly select a winner for every match.
- Record match results.
- Display final 1st, 2nd and 3rd rankings.
- Browser persistence using `localStorage`.
- Responsive interface.

## Tech Stack

- Svelte 5
- JavaScript
- Vite

## Run locally

Requirements: Node.js 18+.

```bash
npm install
npm run dev
```

Open the local URL printed by Vite.

## Production build

```bash
npm run build
npm run preview
```

## Notes

The Svelte assignment does not require SQLite. This implementation uses browser `localStorage` for persistence so the application can run without a backend.

## Submission checklist

- Keep at least five meaningful commits.
- Include the deployed website URL in the pull request.
- Attach the resume and include the email address as requested by the submission guide.
