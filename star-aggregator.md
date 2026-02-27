# ⭐ GitHub Star Aggregator

A simple GitHub Action that counts the total stars across your pinned (or any) repositories and displays a live-updating badge on your GitHub profile README.

![Total Stars](https://img.shields.io/badge/total__stars-193-yellow?style=for-the-badge&logo=github)

## How It Works

A GitHub Action runs every 6 hours (or on demand), queries the GitHub API for star counts on your specified repos, sums them up, and updates a badge in your profile README automatically.

## Setup (5 minutes)

### 1. Create your profile repo (skip if you already have one)

Your profile README lives in a repo named the same as your GitHub username. For example, if your username is `octocat`, create a repo called `octocat` with a `README.md` file.

### 2. Add the badge to your README

Paste this line wherever you want the badge to appear in your `README.md`:

```markdown
![Total Stars](https://img.shields.io/badge/total__stars-0-yellow?style=for-the-badge&logo=github)
```

> **Important:** Shields.io uses `-` as a separator and `_` as a space. To display a literal underscore, use `__` (double underscore). This is why the URL has `total__stars` and not `total_stars`.

Make sure the badge is on its own line with a blank line before and after it, or GitHub won't render it as an image.

### 3. Add the workflow file

Create the file `.github/workflows/update-stars.yml` in your profile repo with the following contents:

```yaml
name: Update Total Stars

on:
  schedule:
    - cron: '0 */6 * * *'  # every 6 hours
  workflow_dispatch:  # manual trigger

jobs:
  update-stars:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Count stars and update README
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');

            // ADD YOUR REPOS HERE
            // format: { owner: 'github-username-or-org', repo: 'repo-name' }
            const repos = [
              { owner: 'YOUR_USERNAME', repo: 'your-repo-1' },
              { owner: 'YOUR_USERNAME', repo: 'your-repo-2' },
              // add as many as you want
            ];

            let totalStars = 0;

            for (const { owner, repo } of repos) {
              try {
                const { data } = await github.rest.repos.get({ owner, repo });
                totalStars += data.stargazers_count;
                console.log(`${owner}/${repo}: ${data.stargazers_count} ⭐`);
              } catch (e) {
                console.log(`failed to fetch ${owner}/${repo}: ${e.message}`);
              }
            }

            console.log(`\ntotal: ${totalStars} ⭐`);

            let readme = fs.readFileSync('README.md', 'utf8');

            readme = readme.replace(
              /!\[Total Stars\]\(https:\/\/img\.shields\.io\/badge\/total__stars-\d+-yellow\?style=for-the-badge&logo=github\)/,
              `![Total Stars](https://img.shields.io/badge/total__stars-${totalStars}-yellow?style=for-the-badge&logo=github)`
            );

            fs.writeFileSync('README.md', readme);

      - name: Commit changes
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git diff --quiet || (git add README.md && git commit -m "update total stars count" && git push)
```

### 4. Enable workflow write permissions

Go to your profile repo's **Settings > Actions > General**, scroll to **Workflow permissions**, select **Read and write permissions**, and save.

### 5. Run it

Go to the **Actions** tab in your repo, click **Update Total Stars** in the sidebar, then click **Run workflow**. The badge will update within a few seconds.

After the first run, the action will automatically run every 6 hours to keep your star count current.

## Customization

### Badge style

Change the badge appearance by modifying the shields.io URL in the workflow:

| Style | URL snippet |
|-------|------------|
| Default (flat) | `?style=flat` |
| For the badge | `?style=for-the-badge` |
| Flat square | `?style=flat-square` |
| Plastic | `?style=plastic` |

### Badge color

Replace `yellow` in the URL with any color:

```
https://img.shields.io/badge/total_stars-100-blue?style=for-the-badge&logo=github
```

Options: `brightgreen`, `green`, `yellow`, `orange`, `red`, `blue`, `lightgrey`, or any hex code like `ff69b4`.

### Include all public repos instead of specific ones

Replace the `repos` array section in the workflow with:

```javascript
// fetch all public repos for a user
const allRepos = await github.paginate(github.rest.repos.listForUser, {
  username: 'YOUR_USERNAME',
  type: 'public',
  per_page: 100,
});

let totalStars = 0;
for (const repo of allRepos) {
  totalStars += repo.stargazers_count;
}
console.log(`total across ${allRepos.length} repos: ${totalStars} ⭐`);
```

### Update frequency

Change the cron schedule in the workflow:

```yaml
schedule:
  - cron: '0 */12 * * *'  # every 12 hours
  - cron: '0 0 * * *'     # once a day at midnight
  - cron: '0 0 * * 1'     # once a week on monday
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Badge shows as raw text, not an image | Make sure the badge is on its own line with blank lines before and after it. Remove any HTML comments directly touching the markdown |
| Badge label shows spaces instead of underscores | Use `__` (double underscore) in the shields.io URL. Single `_` renders as a space |
| Action fails with `Permission denied` | Enable **Read and write permissions** in Settings > Actions > General > Workflow permissions |
| Badge shows `0` | Make sure your repo list in the workflow matches your actual repo names exactly |
| Action doesn't appear in Actions tab | Make sure the `.yml` file is on your default branch (usually `main`) |
| Badge doesn't update | Check the Actions tab for failed runs and review the error logs |

## Example

Here's what it looks like in a profile README:

```markdown
# hey, i'm jia!

[![Instagram](https://img.shields.io/badge/Instagram-71k-E4405F?style=for-the-badge&logo=instagram&logoColor=white)](https://instagram.com/jia.seed)

![Total Stars](https://img.shields.io/badge/total__stars-193-yellow?style=for-the-badge&logo=github)

...rest of your readme
```

## License

Do whatever you want with this. No attribution needed.
