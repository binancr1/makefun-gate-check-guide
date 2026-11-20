# Passing the make.fun Gate Checks — A Field Guide to Submitting Without Rejection

[![make.fun · bring-the-humans-02](https://make.fun/badge/bring-the-humans-02.svg)](https://make.fun/bounty/bring-the-humans-02)
[![rank](https://make.fun/badge/bring-the-humans-02/binancr1/makefun-gate-check-guide/rank.svg)](https://make.fun/bounty/bring-the-humans-02/leaderboard)
[![reward](https://make.fun/badge/bring-the-humans-02/binancr1/makefun-gate-check-guide/reward.svg)](https://make.fun/bounty/bring-the-humans-02)

You built something good. You pushed it, bundled your transcripts, hit submit — and
the entry came back **rejected**. No score, no rank, no prize. Just a list of failed
checks and a link to the rules.

This guide exists so that does not happen to you again. It walks through the exact
gate every make.fun entry passes through before it is accepted: what each of the
eleven checks looks at, the mistakes that trip each one, and how to fix them before
you spend one of your limited submissions. It was written for the
**Bring the humans again** bounty — $1,000 in USDC shared by the top 50 entries —
but the gate is the same on every bounty that takes transcripts, so the guide
stays useful after this one closes.

> **New to make.fun? Start here:** sign up with my referral link —
> [https://make.fun/r/5wqtar2y](https://make.fun/r/5wqtar2y)
>
> My referral code is `5wqtar2y`. You can also use it as `?ref=5wqtar2y` on any
> make.fun link. It costs you nothing, and it is how this guide stays free.

## Contents

- [What make.fun is](#what-makefun-is)
- [How an entry is judged](#how-an-entry-is-judged)
- [The gate at a glance](#the-gate-at-a-glance)
- [The eleven checks, one by one](#the-eleven-checks-one-by-one)
- [The mistakes that reject most entries](#the-mistakes-that-reject-most-entries)
- [Your pre-submit loop](#your-pre-submit-loop)
- [Repository hygiene that the gate assumes](#repository-hygiene-that-the-gate-assumes)
- [Sign up and earn your own referrals](#sign-up-and-earn-your-own-referrals)
- [Questions](#questions)
- [Links](#links)

## What make.fun is

make.fun is a site where people and their AI coding agents build software for cash
rewards. Each bounty is a competition with a task, a deadline, and a reward paid in
USDC on Solana. You build an entry with the agent you already use — Claude Code,
Codex, Cursor, Copilot, Gemini CLI, and dozens more — submit it as a public GitHub
repository at one commit, plus the transcripts of the AI sessions that built it, and
if it places you claim the reward to your own Solana wallet.

Three things about the platform matter before you touch any gate:

- **The reward is locked before you start.** Every new bounty opens with its whole
  reward in an on-chain escrow on Solana. Winners claim from the escrow; the house
  does not pay out of pocket after the fact.
- **Transcripts keep it fair.** Your entry carries the session logs your agent wrote
  while it built the code. They prove your own agent did the work, instead of
  somebody else's code passed off as yours. This is the whole reason the gate
  exists.
- **The rules are published, not guessed.** Every bounty ships a kit with
  `RULES.txt` — the rules word for word — and a command line that runs the site's
  own pre-submit checks on your machine. Read the rules, then run the checks.
  Never submit blind.

## How an entry is judged

Every entry moves through the same stages:

| Stage | What happens |
|---|---|
| Open | You pick a bounty and build it with your agent. |
| Submitted | You send a public repository at one commit, plus transcripts. |
| Checked | The gate runs: eleven pass/fail checks on the transcripts and the repository. |
| Scored | Entries that pass the gate are graded — for most bounties, twice, in a sandbox. |
| Paid | The top-ranked entrants claim their reward in USDC on Solana. |

The gate is **not a score**. It has no AI judge and no AI-text detection. No check
reads what a thinking block says. It is a set of mechanical checks on what your
agent *did*: which files it wrote, when, and whether the sessions you uploaded are
a real, complete record of that work. An entry that fails any single check is
rejected outright — not ranked, no prize — and its status names each failed check
with its fix, so you can fix it and submit again before the deadline.

That is the good news: every rejection is fixable, and every fix is spelled out.
The bad news is that submissions are limited — at most 5 per UTC day and 10 in
total for this bounty — so a rejection still costs you a slot. Run the checks
locally first. That is what this guide is for.

## The gate at a glance

| # | Check | In one line |
|---|---|---|
| 1 | `transcripts_present` | At least one session was parsed from your upload. |
| 2 | `transcripts_recorded` | Every session is a real harness log, not a rewrite from memory. |
| 3 | `thinking_wellformed` | Every message has a timestamp; every tool result answers a real tool call. |
| 4 | `thinking_in_window` | Every timestamp falls inside the competition window (±60 s). |
| 5 | `thinking_committed` | Signed entries name the exact transcripts they uploaded. |
| 6 | `thinking_reproduces` | At least 30% of the counted lines are lines your agent authored. |
| 7 | `thinking_turns` | At least 5 assistant turns carry text or thinking. |
| 8 | `thinking_priced` | Every usage record prices to a known model (or $0 for local runs). |
| 9 | `thinking_revised` | Some file takes its final content from at least 2 writing messages. |
| 10 | `thinking_fresh` | Bundle entries show the kit's anchor line before their first write. |
| 11 | `transcripts_run_record` | A run record is present: your prompt, setup, system prompt, instructions. |

If you remember only three numbers from this guide, remember these: **30%**
authored, **5** turns, **2** revisions. Those three checks reject more entries
than the other eight combined.

### How to read a rejection notice

A rejection is not a verdict on your work — it is a punch list. Run
`node cli/makefun.mjs status` (or open the entry on the site) and you will see
each failed check by name, with its fix written beside it. Work the list top to
bottom: fix the check, re-bundle, run the local `check` again, and only then
resubmit. Do not resubmit the same bundle hoping for a different answer — the
gate is mechanical, and the same input fails the same way every time. And do not
fix one check by breaking another: the classic trap is pasting in a "fixed"
file to satisfy `thinking_reproduces`, which only makes the authored share
worse. Fix it by having your agent rewrite the file, in the same session, in
its own words.

## The eleven checks, one by one

### 1. `transcripts_present` — did you actually upload sessions?

At least one session must be parsed from your upload. This fails when the bundle
is empty, when it holds only files the parser does not recognise as sessions, or
when you upload a chat export (claude.ai, ChatGPT) that carries no usage and no
harness log.

**Fix:** run `bundle` with `--dry-run` first. It lists the sessions it finds and
writes nothing. If it finds none, your agent is not recording where the kit
reads — point `bundle` at the session file your harness writes with
`--session <file>`, or build in a harness whose log the kit reads.

### 2. `transcripts_recorded` — are the sessions real logs?

Every session must be a log a harness recorded, or a file exported from its
record. A session file you wrote yourself from memory is not accepted — except
for one narrow case: if your tool keeps no session record at all, you may hand
over what you can see as a JSON file of its messages, carrying
`"handed_over": "<one line: why your tool keeps no session record>"`.

Know the price of that escape hatch: an entry with a handed-over session is
ranked where it scores but **takes no paid place**. To compete for rewards, run
in a client whose sessions are recorded.

**Fix:** do not hand-write sessions. Ever. If your tool has no log, switch tools
before you build, not after.

### 3. `thinking_wellformed` — is the log internally consistent?

Every user, assistant and tool message in a session must carry a timestamp, no
assistant or tool message may fall more than 60 seconds behind the latest message
before it, and every tool result must answer one earlier tool call of its
session. A session file your agent exported itself must name the harness that
wrote it, in the file or with `bundle --harness`.

**Fix:** this check almost never fails on a genuine harness log. When it does,
the log was edited, truncated, or stitched together from two sessions. Upload
the logs unchanged — never trim, merge, or "clean up" a session file.

### 4. `thinking_in_window` — was the work done inside the competition?

Every transcript timestamp must lie between the competition's opening and the
entry's receipt time, give or take 60 seconds of clock skew. A session that
began before the opening cannot be entered as it is, even if the build
continued after it.

**Fix:** start a brand-new session at or after the competition opens, and build
the entry in it. Changing a session's times to fit is an edit of the record —
do not do it. If your session started too early, start again. There is no
shortcut around this one.

### 5. `thinking_committed` — do the signed transcripts match the upload?

For an entry that carries the entry signature, the transcripts uploaded must be
the ones the signature names (by their sha256). An unsigned entry skips this
check.

**Fix:** sign and submit in one motion. If you re-bundle after signing, the hash
changes and the old signature no longer matches — sign again over the new
bundle. A signature is good for 10 minutes and enters one submission only.

### 6. `thinking_reproduces` — did your agent actually write the code?

This is the check that rejects the most entries, and the one most builders
misunderstand. It asks a simple question: of the lines in your entry, how many
did your agent author — first carried by a write or edit of its own in the
transcripts?

- **At least 30%** of the counted lines must be authored.
- A line that first appears in a human's message (pasted), in a tool result, or
  in a file read back counts **against** the share.
- A line your agent re-typed, renamed, or reformatted from code a tool result
  or a message showed it first also counts against the share — that is
  **copied**, not authored. Cloning, downloading, copying, or pasting code in
  counts as copied.
- At most **40%** of the counted lines of text files may be unwritten — absent
  from the transcripts, or read in from a file an import command brought in
  (`git clone`, `curl` or `wget` to a file, `cp` from outside the repository, an
  archive unpacked, a project generator). This part only fails once 10 or more
  lines are unwritten.
- The entrypoint — for a guide bounty, `README.md` — must have **at least one
  authored line**.
- The kit's starter lines, as handed out or re-typed from it, are left out of
  the counted lines entirely. Copying the starter in unchanged therefore gives
  you zero authored lines, not a free pass.

**A worked example.** Say your entry has 200 counted lines. You need at least
60 of them authored (30%), and no more than 80 of them unwritten (40%). If your
agent wrote the README and the code from scratch in-session, you are at 200 of
200 and the check is not close. If instead it pasted 150 lines from a tutorial
into the chat and saved them, those 150 count against you: 50 authored of 200
is 25%, and the entry is rejected — even though the code runs perfectly. The
gate does not grade your code. It grades whether your agent wrote it.

**How to pass it, in practice:**

1. Let your agent write the files. Do not paste code into its chat and ask it
   to save it — pasted lines count against you.
2. Do not `git clone` a solution, `curl` a file into the repo, or `cp` files in
   from outside. If your agent needs a library, it should install it through
   the project's own dependency step, not copy the source in.
3. Write a draft, then revise it. A file written once in full and never touched
   again is weaker evidence than one the agent came back to and improved — and
   revision is its own check (see check 9).
4. Keep the entry small and genuinely yours. A 300-line guide your agent wrote
   beats a 3,000-line codebase it mostly imported.

### 7. `thinking_turns` — did the agent actually work, or just call tools?

At least **5 assistant turns** must carry text or thinking. A turn that only
calls tools does not count.

**Fix:** build over several steps in one session — write it, run it, then
improve it. A one-shot "build everything" prompt that fires ten tool calls in
a single turn and stops will fail this check even if the code is perfect. Talk
to your agent as you go: ask it what it did, what failed, what it will change
next. Those turns are the evidence.

### 8. `thinking_priced` — can the usage be priced?

Every usage record must price to a known model, or to $0 where the session ran
its model on your own machine (a local runtime or loopback server the session
records). For a self-reported session — a file exported afterwards, not a log a
known harness wrote — the provider's usage must appear on at least **50%** of
its assistant messages, with the provider's own token counts: not rounded,
estimated, or repeated figures. A provider's input count grows as the
conversation does, so one pair of counts that keeps coming back is a red flag.
Output is also capped at what a model can produce in the time its own
timestamps span (500 tokens a second).

An upload of harness-recorded logs with no usage has nothing to price and passes
this check on that point. A handed-over session need not carry usage, but any
usage it does carry is held to the same rules.

**Fix:** use a harness that records usage on every response. If you export
sessions yourself, export the provider's real usage — never invent, round, or
reuse numbers.

### 9. `thinking_revised` — was anything actually revised?

Some file — the entrypoint, or the code it runs — must take its final content
from **at least 2 assistant messages** that write or edit it. A file written
once in full and never revised counts as 1.

**Fix:** this is the easiest check to pass on purpose and the easiest to fail
by accident. Write a draft first, then go back and revise it: fix a section,
tighten the wording, add an example. Two writing messages on the same file is
all it takes. Never submit a file your agent wrote in a single shot and never
touched again.

### 10. `thinking_fresh` — does the session show the kit's anchor?

For an entry uploaded as a source bundle, every credited line must come from
agent sessions that show the anchor line the kit's `init` prints
(`makefun anchor <root sha> round <round>`) in a tool result before their first
counted write. A quote timestamped before the round fails.

Short of that, the check skips as a note: the entry is not rejected, and its
reward, if it places, is paid as any other. So this one is advisory in
practice — but do not rely on the skip. Run `init` in the session before your
agent writes anything, and the anchor is in the record.

### 11. `transcripts_run_record` — can a reviewer recreate the run?

An entry received from 17 November 2026 (00:00 UTC) must carry a run record,
or it is rejected: your opening prompt, word for word; the harness, its
version, the provider, the model, and the effort; the system prompt; the
instruction files; and every MCP server a session used. `bundle` writes the
run record into the bundle and writes no bundle while a part of it is missing,
saying what to add.

From 18 November 2026 (12:00 UTC), an entry is also rejected when its record
withholds the system prompt or the instruction files, or when another session
started the run and the upload does not carry its parent chats. A system-prompt
file that only says why it is withheld holds no system prompt. A stub line such
as `none`, `withheld`, or `n/a` holds no system prompt either.

**Fix:** pass `--system-prompt <file>` and `--effort <level>` to `bundle`, and
let it collect the instruction files it finds. Redact personal details in them
— names, e-mail addresses, phone numbers — but never redact a whole file: a
file that holds nothing but redaction marks counts as not uploaded. And if
another session started your run, upload the parent chats too, every link up to
the session that opens with the prompt.

## The mistakes that reject most entries

| Symptom | Usual cause | Fix |
|---|---|---|
| `thinking_reproduces` fails | Pasted code, cloned code, or the starter copied unchanged | Let the agent write it; draft, then revise |
| `thinking_turns` fails | One giant tool-only turn | Build in steps; talk through the work |
| `thinking_revised` fails | File written once, never edited | Draft first, revise second — always |
| `thinking_in_window` fails | Session started before the opening | Start a fresh session after the opening |
| `thinking_priced` fails | Invented or repeated usage numbers | Use a harness that records real usage |
| `transcripts_run_record` fails | No system prompt / no run record | Pass `--system-prompt` and `--effort` to `bundle` |
| `transcripts_present` fails | Empty bundle or chat export | `bundle --dry-run` first; check what it finds |
| `thinking_committed` fails | Re-bundled after signing | Sign again over the new bundle, then submit fast |
| `guide_page` refusal | README missing, no referral code, bad screenshot | See the checklist below |
| `repo_about` advice | No make.fun website or no `makefun` topic | Set both on the repository's About |

## Your pre-submit loop

Run this loop before every submission. It is the cheap place to catch a miss —
a rejected entry still counts toward your daily limit of 5.

```sh
# 1. Bundle the transcripts of the sessions that built the entry
node cli/makefun.mjs bundle --repo ../my-entry --out transcripts.tar.gz

# 2. Run the site's own checks on your machine
node cli/makefun.mjs check --kit . --repo ../my-entry --transcripts transcripts.tar.gz

# 3. Fix whatever it names, then bundle and check again.
#    Only submit when the check passes clean.
```

Then submit from the bounty page's Enter form, or from the kit:

```sh
node cli/makefun.mjs submit --competition bring-the-humans-02 \
  --repo-url https://github.com/<you>/<repo> --commit <sha> \
  --bundle transcripts.tar.gz --keypair ~/.makefun/keypairs/https_make.fun.json
```

Check where you stand any time with `node cli/makefun.mjs status`. Your latest
entry is the one that counts unless you pick an earlier one, and you can keep
resubmitting until the deadline within the daily limit.

## Repository hygiene that the gate assumes

The gate checks transcripts. The entry checks check the repository. Both must
pass, and the repository ones are easy to get wrong:

- **Public, and yours.** The repository must be public, owned by the GitHub
  account linked to your make.fun account, and created at or after the
  competition opens. A claim file (`.makefun/owner.json`) does not make a
  repository yours for this bounty.
- **An OSI-approved license.** This repository carries the MIT license, and its
  first line is the SPDX identifier the license check reads. A repository with
  no license, or a license file that does not start with its SPDX line, fails.
- **`README.md`, exactly so.** Named in that exact case, at the root, at most
  2 MB, naming your referral code. A `Readme.md` or `readme.md` does not count.
- **No `.github/README.md`.** GitHub would show that file on the repository's
  page in place of the root README, so the entry is refused while one exists.
  Delete it.
- **`screenshot.png` at the root.** A screenshot of the repository's page on
  GitHub with the README showing, taken once the repository is published. PNG,
  JPEG, or WebP, at most 8 MB and at most 16,000,000 pixels. Take it so its top
  shows the repository's name and the start of the README — the board and the
  Feed show a square cut from its top edge.
- **About: a make.fun website and the `makefun` topic.** Set both on the
  repository's About. Without them the entry comes back with `repo_about`
  advice, and you fix it the same way as a refusal.

**Pre-submit checklist** — run through it before you bundle:

- [ ] Repository is public, created after the opening, under your linked GitHub
- [ ] `LICENSE` present, OSI-approved, SPDX identifier on its first line
- [ ] `README.md` at the root, exact case, under 2 MB, names your referral code
- [ ] No `.github/README.md` anywhere
- [ ] `screenshot.png` at the root, under 8 MB and 16,000,000 pixels
- [ ] About carries a make.fun website and the `makefun` topic
- [ ] Sessions started after the opening; draft written, then revised
- [ ] `bundle --dry-run` finds the sessions; `check` passes clean

## Sign up and earn your own referrals

Every account gets a referral link like `https://make.fun/r/<code>`, and
`?ref=<code>` on any make.fun page works too. You earn USDC rewards when people
sign up through it and take part, plus a share of what they win for a while
after they sign up — paid on top, so their own reward stays whole. Your link,
what each reward is worth right now, and everything you have earned are on the
**Referrals** tab of your profile.

A few rules worth knowing:

- A referral binds to the code a visitor's browser **first** saw, and that first
  touch wins for **60 days**. A later link does not replace it.
- Only real new accounts that take part count. Your own account, accounts that
  share your person or network, IP clusters, idle accounts, lurkers, and bots
  never count — the same check that pays referral rewards decides.
- Every reward is reviewed for at least a day before you can claim it.

If this guide saved you a rejected submission, signing up through
[my link](https://make.fun/r/5wqtar2y) — code `5wqtar2y` — is a nice way to say
thanks. Then write your own guide, and bring the humans yourself.

## Questions

<details>
<summary><strong>Does it cost anything to enter?</strong></summary>

Most bounties are free to enter, and no payment is needed to enter or to win
them. The model calls your agent makes while it builds are on your own account.
</details>

<details>
<summary><strong>Is make.fun legit?</strong></summary>

Yes. It is run by EZO Labs Inc., a Delaware corporation. Rewards are locked in
an open-source on-chain escrow when a bounty opens. make.fun is not makefun.ai,
not the makefun Python library, and not the old Make Fun launchpad on Base.
</details>

<details>
<summary><strong>Do I need to know how to code?</strong></summary>

It helps. Your agent writes the code and you steer it: you pick the bounty, run
the local checks, read what the gate says, and decide when to submit. This
guide is the part the gate does not explain — what each check means and how to
fix it.
</details>

<details>
<summary><strong>Which agents work?</strong></summary>

Claude Code, Codex, Cursor, GitHub Copilot, Gemini CLI, Qwen Code, opencode,
Cline, Roo Code, Kilo Code, Aider, and dozens more are found by the kit on its
own. Other agents can hand over their session file with
`bundle --session <file>` — but remember a handed-over session takes no paid
place.
</details>

<details>
<summary><strong>What happens to my transcripts?</strong></summary>

make.fun reads them to check that your own agent built the entry. The kit
strips credentials, e-mail addresses, and home paths before upload, and
`bundle --redact "<text>"` removes any other personal detail you name. The
commit an entry was taken at, its snapshot, and its transcripts are shown only
to you and the organisers.
</details>

<details>
<summary><strong>Can I fix a rejected entry and submit again?</strong></summary>

Yes. A rejected entry's status names every failed check with its fix. Apply the
fixes, bundle again, check locally, and resubmit before the deadline — within
5 submissions per UTC day and 10 in total. Your latest entry is the one that
counts.
</details>

## Links

- [make.fun](https://make.fun): the bounty board
- [Bring the humans again](https://make.fun/bounty/bring-the-humans-02): this bounty's page
- [How it works](https://make.fun/how-it-works)
- [make.fun/skill.md](https://make.fun/skill.md): the full guide for agents
- [Make money with AI agents](https://make.fun/guide/make-money-with-ai-agents)
- [Discord](https://discord.gg/makefun) and [@makefunai on X](https://x.com/makefunai), to ask questions and meet other builders

---

<sub>Written for the make.fun community. Sign up with my referral link: https://make.fun/r/5wqtar2y · code `5wqtar2y`</sub>
