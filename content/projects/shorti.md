---
title: "Shorti"
date: 2026-10-25
description: "URL shortener with click tracking, built with React, Flask and SQLite, running in Docker behind Caddy on an Oracle Cloud Arm VM."
stack: ["React", "Flask", "SQLite", "Docker", "Caddy", "Oracle Cloud"]
repo: "https://github.com/al-laktah/shorti"
website: "https://shorti.al-laktah.com"
---

A web app that shortens URLs and tracks how often each short link is clicked, live at [shorti.al-laktah.com](https://shorti.al-laktah.com).

I originally built it in 2021 for shorti.xyz. In 2026 I brought it back online: I updated the dependencies to run on current Python and Node, made the domain configurable and packaged the whole app with Docker.

## How it works

- **Frontend:** a React app for shortening links and looking up how often a short link has been clicked.
- **API:** a small Flask API, served by gunicorn, with three endpoints: shorten a URL, resolve a short code (counting the click), and return a link's click count.
- **Short codes:** each URL is hashed with a random salt (SHA-256). The first five characters of the hash, randomly capitalised, become the code; on a collision it moves further along the hash.
- **Redirects:** opening a short URL loads the React app, which asks the API for the original URL and sends the browser on to it.
- **Storage:** SQLite, kept in a Docker volume.

## Deployment

Docker Compose runs two containers: the Flask API, and a Caddy server that serves the React build and forwards `/api/` requests to it. In front of them, a shared Caddy proxy handles HTTPS, so other services can run on the same server.

It all runs on the Oracle Cloud Always Free Arm VM from my [al-laktah-cloud](/projects/al-laktah-cloud/) project, which sets up the server with Terraform, backs up Shorti's database every night and monitors the site from a second VM.

## Next steps

- Move the database off SQLite
- Make the contact and report forms send their messages
- A proper page for short URLs that don't exist
- Dark mode, and privacy and terms pages
- Improve the shortening algorithm
