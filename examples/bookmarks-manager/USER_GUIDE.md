# User Guide — Bookmarks Manager

## Welcome

Bookmarks Manager is a personal web app for saving, organizing, and finding the links that matter to you. Instead of relying on your browser's built-in bookmarks — which can get messy and don't travel well between browsers — this app gives you a clean, searchable collection with tags, descriptions, and easy import from any browser. It's designed for one person: you.

---

## Getting Started

### Opening the App

1. Make sure the app is running on your computer (ask whoever set it up, or follow the installation README).
2. Open your web browser and go to **http://localhost:5173** (or whatever address was configured for you).
3. You'll see the main screen: a search bar at the top, a tag panel on the left, and the bookmark list in the center.

If you've never added any bookmarks yet, you'll see a message: *"No bookmarks yet. Add your first bookmark or import from browser."*

You're ready to start saving bookmarks!

---

## Daily Usage

### Saving a New Bookmark

This is the most common thing you'll do.

#### Add a bookmark manually

1. Click the **"Add Bookmark"** button at the top right of the screen.
2. A form opens. Type or paste the **web address** (for example, `https://www.example.com`) into the first field — this is the only required field.
3. Type a **title** so you remember what the page is about (for example, *"Best CSS tricks collection"*).
4. Optionally type a **description** with more detail (for example, *"Great reference for flexbox and grid examples"*).
5. Optionally add **tags** to group this bookmark with similar ones. Type a word and press **Enter** or **comma** to add it. You can add as many tags as you like. Examples: `css`, `reference`, `frontend`.
6. Click **"Save"**.
7. The form closes and your bookmark appears in the list right away.

`[Screenshot: Add bookmark form with fields filled in]`

> **Tip**: Tags are the best way to organize your bookmarks. Think of them like folders, except a bookmark can be in multiple folders at once.

---

### Finding a Bookmark

#### Search by keyword

1. Click inside the **search bar** at the top of the page.
2. Start typing any word — the title, address, or description of the bookmark you're looking for.
3. The list updates automatically as you type (after a brief half-second pause).
4. To clear the search, click the **×** button inside the search bar, or delete what you typed.

`[Screenshot: Search bar with results filtered]`

#### Filter by tag

1. Look at the **tag panel** on the left side of the screen. It shows all your tags with a number next to each showing how many bookmarks use it.
2. Click a tag name — for example, **"css"** — to show only bookmarks with that tag.
3. Click another tag to narrow down further. The list will show only bookmarks that have **all** the tags you've clicked.
4. Active (selected) tags are highlighted.
5. Click a highlighted tag again to deselect it, or click **"Clear filters"** at the top of the tag panel to start over.

`[Screenshot: Tag panel with one tag selected, filtered list]`

#### Search and filter at the same time

You can use both together. Type a keyword in the search bar and click a tag in the tag panel. The list shows only bookmarks that match **both** conditions. This is great when you have many bookmarks in a tag category and need to narrow it down.

---

### Editing a Bookmark

1. Find the bookmark you want to update.
2. Click the **pencil icon** on the bookmark card.
3. The edit form opens, already filled with the current information.
4. Change whatever you need — title, description, tags, or even the web address.
5. To remove a tag, click the **×** on that tag chip. To add a new tag, type it and press Enter.
6. Click **"Save"**.

`[Screenshot: Edit form pre-filled with bookmark data]`

---

### Deleting a Bookmark

1. Find the bookmark you want to remove.
2. Click the **trash icon** on the bookmark card.
3. A confirmation prompt appears: *"Are you sure you want to delete this bookmark?"*
4. Click **"OK"** (or **"Confirm"**) to delete it permanently, or **"Cancel"** to keep it.

> **Warning**: Deletion is permanent. There's no undo or recycle bin. If you're not sure, consider just removing the tags so it's easier to find later.

---

### Importing Bookmarks from Your Browser

If you have bookmarks saved in Chrome, Firefox, or Edge, you can bring them all in at once.

#### Step 1: Export from your browser

**Chrome or Edge**:
1. Click the three-dot menu (⋮) in the top right.
2. Go to **Bookmarks → Bookmark manager**.
3. Click the three-dot menu inside the bookmark manager.
4. Click **"Export bookmarks"**.
5. Save the file — it will be called something like `bookmarks_mm_dd_yy.html`.

**Firefox**:
1. Click the library icon (📚) in the toolbar.
2. Go to **Bookmarks → Manage Bookmarks**.
3. In the menu bar, click **Import and Backup → Export Bookmarks to HTML**.
4. Save the file.

#### Step 2: Import into the app

1. Click the **"Import"** button at the top of the app.
2. An import window appears with a file drop zone.
3. Either **drag and drop** your exported file onto the drop zone, or click the zone and browse to select the file.
4. Click **"Import"**.
5. A spinner shows while the app processes your file (this usually takes just a second or two).
6. When done, you'll see a summary:
   - **"X bookmarks added"** — new bookmarks brought in
   - **"X skipped"** — bookmarks you already had saved (no duplicates!)
   - **"X failed"** — bookmarks with problems (details shown below)
7. Click **"Done"** to close the window.

`[Screenshot: Import modal showing success summary]`

> **What about my browser folders?** Each folder from your browser becomes a tag. For example, if you had a folder called *"Dev Tools"* containing a bookmark, that bookmark will get the tag `dev tools`. Nested folders become multiple tags.

---

### Visiting a Saved Bookmark

Click the **title** of any bookmark card. The web page opens in a new tab so you don't lose your place in the app.

---

### Switching Between Grid and List View

At the top right, you'll see two small icons:
- **Grid icon** — shows bookmarks in multiple columns (good for lots of bookmarks)
- **List icon** — shows bookmarks in a single column (good for reading descriptions)

Click either icon to switch. The app remembers your preference.

`[Screenshot: Grid view vs list view comparison]`

---

### Navigating Multiple Pages

If you have many bookmarks, they're split across pages (20 per page by default).

- Use the **page numbers** at the bottom to jump to a specific page.
- Use **"← Previous"** and **"Next →"** to move one page at a time.
- The current page is highlighted.

---

## Understanding the App

### Tags — and How They Work

Tags are labels you assign to bookmarks. A bookmark can have any number of tags, or none at all.

- Tags are **case-insensitive**: "CSS", "css", and "Css" all become "css".
- Tags are **automatically trimmed**: spaces at the start or end are removed.
- Tags are **shared**: if two bookmarks both use the tag "frontend", they both appear under that tag in the sidebar.
- Tags are **auto-created**: you don't manage a tag list — just type a tag on any bookmark and it appears in the sidebar.
- Tags are **auto-removed from the sidebar**: if you delete the last bookmark using a particular tag, that tag disappears from the sidebar on its own.

---

### What the Tag Count Means

Next to each tag name in the sidebar, you'll see a number. That number is how many bookmarks currently use that tag. It updates automatically when you add, edit, or delete bookmarks.

---

## Common Scenarios

### Scenario: You found a useful article and want to save it for later

1. Copy the web address from your browser's address bar.
2. Click **"Add Bookmark"**.
3. Paste the address into the URL field.
4. Add a meaningful title (the page title is a good start).
5. Add one or two tags so you can find it later (for example, `reading-list` or `tutorial`).
6. Click **"Save"**.

Done — it's in your collection now.

---

### Scenario: You're looking for a recipe you saved a while ago but can't remember the name

1. Type **"recipe"** in the search bar — or a key ingredient you remember.
2. If you tagged it, click the relevant tag in the sidebar (for example, `recipes` or `cooking`).
3. Browse the results. The search covers titles, addresses, and descriptions, so even partial memories help.

---

### Scenario: You just switched browsers and want to bring your bookmarks over

1. Export your bookmarks from your old browser (see the import instructions above).
2. Click **"Import"** in the app.
3. Drop the exported file into the import window.
4. The app adds everything new and skips what you already have.
5. After importing, check the tag sidebar — your old browser folders are now tags you can filter by.

---

### Scenario: You have hundreds of bookmarks and want to clean up old ones

1. Click a tag in the sidebar to see bookmarks under a specific category.
2. Look through the list for outdated bookmarks.
3. Click the trash icon on any you no longer need, and confirm deletion.
4. Repeat for each tag category.

---

### Scenario: You want to find all bookmarks related to two topics at once

For example, you want to see everything tagged both **"python"** and **"tutorial"**:

1. Click **"python"** in the tag sidebar — it highlights.
2. Click **"tutorial"** — both are now highlighted.
3. The list shows only bookmarks that have **both** tags.
4. When done, click **"Clear filters"** to see all bookmarks again.

---

## Troubleshooting

### "I can't save a bookmark — it says the URL already exists"

This means you've already saved that web address. The app won't create duplicates.

**To fix**: Close the form and use the search bar to find the existing bookmark. You can then edit it to update the title, description, or tags.

---

### "I pasted a link but the form won't let me save it"

This usually happens because the link isn't a standard web address. The app only accepts addresses starting with `http://` or `https://`.

**To fix**:
- Make sure you copied the full address from your browser's address bar (not just the page title or a shortened link).
- If the address starts with something else (like `ftp://` or `file://`), it won't be accepted.

---

### "My import file shows an error: 'Invalid file type'"

The app only accepts `.html` and `.json` bookmark export files.

**To fix**: Make sure you used your browser's built-in **"Export Bookmarks"** feature (not a screenshot or PDF). The file it creates should end in `.html`. If you saved the wrong file, go back to your browser and export again.

---

### "Some bookmarks failed to import"

A small number of bookmarks in your browser may have unusual or broken links that the app can't save.

**To fix**: Check the error details shown in the import summary. Each failed bookmark will show its address and the reason. You can add those manually using the **"Add Bookmark"** button if they're important.

---

### "My bookmarks are gone after I close the app"

This can happen if the app's data file was moved or deleted.

**To fix**: Ask your system administrator to check that the database file (`bookmarks.db`) is still in the correct location. If it's been accidentally deleted, the data is unfortunately not recoverable — this is why regular backups of the data folder are recommended.

---

### "The search bar doesn't seem to be working"

The search updates after a brief half-second pause (to avoid slowing things down while you type).

**To fix**: Wait a moment after typing, then check if the results updated. If you typed a word and nothing appeared, try a shorter or different keyword — search only covers titles, web addresses, and descriptions, not tags.

---

### "I clicked a bookmark title but nothing opened"

Your browser may have blocked the new tab from opening.

**To fix**: Look for a pop-up blocker notification in your browser's address bar and click **"Allow"**. Then click the bookmark title again.

---

## FAQs

### Can I use this on my phone?

The app works in a mobile web browser, but it's designed primarily for desktop use. The layout adapts to smaller screens, but some features (like the tag sidebar) may be more compact.

---

### Are my bookmarks private?

Yes. The app runs on your own computer and stores everything locally. No data is sent to any external service. Only people with access to the same computer or network can reach the app.

---

### Can I share my bookmarks with someone else?

Not directly — this is a single-person app. However, you can export your bookmarks from your browser and give the file to someone else, who can import it into their own instance of the app.

---

### Can I undo a delete?

No. Once you confirm deletion, the bookmark is gone. Always double-check before confirming.

---

### I have thousands of bookmarks. Will the app handle them?

Yes. The app is built for large collections and uses efficient search technology (SQLite full-text search) that stays fast even with many thousands of bookmarks. Importing a large browser bookmark file may take a few seconds.

---

### Why did my browser's folder structure become tags?

When you export bookmarks from a browser, folders are the only organizational structure. Tags are the equivalent in this app. The import automatically converts each folder into a tag, so your organization is preserved — just in a different form.

---

### Can I have a bookmark without any tags?

Absolutely. Tags are optional. Bookmarks without tags are still fully searchable by keyword.

---

### Can I change the sort order?

Yes. When browsing your bookmarks, you can sort by:
- **Most recently added** (default)
- **Most recently updated**
- **Alphabetical by title**

Use the sort controls above the bookmark list to change the order.

---

## Getting Help

This is a self-hosted personal app. If something isn't working:

1. **Check this guide** — most common issues are covered in the Troubleshooting section above.
2. **Restart the app** — stop and start the server again; this fixes most temporary issues.
3. **Check the server logs** — if you or someone set up the app, the terminal window running the server shows detailed error messages.
4. **Report a bug** — if you believe something is broken, note down exactly what you did, what you expected to happen, and what happened instead. This makes it much easier to diagnose.

When asking for help, useful information to share:
- What browser you're using
- What step you were on when the problem occurred
- The exact error message you saw (if any)
- Whether the problem happens every time or only sometimes
