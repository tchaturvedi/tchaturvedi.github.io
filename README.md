# tchaturvedi.github.io

Personal portfolio site, built with Hugo. Live at
[tchaturvedi.github.io](https://tchaturvedi.github.io/).

Leads with the business problem behind the portfolio (`chamberlain` and friends) and stays
honest about what's actually built versus planned — see the first post,
["Why this portfolio is shaped the way it is"](content/posts/why-this-shape.md).

`surprises.md` in this repo is the central, append-only log of what went wrong across every
repo in the portfolio — it lives here because this is where the written content lives, not
duplicated per-repo.

## Develop

```
hugo server -D
```

## Build

```
hugo --gc --minify
```

Matches what CI (`.github/workflows/deploy.yml`) runs on push to `main`; output goes to
`public/` and deploys to GitHub Pages.
