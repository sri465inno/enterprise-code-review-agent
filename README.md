# Enterprise Code Review plugin

A Devin plugin that reviews pull requests against:

1. **Functional / non-functional requirements** from the Jira story or epic linked to the PR (via the Atlassian MCP). For an epic, its child stories are included. Without Jira access, requirements can be pasted into the review request, placed in a requirements Markdown file in the reviewed repo, or written in the PR description.
2. **Enterprise standards** stored as Markdown in `skills/enterprise-code-review/standards/`.

It also reviews the requirements themselves and reports gaps (missing, ambiguous or untestable acceptance criteria, unhandled scenarios, missing NFRs) as suggestions for the product owner. The PR summary starts with a plain-language section for non-technical stakeholders; developer detail is collapsed below it.

## Layout
```
enterprise-code-review-agent/
├── .devin-plugin/plugin.json
├── skills/enterprise-code-review/
│   ├── SKILL.md                  # review procedure + report format
│   └── standards/                # your enterprise "skills" — one file per standard
│       ├── coding-standards.md
│       ├── approved-libraries.md
│       ├── tech-stack.md
│       ├── nfr-checklist.md
│       └── requirements-quality.md  # applied to the Jira story/epic
└── templates/REVIEW.md           # optional: copy into target repos for built-in Devin Review
```

## Adding or updating standards
- Add or edit any `*.md` file in `standards/`. Every file is loaded on each review.
- Put checkable rules under a `## Rules` heading, one bullet per rule, each with a unique ID in brackets, e.g. `- [SEC-07] ...`. The ID is used in the review's traceability tables and inline comments.
- Commit to the default branch; the next review picks it up.
- Alternatively, zip this folder and use Customize → Plugins → Add plugin → **Upload .zip**, or edit the installed plugin in the web editor.

## Install (org admin)
Customize → Plugins → Add plugin → **From repository**: enter `sri465inno/enterprise-code-review-agent` (repo root), scope **Organization**.

## Prerequisites
- Optional: Atlassian MCP connected at the organization scope (Jira read access). Without it, use one of the non-Jira requirements sources below.
- GitHub integration with access to the repositories being reviewed.

## Running a review
- Manually: start a Devin session with the `!enterprise_review` playbook and a PR URL (optionally a Jira key).
- Automatically: a Devin Automation on `pull_request` opened / ready_for_review, or a PR comment starting with `/review-reqs`.

Requirements source (first available wins; the report states which was used):
1. Jira: key (story or epic) passed explicitly or taken from the PR title, branch name, or body (pattern `ABC-123`).
2. Requirements text pasted in the request.
3. A requirements file: the given path, else `REQUIREMENTS.md`, `requirements.md`, `docs/requirements.md`, or `docs/requirements/*.md` in the PR branch.
4. A `Requirements` / `Acceptance criteria` / `NFRs` section in the PR description.

Example without Jira:
```
!enterprise_review https://github.com/org/repo/pull/42
Acceptance criteria:
- Given a guest with a confirmed booking, when they cancel 48h+ before check-in, then they receive a full refund.
Non-functional:
- Cancellation API responds in < 500 ms p95.
```
