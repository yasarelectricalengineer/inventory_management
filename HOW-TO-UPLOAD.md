# Single-file build (Version C)

This folder has ONE app file: `index.html`. The CSS and JavaScript are inside it, so nothing else needs to be uploaded. It is the same app as Version C (same features, same data storage, same app version `2026.10.24-C`).

## Put it on GitHub Pages
1. Open your repository on github.com.
2. If the old files are there, you can delete the old `css` and `js` folders and the old `index.html` (optional; they are not needed any more).
3. Click **Add file > Upload files** and drag in **only `index.html`** from this folder. Click **Commit changes**. (If a file called `index.html` already exists, this replaces it.)
4. Repository **Settings > Pages**: Source = *Deploy from a branch*, Branch = `main`, Folder = `/ (root)`, then Save.
5. Wait 1 to 2 minutes, then open `https://YOUR-USERNAME.github.io/YOUR-REPO/` and press **Ctrl+F5** once.

## If it still looks wrong
- Check that the page address ends with your repository name and that `index.html` is in the repository's top level (not inside another folder).
- Open the **Actions** tab in the repository: a failed "pages build and deployment" shows the reason.
- Press F12, open **Console**, and send the red text.

## Your data
Browser data belongs to the exact web address. A new address (for example github.io instead of localhost or file) starts empty. Use **Export** on the old address and **Import** on the new one.
