# dfu99.github.io

Jekyll + minima, published by GitHub Pages from `main`. Posts live in
`_posts/YYYY-MM-DD-name.markdown` and get the default permalink
`/YYYY/MM/DD/name.html` unless they set `categories`.

## Writing rules for posts and project write-ups

These are hard constraints, not style preferences. They were given as a
correction; violating them means the post gets rewritten.

**Report results, not process.** The unit of a post is a confirmed
intermediate result — a `tasks/objectives.yaml` entry in the source project —
with its real numbers, ordered so the argument builds. An agent's exploration
path is a random walk, not a story; it reads as filler and it is not what the
work established.

**No meta-commentary.** Cut anything about how the work or the writing went:
"my first read was", "the part I got wrong", "that probably deserves its own
post", "the thing I keep repeating to people", asides about earlier drafts or
earlier framings. Cut rhetorical turns that exist to build suspense ("It
doesn't.", "the obvious move is", "that's the bottleneck I wanted to attack").

**Descriptive section headings.** "The genu lock", "Model limitations",
"Tool-calling: insufficient" — name the content. Not "Four Ways an MD Result
Can Be a Believable Wrong Answer", not "The first architecture didn't work".

**Keep the negatives.** Null models, failed predictions, caveats and model
limitations stay in — they are findings. State them as properties of the
result, not as personal error:

- yes: "Zero force is the wrong ensemble; the barrier is absent."
- no:  "I thought it was under-sampled, but I was wrong."

**Every number comes from a project artifact.** Source posts from
`tasks/objectives.yaml`, `tasks/planning.md` checkpoints, paper drafts and
figure scripts in the relevant repo. Do not generate plausible-sounding
numbers, dates or venues; verify external facts (conference dates, DOIs,
whether a repo is public) before publishing them.

Applies to Slack checkpoint reports too.

## Assets

Images go in `images/YYYY-MM-DD/` for posts, `images/research/` for the
research page. Cap raster width at ~1400px (2x a 700px display) and quantize
PNGs to 256 colors — `sips -Z 1400` then Pillow `quantize`, since no
`pngquant`/`optipng` is installed. The gallery PNG went 724K -> 92K that way.

## Gotchas

- Retitling a post: change front-matter `title:` only. Renaming the file
  breaks the permalink and every link into it from `research.markdown`.
- `bundle exec jekyll serve` does not run here — system Ruby 2.6 lacks the
  bundler version in `Gemfile.lock`. Verify front matter and that every
  `/images/...` path resolves on disk instead of rendering a preview.
- Commit and push only when asked. GitHub Pages publishes on push.
