# VPS for AI Agents: Do You Need a GPU, How Much RAM Claude Code and Codex Really Need, and Which Plan Should You Buy? (Spec Table + Unattended Setup)

The model stays on Anthropic's or OpenAI's servers, so the server you rent has no GPU in it. What you're paying for is CPU, RAM and fast disk for the CLI, the git checkouts, the builds, the test runs and Docker. The floor is **4 GB of RAM** for one Claude Code or Codex session, and **8 GB** once you want Codex at OpenAI's recommended size, two or three agents in parallel, or OpenClaw.

What follows covers what a RackNerd KVM VPS special actually includes, the RAM thresholds published in RackNerd's own one-click catalog, the subscription limits a new server does not remove, and the steps from checkout to an agent running unattended in tmux.

> **Quick pick.** One agent on a small or medium repo: a RackNerd special with **4 GB RAM** and at least 2 vCPU. OpenClaw, or two to three agents alongside Docker builds: **8 GB**. A team sharing one box: 16 GB. Every special is unmanaged KVM with full root SSH, a 1 Gbps port and the ctrl panel, which installs Hermes Agent or OpenClaw in one click.
> 👉 [Check the RAM and price on each current RackNerd VPS special](https://bit.ly/RacKnerd)

## What RackNerd sells, and why there's no permanent price list to copy

RackNerd sells its VPS line as promotional "specials" rather than a fixed monthly rate card. The batch live on the specials page changes, and new batches land around shopping holidays: an 11.11 sale in mid-November, Black Friday and Cyber Monday at the end of November, 12.12, Christmas, New Year, plus short flash sales that can sell out inside an hour. A grid of plan names and prices copied into an article goes stale within weeks; the specs you're actually choosing between do not.

Those specs stay consistent across the lineup:

| What every RackNerd VPS special includes | Detail |
| --- | --- |
| Virtualization | KVM, full root SSH access |
| Network | 1 Gbps port |
| Operating systems | CentOS, Rocky Linux, AlmaLinux, Fedora, Debian, Ubuntu; custom ISO installs on request via support ticket |
| Control panel (ctrl) | Reboot, OS reinstall, out-of-band HTML5 VNC console, one-click apps, inbound firewall rules, root password reset, upgrades |
| One-click apps | Hermes Agent, OpenClaw, AnythingLLM, Ollama, LibreChat, n8n, Coolify, Docker with Portainer |
| Management | Unmanaged. 24/7 support covers the platform, not your stack |
| Provisioning | Instant |

Two things decide your bill and your experience more than the headline price. The billing term shown at checkout, since a promotional annual figure and a monthly figure are not the same number once you divide them out. And the renewal rate: RackNerd's own FAQ carries a question about whether a special renews at the same rate, so read that answer before you commit to a multi-year term rather than assuming the first-term price carries forward.

## How much RAM each agent actually needs

Two independent sources land on the same numbers, which is why I'd treat them as the real thresholds rather than marketing noise.

| Workload | Minimum RAM | Where the number comes from |
| --- | --- | --- |
| n8n, Coolify | 2 GB | RackNerd's one-click catalog |
| One Claude Code session | 4 GB | Anthropic's listed minimum |
| One Codex session | 4 GB | OpenAI's listed minimum |
| Hermes Agent, AnythingLLM, Ollama, LibreChat | 4 GB | RackNerd's one-click catalog |
| Codex at OpenAI's recommended size, 2–3 agents, Docker builds | 8 GB | OpenAI's recommendation |
| OpenClaw | 8 GB | RackNerd's one-click catalog |
| Four or more agents, or a small team on one box | 16 GB | Hosting vendors' sizing guidance for parallel agents |

The 4 GB line is where Claude Code, Codex and every self-hosted agent image RackNerd ships agree. The 8 GB line shows up twice for different reasons: OpenAI recommends it for Codex, and RackNerd requires it for OpenClaw. If your workload appears in both columns, buy the 8 GB tier.

vCPU matters less than RAM for an idle agent and more than RAM during a build. A single session plus a language server is comfortable on 2 cores. Three agents each running `npm test` in their own git worktree will queue on 2 cores and feel slow, which is the point at which you move up a tier rather than adding RAM alone.

## Sizing rules: three tiers, one decision

- **One agent, small or medium repository, occasional builds.** 4 GB and 2 vCPU. This is the majority case, and buying more RAM will not make the agent smarter or faster.
- **Two or three agents, or Codex at its recommended size, or Docker in the loop.** 8 GB. The extra memory covers the language servers, the test suite and the build cache that accumulate around each session.
- **Four or more agents, or a team sharing one machine.** 16 GB. Past this point you're buying headroom for parallel git worktrees and heavier test suites, not for the model.

👉 [See which RackNerd special clears the 8 GB line](https://bit.ly/RacKnerd)

Pick by the RAM column first and the price second. A cheap plan that sits at 2 GB will run n8n fine and will choke the moment you point an agent at a real repository.

## What the VPS invoice doesn't cover

The server bill and the AI bill are two separate payments, and the second one is usually the larger.

**Claude Code needs a paid plan.** Claude Pro, Max, Team, Enterprise or a Console account all work. The free Claude plan does not.

**Codex CLI needs ChatGPT Plus, Pro, Business or Enterprise, or an OpenAI API key.**

**Usage caps follow you onto the server.** Claude Pro and Max carry a five-hour session limit and a weekly limit; ChatGPT's tiers have their own. Nothing about renting a machine in a data center lifts those. When you hit the cap, the agent stops mid-task and sits there until the window resets or you log in again. Overnight runs that finish at 3 a.m. can stall at 3:40 a.m. and stay stalled.

**Scheduled work should use an API key, not a subscription login.** Subscription sessions expire and need a device-code round trip to sign back in. A cron job cannot do that at 4 a.m. Point cron entries and long-running automation at an API key and leave the interactive subscription login for the sessions you're watching.

**Token costs don't disappear.** Prompts go to the provider and are billed there, per token or against your plan's allowance. Some hosts advertise "no per-token charges," which is true only when the model itself runs on your box, a different setup with different RAM requirements.

👉 [Pick a special with the RAM and billing term you settled on](https://bit.ly/RacKnerd)

## From checkout to a tmux session that outlives your laptop

1. **Choose the image.** Ubuntu 24.04 LTS or Debian. Pick Debian if you want to skip one configuration step later (see item 6).
2. **Deploy and log in.** The VPS provisions instantly; you get an IP and a root password, and ctrl can reset that password if you lose it. Create a normal user before you install anything.
3. **Install the agent.** For Hermes Agent, OpenClaw, AnythingLLM, Ollama, LibreChat or n8n, open One-Click Apps in ctrl, select the app and wait; the credentials appear in the panel when it's ready. For Claude Code or Codex CLI, install over SSH.
4. **Sign in without a browser on the server.** Claude Code prints a login URL in the terminal; open it on your phone or laptop, sign in, and paste the returned code back into the SSH session. For Codex, enable device code login in your ChatGPT security settings first, then run `codex login --device-auth`.
5. **Work inside tmux.** `tmux new -s agent` to start, `Ctrl-b` then `d` to detach, `tmux attach -t agent` to come back. Closing your laptop no longer ends the run.
6. **On Ubuntu 24.04 and later, configure AppArmor for the sandbox.** Claude Code's sandbox needs a bubblewrap AppArmor profile on those releases; Anthropic's sandboxing documentation has the profile steps. Debian images avoid this.
7. **Optional: run OpenClaw as a service.** `openclaw onboard --install-daemon` registers it with systemd so it restarts on reboot.
8. **Check in remotely.** Claude Code's Remote Control makes outbound connections only, so there are no extra ports to open and nothing to expose to the internet. It requires a Pro, Max, Team or Enterprise plan.

👉 [Start with the entry special and get Claude Code into tmux tonight](https://bit.ly/RacKnerd)

## Leaving it unattended: six habits

Running an agent without permission prompts on the laptop that holds your SSH keys, your browser sessions and your customer files is the actual risk. On a dedicated VPS, the worst case is one machine you reinstall.

- **Run the agent as a normal user.** A separate account without passwordless sudo means a bad command can't rewrite the system.
- **Use SSH keys and turn off password authentication.** An always-on box with a public IP gets scanned continuously.
- **Give the server its own credentials.** A deploy key or a fine-grained token scoped to the repositories the agent needs, never your personal SSH key.
- **Keep production secrets off the machine.** Anything the agent can read, it can leak, including environment files it stumbles into during a refactor.
- **Close every port you don't need.** SSH in, and open others only for services you intend to expose. ctrl's firewall handles inbound rules.
- **Have a way back.** Push code to a remote and keep config in a repo, because a reinstall from ctrl gives you a clean server, not your files. Check whether backups are included on the plan you pick; on several hosts they're a paid add-on.

## What the same money buys from other hosts

Worth knowing before you decide the RackNerd price is either good or suspiciously low. All figures below are as published; currencies differ, and the euro prices include VAT.

- **Space-Node** lists a 4 GB plan at €4.99/month and an 8 GB plan at €8.99/month in EU datacenters. The vCores are shared even though RAM is dedicated, backups are not included, there is no one-click installer for any of these agents, and Standard-tier orders are provisioned manually within 24 hours. Support runs 08:00–20:00 UTC.
- **CometVPS** prices 4 GB at $14/month and 8 GB at $32/month across six US locations, with backups as a paid add-on and a 99.9% SLA.
- **IONOS** starts at $2/month and scales to 12 vCores and 24 GB, installs Hermes Agent during the order, includes a 30-day money-back guarantee, and does not support downgrading to a smaller plan later.
- **Bluehost** offers one-click Hermes Agent deployment on self-managed VPS, but its terms prohibit sustained use of more than 25% of system resources for over 90 seconds. Its NVMe 2 plan holds an A grade for performance stability from VPSBenchmarks, whose evaluation page is dated February 2026.
- **Contabo** installs Hermes as a one-click add-on and leaves inbound traffic unmetered, with outbound subject to fair-use throttling. Add-on support covers the initial installation; keeping the open-source software patched is your job.

The pattern is consistent: cheap plans cut somewhere, whether that's shared vCores, missing backups, no one-click installs, manual provisioning windows or resource clauses buried in the terms. RackNerd's trade-off is that the server is unmanaged and the plan lineup rotates. In exchange you get root, a 1 Gbps port, the ctrl panel with its one-click catalog, and a price that changes with the sales calendar.

## FAQ

**Do I need a GPU?**
No, for Claude Code, Codex CLI, OpenClaw or Hermes Agent as normally deployed. The model runs at the provider. A GPU becomes relevant only if you plan to run local inference on the same machine, which is a different product decision.

**Is 4 GB really enough?**
For one session on a small or medium repository, yes, and it's the figure Anthropic and OpenAI both publish as the minimum. It stops being enough when you add a second agent, run Codex at its recommended size, or expect Docker builds alongside the session.

**Can I run this on the free Claude plan?**
No. Claude Code requires Claude Pro, Max, Team, Enterprise or a Console account. Codex CLI requires a paid ChatGPT tier or an OpenAI API key.

**Will the agent keep working after I hit my usage limit?**
No. Claude Pro and Max cap usage per five-hour window and per week, and ChatGPT's plans have their own limits. The process stays alive but stops making progress until the window resets.

**Is Hermes Agent the same thing as the Hermes model?**
No. Hermes is an open-weights model family from Nous Research, with Hermes 3 as the current line and Hermes 4 rolling out alongside it. Hermes Agent is a separate product that remembers past sessions, schedules tasks, delegates to subagents and uses tools to browse files and run code. RackNerd's ctrl installs the Agent, and it needs 4 GB or more; the model you point it at is your choice and your API bill.

## Which one to order

Match the plan to the workload, not to the discount.

One Claude Code or Codex session on a normal repository takes the smallest special that clears **4 GB with 2 vCPU**. Two or three agents, Docker in the build loop, or OpenClaw takes **8 GB**. A team or an agent fleet takes **16 GB**.

Two checks before you pay. Read RackNerd's renewal answer on the plan card, so you know what the second term costs rather than only the first. Then pick the datacenter closest to you, because SSH from a phone across an ocean is the part of "check in from anywhere" that articles leave out.

👉 [Compare RackNerd's current VPS specials and their RAM tiers](https://bit.ly/RacKnerd)
