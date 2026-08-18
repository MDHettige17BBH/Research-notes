# Research-notes

Distilled security knowledge, organized by **vulnerability class** — not
by where it came from. This is a knowledge base, not a reading log:
material only counts once it's been worked through and written in my
own words, not just saved.

## Structure

```
Research-notes/
├── README.md
├── Template.md
├── access-control-idor/     ← concepts, techniques, cheat sheets for IDOR / broken access control
├── general-techniques/      ← cross-cutting skills not tied to one vuln class (e.g. nmap-fundamentals.md)
└── journal/                 ← raw, dated, unedited capture — the record before distillation
```

`authentication/` and `business-logic/` will follow the same pattern as
`access-control-idor/` once there's real content to put in them — empty
placeholder folders don't get created ahead of the work.

## Focus areas

Three vulnerability classes, deliberately narrow:

- **IDOR / Broken Access Control**
- **Authentication**
- **Business Logic**

Adjacent topics (OAuth, JWT, API security, GraphQL) get folders if and
when they produce real notes — not before. Everything outside these
areas (XSS, SQLi, and similar) is intentionally out of scope here; three
sharp focuses beats a wide, thin spread.

## How material gets in here

1. Raw capture — reading a disclosed report, doing a lab, working through
   a blog post — lands in `journal/`, dated, and is never edited after
   the fact. It's a record of what happened, not a polished reference.
2. Once actually understood, the insight gets **merged into** the
   relevant concept file under its vuln-class folder — not saved as a
   new, separate file. Existing files get better over time; they don't
   get siblings.
3. Source platform (OWASP, HackerOne, Bugcrowd, Medium, etc.) is tracked
   as a line in `reading-list.md` once that file exists — never as its
   own folder. Nobody mid-hunt thinks "what did Medium teach me," they
   think "what do I know about IDOR."

## Rule

If a `journal/` entry never gets integrated into a concept file, it
didn't count as learning — it's just something that got read.
