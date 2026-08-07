<<<<<<< HEAD
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
=======
# Candidate Assignment Submission Guide

Follow these steps to submit your assignment to this repository.

## Step 1: Fork the Repository

Click the **Fork** button in the top-right corner of this repository. This creates your own copy of the project in your GitHub account.

## Step 2: Clone Your Fork

Copy the URL of your fork and run:

```bash
git clone https://github.com/your-username/repository-name.git
```

Go to the project directory:

```bash
cd repository-name
```

## Step 3: Create a New Branch

Create a branch for your changes.

```bash
git checkout -b your-branch-name
```

## Step 4: Work on the Assignment

Complete the assigned task in your local repository and save your changes.

or

If you have already completed the project using another repository then follow this steps:

1. Copy all of your completed project files into your forked repository.
3. Make sure all required project files are included.
4. Verify that the project builds and runs correctly before committing your changes.

## Step 5: Commit Your Changes

Stage and commit your work.

```bash
git add .
git commit -m "Describe your changes"
```

## Step 6: Push Your Branch

Push your branch to your GitHub fork.

```bash
git push origin your-branch-name
```

## Step 7: Create a Pull Request

1. Open your fork repository on GitHub.
2. Click **Compare & pull request**.
3. Add a clear title and description of your changes.
4. In the PR description, also include:
   - Your email ID
   - Your resume as an attachment
   - A live link or demo video of the assignment
     - **Svelte assignment:** A live website link is **mandatory**.
     - **Flutter assignment:** Using Flutter Web, you can deploy the project and include the live website link.
5. Click **Create pull request**.

Your submission is now ready for review.

---

## Instructions

- Write clear and meaningful commit messages.
- Respond to review comments and update your Pull Request if requested.
- Do not push auto-generated files to the repository. Add them to `.gitignore` as per the task requirements. If auto-generated files have already been pushed, remove them from the repository first, then commit the updated `.gitignore` in your next commit.
>>>>>>> 726b7d6f35e39905c6fe94ae760b419edb3df079
