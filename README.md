# fin-site

Source of <https://fin.jdkno.com>, served by GitHub Pages from `main` (root). Pages are plain HTML:
`index.html`, `privacy/`, `terms/`, `refunds/`, `plus/` (the plans and prices page), `support/`, with images and fonts under `web/`.

`README.md` is not published: `_config.yml` excludes it from the Jekyll build that GitHub Pages runs. If a
`.nojekyll` file is ever added (turning Jekyll off), this file would be published at `/README.md`; exclude
it another way first.

## Language of a page

Every page opens in the language the visitor last chose (kept in the browser), else the browser's language. A link can force one with
`?lang=es` or `?lang=en` (for example `https://fin.jdkno.com/terms/?lang=es`); the app uses this to show its own language. The visitor can
still switch with the EN / ES buttons. The script is the one at the end of each page.

## Tutorial videos

`tutorials/<lang>/<name>-<size>.mp4`, with a poster `tutorials/<lang>/<name>.jpg`.

| | |
|---|---|
| `<lang>` | `en` or `es` |
| `<name>` | `apple-pay`, `notification`, `message` or `wallet-card-name` (this one is vertical, 9:16) |
| `<size>` | `720p` for the app, `1080p` for the web |

Example URL: `https://fin.jdkno.com/tutorials/es/message-720p.mp4`

The files are H.264 with the index at the start (streamable). Keep them that way if you re-export.

### Replacing a video

Overwrite the file(s) and commit. **Git keeps every committed version forever**, so each replaced
video adds roughly its own size to the repository (a few MB to about 25 MB) even though the site
only serves the new one. Unchanged files are never duplicated: committing other site changes does
not add copies of the videos.

Keep an eye on the size with:

```bash
git count-objects -vH                      # size-pack is what a clone downloads
git rev-list --objects --all \
  | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
  | awk '$1=="blob"' | sort -k3 -n | tail -20      # biggest files in all of history
```

### Purging the old versions of the videos from history

Only needed when the repo has grown because videos were replaced. This **rewrites history**: every
commit after the first one that touched `tutorials/` gets a new hash, and the push must be forced.
It is safe while you are the only person using the repo. Anyone else, or any other clone, must
re-clone afterwards.

The idea: remove `tutorials/` from the whole history, then add the current videos back in one commit.
That leaves exactly one copy of each video.

1. **Install the tool once:** `brew install git-filter-repo`.
2. **Save the videos you want to keep** (the current files in your working copy):

   ```bash
   cp -R ~/development/fin-site/tutorials ~/tutorials-keep
   ```

3. **Make a fresh clone to work in** (filter-repo is meant for fresh clones, and it leaves your
   normal clone untouched as a backup):

   ```bash
   git clone https://github.com/jkano/fin-site.git ~/development/fin-site-purge
   cd ~/development/fin-site-purge
   ```

4. **Remove the folder from all of history:**

   ```bash
   git filter-repo --path tutorials --invert-paths
   ```

5. **Put the current videos back and commit them once:**

   ```bash
   cp -R ~/tutorials-keep tutorials
   git add tutorials
   git commit -m "Add tutorial videos"
   ```

6. **Push the rewritten history.** filter-repo removes the `origin` remote on purpose:

   ```bash
   git remote add origin https://github.com/jkano/fin-site.git
   git push --force origin main
   ```

7. **Switch to the clean clone.** Rename or delete the old `fin-site` folder after checking the
   site still loads, and use `fin-site-purge` (rename it to `fin-site`). Check the result with
   `git count-objects -vH`.

Things to know:

- GitHub does not free the space at once. The old objects stay reachable by their commit hashes
  for a while, and the repository size on GitHub shrinks after its own cleanup. If you need them
  gone sooner, ask GitHub Support to run garbage collection on the repository.
- Nothing about the live site changes: Pages republishes from the new `main`, and `CNAME` stays
  in the repo.
- The same recipe works for any folder: replace `tutorials` with the path to purge.
