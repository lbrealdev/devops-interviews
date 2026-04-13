# Hosting on GitHub Pages

This is a simple client-side web app that can be hosted for free on GitHub Pages.

## Steps

### 1. Create a GitHub Repository

Create a new public repository on GitHub (or use an existing one).

### 2. Upload the Files

Upload the `interview-form/` folder contents to your repository:

```
your-repo/
└── index.html
```

You can do this via:
- **GitHub Web UI**: Drag and drop the file
- **Git CLI**: `git clone` your repo, copy `index.html`, commit, push

### 3. Enable GitHub Pages

1. Go to your repository on GitHub
2. Click **Settings** → **Pages** (in sidebar)
3. Under **Source**, select:
   - **Branch:** `main` (or your preferred branch)
   - **Folder:** `/ (root)` or `/docs`
4. Click **Save**

### 4. Access Your App

After a few minutes, your app will be available at:

```
https://yourusername.github.io/your-repo-name/
```

For example: `https://username.github.io/interview-notes/`

## Notes

- The app is fully client-side - no server required
- All data stays in the browser (localStorage optional, if you add that feature later)
- GitHub Pages supports custom domains if needed
