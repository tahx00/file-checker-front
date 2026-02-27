# File Integrity Checker — Frontend

A Vue.js web application for detecting file tampering using **SHA-256 cryptographic hashing**. Built as a cybersecurity tool to verify file integrity and track modification history.

## Live Demo

🔗 https://tahxfc.netlify.app/

## Features

- Upload any file and instantly compute its SHA-256 hash
- Compare two files to detect any differences
- Compare a file against a previously stored hash
- View history of last 20 hashing operations
- Clean, responsive UI

## Screenshots

> Add screenshots here

## Tech Stack

- **Framework:** Vue.js
- **HTTP Client:** Axios
- **Styling:** CSS3
- **Deployment:** Netlify

## Installation

```bash
git clone https://github.com/tahx00/file-checker-front
cd file-checker-front
npm install
npm run dev
```

## Backend

The REST API backend for this project:
- **Repo:** https://github.com/tahx00/file-checker-backend

## How It Works

1. User uploads a file
2. Backend computes SHA-256 hash using Node.js `crypto` module
3. Hash is stored in history for future comparison
4. Any file modification — even a single byte — produces a different hash
5. Tampering is immediately detected

## Security Concept

SHA-256 is a one-way cryptographic hash function widely used in cybersecurity for file integrity verification, digital signatures, and blockchain technology.
