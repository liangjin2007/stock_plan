# AGENTS.md

## What this repo is

Thin aggregator for OpenCode/Claude skills. The root only tracks `.gitmodules` and one git submodule:

- `opencode/skills/jingshui` → https://github.com/Ming-H/jingshui.git (branch `main`)
- Parent remote: https://github.com/liangjin2007/stock_plan.git

There is no app code, build, lint, test suite, CI, or package manifest here. Do not invent or run them.

## Layout gotchas

- Tracked skill source is `opencode/skills/jingshui` (**no leading dot**). Edit content there.
- `.opencode/` is local OpenCode runtime state and is fully gitignored. Never edit or commit it.
- `.opencode/skills/` is empty; this repo does not auto-load the skill. Installation is per the skill README (copy to a skills directory such as `~/.claude/skills/`).
- Content-in-dot-dir confusion: only `opencode/` is versioned.

## Submodule workflow

All substantive changes are commits in the submodule, then a pointer bump in the parent:

```powershell
git -C opencode/skills/jingshui status
git -C opencode/skills/jingshui add -A
git -C opencode/skills/jingshui commit -m "..."
git -C opencode/skills/jingshui push origin main
git add opencode/skills/jingshui
git commit -m "Update jingshui submodule"
```

- `git add` in the parent only records the new submodule commit; content edits must be committed inside the submodule first.
- Clone/restore with `git submodule update --init --recursive`.
- Never rewrite submodule history or force-push `main`; it is a public upstream repo (`Ming-H/jingshui`) not owned by this parent repo.

## Regenerating the source index

Only script: `scripts/build_collection.py` (Python 3, stdlib only). It rebuilds `原文索引.md` and an external full-text collection from MediaCrawler JSON.

```powershell
python opencode\skills\jingshui\scripts\build_collection.py `
  --json-dir "D:\path\to\MediaCrawler\data\zhihu\json" `
  --out-collection "D:\path\to\静水2008-知乎合集.md"
```

- Defaults are hardcoded macOS paths (`/Users/z/...`); always pass `--json-dir` and `--out-collection` on Windows.
- `--out-index` defaults to the submodule's `原文索引.md`; the JSON dir must contain `creator_contents_*.json` and `creator_creators_*.json` (newest filename wins by sort).
- `原文索引.md` is generated; do not hand-edit it.
- Re-crawl updates must be synced across `SKILL.md`, `README.md`, `原文索引.md` (counts + "最近抓取/更新" dates), and any new examples in `references/财报范例/`.

## Content conventions

- All skill content is Chinese. Preserve Chinese section numbering (`一、二、…`, `1、2、…`) and the fixed six-section financial-report template.
- `SKILL.md` frontmatter `name: jingshui` must match the directory; its long `description` is routing/trigger text, not a summary. Keep the Chinese trigger phrases when editing it.
- Two functions, one skill: methodology in `SKILL.md`; financial-report analysis in `references/财报分析框架.md` with few-shot examples in `references/财报范例/`. Load references only when needed.
- The framework assumes external skills `cninfo-disclosure`, `hithink-finance-query`, and `tenscreener`; they are referenced but not included here, so verify availability before relying on them.
- Every report and disclaimer must say it is not investment advice, cite 静水2008, and refuse buy/sell recommendations for specific tickers. Keep the exact disclaimer format from the framework/examples.
- Original prose is the author's copyright; structure/docs are MIT. Do not remove attribution/source links when editing.
