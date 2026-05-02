# Functional Flows — Bookmarks Manager Web App

## Actors

| Actor | Role | Permissions |
|---|---|---|
| User | The sole person using the application | Full access: save, view, edit, delete, search, filter, and import bookmarks |

> This is a single-user personal application. There is no login, no accounts, and no sharing. Everything the User does affects their own personal bookmark collection.

---

## Flows by Actor

### User

---

#### Flow: Save a new bookmark

**Goal**: The User wants to save a web page so they can find it later.

**Steps**:
1. The User clicks the **"Add Bookmark"** button at the top of the page.
2. A form appears asking for a web address (required), a title, a description, and tags.
3. The User types or pastes the web address of the page they want to save.
4. Optionally, the User types a title to help them remember what the page is about.
5. Optionally, the User adds a short description with more details.
6. Optionally, the User types tags (like "dev", "tools", "recipes") to group the bookmark with others. Each tag is added by pressing Enter or comma.
7. The User clicks **"Save"**.
8. The system saves the bookmark and closes the form.
9. The new bookmark appears in the list immediately.
10. A brief "Bookmark saved!" confirmation message appears at the bottom of the screen.

**Outcome**: The bookmark is saved and visible in the list with all its details.

**Possible errors**:
- *Missing web address*: The system highlights the web address field in red and shows "Invalid URL". The form stays open so the User can fix it.
- *Web address already saved*: The system shows "Bookmark with this URL already exists". The form stays open. The User can close it and edit the existing one instead.
- *Invalid web address format*: The system highlights the field and shows a validation message. The User corrects the address.

---

#### Flow: Edit an existing bookmark

**Goal**: The User wants to update the title, description, or tags of a bookmark they already saved.

**Steps**:
1. The User finds the bookmark they want to change (by scrolling, searching, or filtering).
2. The User clicks the **pencil (edit) icon** on the bookmark card.
3. The form appears pre-filled with the bookmark's current title, description, and tags.
4. The User changes any of the fields they want.
5. To remove a tag, the User clicks the **×** on the tag chip. To add a tag, they type and press Enter.
6. The User clicks **"Save"**.
7. The system updates the bookmark immediately.
8. A "Bookmark saved!" confirmation appears.

**Outcome**: The bookmark now shows the updated information.

**Possible errors**:
- *Invalid web address*: Same as when creating — the field is highlighted and the User must fix it.
- *Web address already used by another bookmark*: The system shows a conflict message. The User must use a different address or close without saving.

---

#### Flow: Delete a bookmark

**Goal**: The User wants to permanently remove a bookmark they no longer need.

**Steps**:
1. The User finds the bookmark they want to delete.
2. The User clicks the **trash (delete) icon** on the bookmark card.
3. The system asks: *"Are you sure you want to delete this bookmark?"* with a Confirm and Cancel option.
4. The User clicks **"Confirm"**.
5. The bookmark disappears from the list immediately.

**Outcome**: The bookmark is permanently removed. This cannot be undone.

**Possible errors**:
- *User clicks Cancel*: Nothing happens. The bookmark remains.

---

#### Flow: Browse all bookmarks

**Goal**: The User wants to see all their saved bookmarks.

**Steps**:
1. The User opens the application. All bookmarks are shown automatically, most recently added first.
2. Bookmarks are displayed as cards showing the website's icon, title, address, description preview, and tags.
3. If there are more bookmarks than fit on one page, navigation controls appear at the bottom.
4. The User clicks **Next** or a page number to see more bookmarks.
5. The User can switch between **grid view** (multiple columns) and **list view** (single column) using the toggle buttons at the top right.

**Outcome**: The User sees their bookmarks organized and easy to browse.

**Possible errors**:
- *No bookmarks saved yet*: The page shows a message: *"No bookmarks yet. Add your first bookmark or import from browser."*

---

#### Flow: Search for bookmarks by keyword

**Goal**: The User wants to find bookmarks about a specific topic without scrolling through everything.

**Steps**:
1. The User types a word or phrase into the **search bar** at the top of the page.
2. After a brief pause (about a third of a second), the bookmark list automatically updates to show only bookmarks whose title, web address, or description contains the typed word.
3. The number of matches is shown.
4. The User can clear the search by clicking the **×** button in the search bar, which brings back all bookmarks.

**Outcome**: The User sees only the bookmarks relevant to their search.

**Possible errors**:
- *No matches found*: The list shows an empty state message: *"No bookmarks match your search."* The User can try a different word or clear the search.

---

#### Flow: Filter bookmarks by tag

**Goal**: The User wants to see all bookmarks under a specific category (tag).

**Steps**:
1. The **tag panel** on the left side of the screen shows all the tags the User has used, along with the number of bookmarks each tag has.
2. The User clicks on a tag name (for example, "recipes").
3. The bookmark list instantly updates to show only bookmarks tagged with "recipes".
4. The selected tag is highlighted to show it's active.
5. The User can click additional tags to narrow results further — the list then shows only bookmarks that have **all** the selected tags.
6. To remove a filter, the User clicks the active tag again to deselect it.
7. To remove all tag filters at once, the User clicks **"Clear filters"** at the top of the tag panel.

**Outcome**: The User sees only the bookmarks that match their chosen tags.

**Possible errors**:
- *No tags yet*: The tag panel shows *"No tags yet"* until at least one bookmark with a tag is saved.
- *Combination returns no results*: Selecting multiple tags that no single bookmark has returns an empty list. The User deselects one or more tags to broaden the filter.

---

#### Flow: Filter and search together

**Goal**: The User wants to find a specific bookmark within a tag category.

**Steps**:
1. The User selects one or more tags from the tag panel to filter by category.
2. The User also types a keyword in the search bar.
3. The list shows only bookmarks that **both** match the keyword **and** have all the selected tags.
4. Clearing the search bar or deselecting tags broadens the results accordingly.

**Outcome**: The User quickly pinpoints the specific bookmark they're looking for.

---

#### Flow: Import bookmarks from a browser

**Goal**: The User wants to bring in their existing bookmarks from Chrome, Firefox, or Edge all at once.

**Steps**:
1. The User clicks the **"Import"** button at the top of the page.
2. A window appears with a file drop zone.
3. The User exports their bookmarks from their browser (typically: Browser Menu → Bookmarks → Export Bookmarks). This creates a file on their computer (usually named `bookmarks.html` or similar).
4. The User either **drags and drops** the exported file onto the drop zone, or clicks the zone to browse their computer and select the file.
5. The system checks the file is valid (must be `.html` or `.json`, and under 10MB).
6. The User clicks **"Import"** and sees a spinning indicator while the system processes the file.
7. When finished, the system shows a summary:
   - ✅ *"3 bookmarks added"*
   - ⏭ *"12 skipped (already in your collection)"*
   - ❌ *"1 failed"* (with details if any had problems)
8. The User clicks **"Done"** to close the window, or **"Import another file"** to repeat the process.
9. The bookmark list and tag panel refresh automatically with the newly imported bookmarks.

**Outcome**: The User's browser bookmarks are now in their collection. Bookmarks they already had saved are not duplicated. Folders from the browser become tags on the imported bookmarks.

**Possible errors**:
- *Wrong file type*: If the User selects a file that isn't `.html` or `.json`, the system immediately shows *"Invalid file type"* without trying to upload it. The User selects the correct file.
- *File too large*: If the file exceeds 10MB, the system shows *"File exceeds maximum allowed size"*. This is rare for bookmark files.
- *Unrecognizable file format*: If the file is `.html` or `.json` but not in a recognized bookmark format, the system shows a parse error. The User tries exporting again from their browser using the standard export option.
- *Some bookmarks failed*: Individual bookmarks with invalid or unsupported web addresses are listed in the error details. The User can add them manually if needed.

---

#### Flow: Click through to a saved bookmark

**Goal**: The User wants to visit a web page they previously saved.

**Steps**:
1. The User finds the bookmark in their list (by browsing, searching, or filtering).
2. The User clicks the **title** of the bookmark card.
3. The web page opens in a new browser tab.

**Outcome**: The bookmarked web page opens without navigating away from the bookmarks app.

---

#### Flow: Switch between grid and list view

**Goal**: The User prefers to see bookmarks in a different layout.

**Steps**:
1. At the top right of the page, the User sees two layout icons: a **grid icon** and a **list icon**.
2. The User clicks the layout they prefer.
3. The bookmark list immediately switches to that layout.
4. The preference is remembered — the next time the User opens the app, the same layout is used.

**Outcome**: Bookmarks are displayed in the User's preferred arrangement.

---

## State Machines

### Bookmark State Machine

Bookmarks do not have a complex lifecycle — they exist in a single "saved" state. The relevant transitions relate to the **tag associations** on a bookmark:

- **Created**: A new bookmark is added with zero or more tags.
- **Updated**: The User edits any field. Tags can be added or removed entirely.
- **Deleted**: The bookmark is permanently removed. All its tag associations are automatically cleaned up.

> If all bookmarks with a particular tag are deleted, that tag's count drops to zero — it may still appear in the tag panel until the page refreshes, then disappears naturally.

---

### Import Process State Machine

| State | What the User Sees | How to Move Forward |
|---|---|---|
| **Idle** | Drop zone with instructions | Drop or select a file |
| **Validating** | Instant — error shown if invalid | Fix file selection |
| **Importing** | Spinner: *"Importing bookmarks..."* | Wait for completion |
| **Success** | Summary: added / skipped / failed counts | Click "Done" or "Import another file" |
| **Error** | Error message with **Retry** button | Click Retry to return to Idle |

---

## Business Rules

1. A bookmark's **web address must be unique** — the same web address cannot be saved twice.
2. A bookmark's **web address is required** — a bookmark cannot be created without one.
3. Only **web addresses starting with `http://` or `https://`** are accepted — other types of links are rejected for safety.
4. Tags are automatically converted to **lowercase** — "Dev" and "dev" are treated as the same tag.
5. Tags have **leading and trailing spaces removed** — " tools " becomes "tools".
6. When a browser bookmark **folder structure** is imported, each folder in the path becomes a tag on that bookmark.
7. During import, bookmarks whose web address **already exists** in the collection are **skipped** (not duplicated) and counted as "skipped".
8. During import, the entire batch is processed as a unit — if the file itself is unreadable, no bookmarks are added.
9. **Deleting a bookmark is permanent and immediate** — there is no recycle bin or undo.
10. Tag filtering uses **AND logic** — selecting two tags shows only bookmarks that have **both** tags, not bookmarks that have either one.
11. Search matches are found in the **title, web address, and description** of a bookmark — tags are not searched by the text search bar (use the tag panel for tag-based filtering).
12. The bookmark list is sorted by **most recently added first** by default. The User can change the sort to most recently updated or alphabetical by title.
13. Imported bookmarks from a browser **preserve the original title** from the export file. If the title is empty in the export file, the web address is used as the display title.
14. The maximum import file size is **10 megabytes**. Standard browser bookmark exports are well under this limit.
