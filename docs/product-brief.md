# Todo-Together: product brief

## Purpose

A shared todo app for a two-person household. The problem it solves is visibility: each partner carries a load the other can't see, so they can't help with it and can't appreciate it. The app lets each person see what is on the other's plate, take things off it, and see what the other has done.

## Principles

- **Visibility over tracking.** The todo list is the mechanism; seeing each other's load and effort is the goal.
- **No scoreboard.** Effort is shown per task and in the history. It is never totaled per person or compared side by side. Joint totals ("together you finished 12 things") are fine.
- **Consent over assignment.** Nobody can put a task directly on the other person's plate.
- **Few notifications.** If notifications become a burden they get turned off, which defeats the app.
- **Fast capture.** Adding a task must take a few seconds. Fields are optional unless the household decides otherwise.

## Core (first version)

### Tasks

- Every task is owned by one person or is **unclaimed**.
- Status: To do, In progress, Done. Shown as a board.
- Optional fields: due date, importance, effort size (small / medium / large), notes, sub-steps.
- Effort size is an estimate. Whoever does the task can correct it.
- Household settings decide which optional fields are required.

### Taking tasks

| Task state | What happens |
|---|---|
| Unclaimed | Taken freely. Reported in the next batched notification. |
| Owned by the other person, not started | Taken freely. The owner is notified immediately. |
| Owned by the other person, in progress | The taker is warned first. The owner is notified immediately if they go ahead. |

### Requests

- To give someone a task, you send a request. They accept or decline.
- Declining is one tap, needs no explanation, and leaves the task where it was.
- A pending request does not move the task.

### History

- A feed of who created, took, and finished what, with takeovers highlighted and effort size shown.
- Each person's current load is visible on the board (what they own, with sizes).
- No per-person totals or comparisons.

### Notifications

- **Immediate:** things that change your own plate (a request sent to you, a task of yours taken).
- **Batched:** everything else (new tasks, edits, claims of unclaimed tasks, completions). One summary is sent after the other person stops editing for a few minutes. Claims lead the summary. A single change is described specifically ("Connor took 'Shop for car tires'"); many changes are summarized ("Connor changed 15 tasks").
- A batch is skipped if the recipient has already seen the changes in the app.
- Every notification points at something recorded in the history, so a missed push delays information and never loses it.

## Backlog, in priority order

1. **Routines (recurring tasks).** A separate page that never mixes with the board. Shows only what is due today or this week. Completions go to the history but never get their own push. A per-routine option can create a normal todo on the board when an infrequent routine comes due.
2. **Private tasks.** Tasks only the owner can see (gifts, surprises).
3. **Schedule and capacity.** Not part of the core app. To be built as a separate companion against the server's API, not as an in-app plugin.
4. **Open source and self-hosting.** Sign-up, invites, multi-household administration, hosting documentation.

## Decisions that keep the backlog possible

- **Platform: installable web app (PWA).** She has an iPhone, he has an Android, and there is no Mac. A PWA needs no Mac, no App Store, and no Apple developer account, which also keeps it free for anyone who self-hosts later.
- **Real user accounts and a household concept from day one.** All data is scoped to a household, even though there is only one.
- **Clean server API.** Lets a companion app or a native client be added later without changing the back end.
- **Configuration is not tied to one deployment.**

## Build order

The order is chosen to reach a version both people can use as early as possible. Work is tracked as GitHub issues grouped into these milestones.

1. **Foundation.** Database, accounts, household, sign-in.
2. **Tasks and board.** Create, edit, claim, and complete tasks; see each other's plates.
3. **Hosting and first install.** Docker, the laptop server, a public address, backups, and the app on both phones. This is the first usable version.
4. **Taking, requests and task options.** Taking the other person's tasks, requests, sub-steps, required-field settings.
5. **History.** The activity feed.
6. **Notifications and live updates.** Push, batching, and the board updating live.
7. **Backlog.** The items listed above.

## Tech stack

| Part | Choice |
|---|---|
| Runtime | .NET 10 |
| Server | ASP.NET Core Web API with EF Core |
| Client | Blazor WebAssembly, installed as a PWA |
| Live updates | SignalR |
| Push | Standard Web Push (VAPID keys) |
| Sign-in | ASP.NET Core Identity with cookies |
| Database | SQLite, stored outside the container so it survives updates |
| Packaging | One Docker image containing the server and the client files |
| Hosting | A spare laptop at home running Debian |

Hosting details still to settle:

- A public HTTPS address. Proposed: a Cloudflare Tunnel with our own domain.
- Scheduled backups of the SQLite file to somewhere off the laptop.
- Debian configured not to suspend when the lid closes.

## Known risks

### Push on iPhone

Push notifications for web apps on iPhone are less dependable than native ones. This is tested at the start of the notifications milestone, by which point the app is already installed on her phone.

It is not tested earlier because the result would not change what gets built first. The PWA decision stands for other reasons (no Mac, free self-hosting), and everything before notifications works without push.

### Launch time

Blazor WebAssembly starts more slowly than a JavaScript front end: a first load of a few megabytes, then a cold start while the runtime boots. This works against fast capture.

Decision: accept it and do not optimize up front. Both phones are recent, and staying in C# matters more. Cold start will be observed once the first version is installed on both phones.

If it turns out to be a problem, the options are, in order:

1. A plain HTML quick-add box that appears before Blazor loads and queues what is typed.
2. Lazy-loading rarely used pages (settings, history).
3. Keeping the client free of heavy component libraries.

ReadyToRun does not apply to WebAssembly, and AOT compilation is not a startup fix (it roughly doubles the download).
