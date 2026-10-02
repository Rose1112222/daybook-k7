# Maintaining Tally

Tally is a personal calorie log that runs as an installed web app on a phone. This guide is for whoever changes it next, human or AI. It assumes you have never seen the app before.

## What is here

- `index.html` is the whole app: markup, styles, script, and the built-in USDA food list, in one file. There are no libraries and no build step. Edit the file and publish it.
- `sw.js` is a small service worker that keeps a copy of the app on the device so it opens offline.
- `manifest.webmanifest` and the three icon files make it installable.
- `DATA-FORMAT.md` describes everything the app stores. Read it before changing anything that touches saved data.

## Rules that must not be broken

1. Never lose or damage stored data. The owner has years of logs in this format. Do not rename a stored field or change what it means. Add new fields instead, and make the code treat a missing field as its old default.
2. If a change to the stored format is truly needed, write a one-time migration that runs when a record is loaded, and test it against a real backup file first. `loadMonth()` contains an example (the `remap5` meal migration).
3. No advertising, analytics, tracking, suggestions, pop-ups, or outside code libraries.
4. The app may contact only `api.nal.usda.gov` (barcode lookups). The Content-Security-Policy tag at the top of `index.html` enforces this. Do not widen it without a clear need.
5. Any text that comes from outside the app (USDA results, a restored backup, anything typed) must pass through `esc()` before it goes into HTML.
6. The USDA key is stored only in `localStorage` under `tally-usda-key`. Keep it out of the stored documents and out of backups.

## How the code is organized

The script at the bottom of `index.html` starts with a comment that maps its sections. In short:

- `S` is all in-memory state. `Store` saves and loads named JSON documents and has three back ends behind one interface: IndexedDB (the installed app), a hosted document store (when the page runs inside the Claude viewer, where it was prototyped), and localStorage (fallback).
- Each tab has a `view...()` function that returns an HTML string. `render()` redraws the current tab.
- Full-screen panels are "sheets". `openSheet({type: ...})` shows one and `renderSheet()` draws it.
- Every tap is an element with a `data-act` attribute. The handlers are the properties of the object `A`. Text inputs use `data-in` and are handled in the `input` and `change` listeners at the bottom.
- `today()` is not the calendar date. A log day runs until the rollover hour (setting `roll`, default 6 am), or ends early when the day is marked finished (setting `fin`).
- `STANDALONE` is true when the page runs at its own web address. Barcode scanning, the USDA key, and file backups are only shown then.

## Making a change

1. Edit `index.html`.
2. Test it: serve the folder locally (`python3 -m http.server` in this folder, then open `http://localhost:8000`). The app behaves as the installed version there. Check the browser console for errors, including Content-Security-Policy errors.
3. Restore a real backup into the test copy (Settings, Restore from backup) and check that old data still displays correctly.
4. Publish by uploading the changed file to the hosting repository. The phone picks up the new version the next time the app is opened with a connection, and shows it on the following launch.
5. Only change the `CACHE` name in `sw.js` if the list of files changes.

## Things that depend on other parties

Each of these can fail without affecting logging, which needs nothing outside the device.

- USDA barcode lookup: `usdaLookup()` calls the FoodData Central search API. If the API changes or is retired, the app falls back to manual entry. The mapping from the API's nutrient ids is in that one function.
- Camera scanning uses the browser's `BarcodeDetector`. Where it is missing, the scan screen offers typing the number.
- The built-in food list is the USDA SR Legacy release (April 2018, public domain), embedded as JSON in the `<script id="usda">` tag. Values are per 100 g; -1 means the USDA did not measure that nutrient.

## Backups

Settings has buttons that write every stored document to one JSON file, and a button that restores from such a file. The restore replaces documents with the same name. The backup format is in `DATA-FORMAT.md`.
