`ez_mcp` is a single-file, zero-dependency developer toolbox that operates simultaneously as a Model Context Protocol (MCP) server for AI agents (such as Claude Code and Cursor) and as an interactive CLI for environment maintenance and scaffolding.

### Key Architectural Highlights

* **Zero External Dependencies:** Written as a Bash launcher with embedded Python (3.9+ standard library only). No `pip` installs or external packages are required.
* **Single-File Multi-Call Binary:** Symlinking or copying `ez_mcp` to filenames like `ez_doctor`, `ez_scan`, `ez_status`, `ez_scaffold`, or `ez_serve` allows each command to run independently as a standalone binary without code duplication or version drift.
* **Read-Only Safety Model:** MCP calls are strictly confined to authorized root directories (`~/github_projects` and `~/ez_mcp` by default, or customized via `EZ_MCP_ROOTS` / `--root`). Destructive operations or project code execution require launching the server with explicit write access using `--allow-write`.

---

### Command & Tool Capability Matrix

| Command / MCP Tool | Primary Function | MCP Safety Gate |
| --- | --- | --- |
| `doctor` | Checks live DNS, `/etc/resolv.conf`, VPN tunnels, file permissions, and tool states. | Read-only |
| `status` | Summarizes Git repository states (dirty files, ahead/behind branch offsets) across project directories. | Read-only |
| `scan` / `scan_text` | Analyzes text or files for prompt injection vectors, hidden Unicode, dangerous shell calls, and leaked secrets. | Read-only (masks secrets) |
| `scaffold` | Runs a guided wizard to generate DevContainer + Docker Compose + Postgres + MCP server starter kits. | Requires `--allow-write` (forces dry-run otherwise) |
| `fix-dns` / `dns_repair_plan` | Generates or executes automated repair plans for broken or dangling DNS configurations. | Read-only plan (execution via CLI `sudo`) |
| `check` / `run_checks` | Runs test and lint suites (`pytest`, `unittest`, `ruff`) in target project trees. | Requires `--allow-write` |
| `serve` | Launches the stdio MCP server for agent integration. | Configurable via flags |
| `selftest` | Executes end-to-end MCP protocol handshakes and tool gating verification in-process. | Internal testing |

---

### Custom Tool Manifests

You can expose custom shell or Python scripts as both CLI commands and typed MCP tools without modifying `ez_mcp` source code. Defining a JSON schema in `~/.config/ez_mcp/tools.json` provides input validation, environment scrubbing, and execution timeouts automatically:

```json
{
  "tools": [
    {
      "name": "deploy_staging",
      "description": "Trigger a staging deployment script",
      "command": ["/usr/local/bin/deploy.sh"],
      "enabled": true,
      "mutates": true,
      "properties": {
        "target_env": { "type": "string", "enum": ["staging-a", "staging-b"] }
      },
      "required": ["target_env"]
    }
  ]
}

```
## Overview of `ez_mcp` Architecture & Build Mechanics

`ez_mcp` is designed as a zero-dependency, single-file polyglot developer utility and Model Context Protocol (MCP) server.

### Dual-Interface Architecture

* **CLI & Stdio MCP Server:** Operates as both an interactive CLI / terminal utility for developers and a stdio MCP server for AI coding agents like Claude Code and Cursor.
* **Polyglot Execution:** Uses a Bash launcher that feeds an embedded Python 3.9+ payload to standard input via file descriptor 3 (`python3 /dev/fd/3 "$@" 3<<'PYEOF'`). This leaves `stdin` open for interactive terminal prompts or JSON-RPC streams.
* **Symlink Multi-Call Routing:** Employs BusyBox-style command aliasing. Symlinks named after specific subcommands (e.g., `ez_doctor`, `ez_scan`, `ez_status`, `ez_serve`, `ez_scaffold`, `ez_fix_dns`) inspect `$(basename "$0")` at runtime to dispatch to the appropriate tool directly.

### Safety Model & Gating

* **Root Confinement:** Restricts file access for AI callers (`origin == "mcp"`) to configured directory roots (defaults to `~/github_projects` and `~/ez_mcp`, customizable via `EZ_MCP_ROOTS` or `--root`).
* **Read-Only Default:** Mutation tools (such as project scaffolding or running test suites) require starting the server with `--allow-write`. Dry-run capable tools automatically revert to `dry_run=True` if write permissions are absent.
* **Isolated Command Execution:** External processes run without shell invocation, using scrubbed environments (`SAFE_ENV_KEYS`) and strict timeouts.

---

## Analysis of Git Log Errors & Build Resolution

The terminal output reflects a sequence of initial setup commands and their resolution:

1. **Initial Git Revision Failures:**
* Running `git log -2 --oneline` and `git diff HEAD^ HEAD` produced `fatal: your current branch 'main' does not have any commits yet` and `fatal: ambiguous argument 'HEAD'`.
* **Cause:** These commands were executed on an initialized but empty repository prior to creating a root commit.


2. **Build Assembly (`build_ez_mcp.sh`):**
* The script concatenated core Python modules (`part1_core.py` through `part5_friendly.py` and `scaffold_old.py`) into the polyglot shell wrapper `ez_mcp`.
* Created symbolic links routing specific commands (`ez_doctor`, `ez_scan`, `ez_status`, `ez_scaffold`, `ez_serve`, `ez_fix_dns`) back to the main executable.


3. **Commit Completion:**
* Executing `git add .` and `git commit -m "feat: atomic implementation of ez_mcp polyglot toolbox"` successfully established the initial repository commit (`ff6afe4`), resolving the working tree revision errors.



---

## Open-Source Maintenance & System Handover Strategy

To maintain `ez_mcp` effectively as an active open-source project and ensure smooth maintenance transitions:

### Automated Maintenance & Stale Management

* **Automated Staleness Policy:** Implementing workflows like `actions/stale` scans inactive issues and PRs on a schedule, applying a `stale` label before auto-closing.


* **Differentiated Timers:** Issues and pull requests often require distinct inactivity thresholds (e.g., 360 days before marking an issue stale vs. 180 days for a PR).


* **Label Exemptions:** High-signal issues or PRs awaiting maintainer input should be exempted using labels such as `needs-triage` or `needs-review`.


* **Triage & Burnout Prevention:** Establishing regular, timed triage sessions (e.g., 15 minutes daily) and using issue templates streamline incoming maintenance requests.



### Documentation & Knowledge Transfer

* **Software Handover Protocol:** Software transitions require transferring system knowledge, documentation repositories, and code bases systematically.


* **Repository Documentation:** Maintaining comprehensive `CONTRIBUTING.md` files, pull request checklists, and architecture guides prevents knowledge loss during maintainer handovers.


* **Branch Strategy:** Maintaining clear branching structures (such as `main` for master releases, feature branches for development, and maintenance branches for patch releases) provides operational stability during software transfers.ez_mcp
Custom Gem
Execution Summary & Current Repository State
HANDOVER.json was generated and committed to your Git repository under the commit message "docs: generate extensive conversation-based handover JSON for dev team".

Key Technical Context Captured in HANDOVER.json
File Descriptor 3 (/dev/fd/3) Execution: Passes the Python payload via FD 3 to keep logic in memory without writing temporary files, leaving standard input (STDIN / FD 0) completely open for MCP JSON-RPC protocol communication.

Root Confinement & Security: Enforces path isolation when origin == "mcp", restricting tool access strictly to allowed directory roots (e.g., ~/github_projects) and blocking path traversal attempts (../).

BusyBox-Style Multi-Call Binary: Routes symlinked commands (ez_doctor, ez_scan, ez_scaffold, ez_serve) through a single executable by inspecting $(basename $0).

Crostini / Container Constraints: Documents file permission boundaries when accessing ChromeOS mounts (/mnt/chromeos) from within the Linux container environment.

Development Workflow: Establishes the core maintenance loop: edit source modules in parts/, concatenate via build_ez_mcp.sh, and validate via ./ez_mcp selftest.

Terminal Status & Verification
The terminal errors (zsh: command not found, zsh: no matches found) occurred prior to the heredoc execution because unquoted Markdown text was pasted directly into the interactive Zsh prompt.

To clean your shell session and verify the repository state:

Bash
# Reset shell prompt state
clear

# Inspect the latest commit containing HANDOVER.json
git log -1 -p HANDOVER.json

# Run the built-in end-to-end self test
./ez_mcp selftest
Would you like to validate HANDOVER.json against a custom schema or run additional build verification checks on ez_mcp?

-rw-r--r-- 1 tangleroot013 tangleroot013     451 Sep  8 19:52  pytest.ini

-rw-r--r-- 1 tangleroot013 tangleroot013    3883 Sep  4 22:01  README.md

drwx------ 1 tangleroot013 tangleroot013      72 Sep  5 23:47  recovery-backup

drwxr-xr-x 1 tangleroot013 tangleroot013       0 Sep 11 06:40  .repair-backups

drwxr-xr-x 1 tangleroot013 tangleroot013     114 Sep  8 07:29  repo-safety-backups

drwxr-xr-x 1 tangleroot013 tangleroot013      64 Sep  8 21:02  .rest-client

drwxr-xr-x 1 tangleroot013 tangleroot013     226 Jul  5 09:24  .runelite

drwxr-xr-x 1 tangleroot013 tangleroot013    1242 Sep 11 06:40  scripts

-rw-r--r-- 1 tangleroot013 tangleroot013     361 Sep  4 20:48  scripts_dns_leak_test_sh.bash

-rw-r--r-- 1 tangleroot013 tangleroot013     727 Sep  4 20:48  scripts_handshake_freshness_test_sh.bash

-rw-r--r-- 1 tangleroot013 tangleroot013     398 Sep  4 20:48  scripts_performance_monitor_sh.bash

-rw-r--r-- 1 tangleroot013 tangleroot013    1526 Sep  4 20:48  scripts_recovery_daemon_sh.bash

-rw-r--r-- 1 tangleroot013 tangleroot013    1039 Sep  4 20:48  scripts_server_discovery_sh.bash

-rw-r--r-- 1 tangleroot013 tangleroot013    1031 Sep  4 20:48  scripts_test_connection_sh.bash

-rw-r--r-- 1 tangleroot013 tangleroot013      66 Sep  8 11:19  .selected_editor

drwxr-xr-x 1 tangleroot013 tangleroot013      34 Jul 12 20:24  session_scripts_archive

-rw-r--r-- 1 tangleroot013 tangleroot013     816 Sep  4 20:48  setup_orchestrator.bash

drwxr-xr-x 1 tangleroot013 tangleroot013      66 Sep  5 22:58  shell_backups

drwxr-xr-x 1 tangleroot013 tangleroot013      62 Aug  4 10:49  .sonarlint

lrwxrwxrwx 1 tangleroot013 tangleroot013      34 Aug  6 01:26  .ssh -> /home/tangleroot013/home-data/.ssh

-rw-r--r-- 1 tangleroot013 tangleroot013       0 Jul  5 08:20  .sudo_as_admin_successful

-rw------- 1 tangleroot013 tangleroot013   16384 Sep  6 09:50  .swp

-rw-r--r-- 1 tangleroot013 tangleroot013    1598 Sep  9 20:02  test_ez_grav.py

drwxr-xr-x 1 tangleroot013 tangleroot013     142 Jul  9 06:12  tests

drwxr-xr-x 1 tangleroot013 tangleroot013      14 Sep  7 21:05  .tmux

-rw-r--r-- 1 tangleroot013 tangleroot013    2915 Sep  7 21:40  .tmux.conf

-rw-r--r-- 1 tangleroot013 tangleroot013     140 Jul  5 06:41  .tmux.zip

-rw-r--r-- 1 tangleroot013 tangleroot013     942 Sep  4 20:48  uninstall_script.bash

-rwxr-xr-x 1 tangleroot013 tangleroot013    1197 Aug  9 04:03  up-wg.sh

-rw-r--r-- 1 tangleroot013 tangleroot013   11720 Jul  9 20:23  vanguard.mjs

drwxr-xr-x 1 tangleroot013 tangleroot013       6 Dec 10  2025  .var

drwxr-xr-x 1 tangleroot013 tangleroot013      66 Aug  9 12:14  .venv

-rwxr-xr-x 1 tangleroot013 tangleroot013    3266 Sep  9 18:51  verify_environment.py

-rwxr-xr-x 1 tangleroot013 tangleroot013    1310 Sep  7 11:16  verify_mpd_port.py

-rw------- 1 tangleroot013 tangleroot013   20653 Sep  8 20:01  .viminfo

drwxr-xr-x 1 tangleroot013 tangleroot013      70 Aug  4 12:47  .vscode

drwxr-xr-x 1 tangleroot013 tangleroot013      56 Jul  7 11:00  .vscode-react-native

drwxr-xr-x 1 tangleroot013 tangleroot013      34 Aug  4 10:52  .vscode-server

drwxr-xr-x 1 tangleroot013 tangleroot013      26 Jul  7 09:38  .vscode-shared

drwxr-xr-x 1 tangleroot013 tangleroot013     138 Jul  8 23:26  .wakatime

-rwxr-xr-x 1 tangleroot013 tangleroot013      24 Sep  4 17:31  wg-diagnose.sh

-rw-r--r-- 1 tangleroot013 tangleroot013     259 Sep  8 19:54  .wget-hsts

-rw-r--r-- 1 tangleroot013 tangleroot013    1214 Sep  5 23:47  wherewasi.py

-rwxr-xr-x 1 tangleroot013 tangleroot013    4105 Sep  4 16:26  wireguard_setup.sh

-rw------- 1 tangleroot013 tangleroot013     104 Oct  2 20:00  .Xauthority

-rw-r--r-- 1 tangleroot013 tangleroot013     275 Sep 20 07:09  .xtimidity

drwxr-xr-x 1 tangleroot013 tangleroot013      32 Jul  7 10:11  .yarn

-rw-r--r-- 1 tangleroot013 tangleroot013  119856 Jul  7 10:42  yarn.lock

-rw-r--r-- 1 tangleroot013 tangleroot013   51059 Sep  6 05:46  .zcompdump

drwxr-xr-x 1 tangleroot013 tangleroot013     114 Aug  6 06:03  .zsh

drwxr-xr-x 1 tangleroot013 tangleroot013      48 Jul  6 08:21  .zsh_backup

drwxr-xr-x 1 tangleroot013 tangleroot013     930 Sep  8 02:00  .zsh_backups

-rw------- 1 tangleroot013 tangleroot013 4036962 Oct  2 20:12  .zsh_history

-rw------- 1 tangleroot013 tangleroot013    3757 Oct  2 00:51  .zshrc

-rw-r--r-- 1 tangleroot013 tangleroot013   15741 Sep  2 00:24  .zshrc.backup.20260902-002403

-rw------- 1 tangleroot013 tangleroot013    3757 Oct  2 14:20  .zshrc.backup.20261002-142011

-rw-r--r-- 1 tangleroot013 tangleroot013   35887 Oct  2 20:02  .zshrc.local

-rw-r--r-- 1 tangleroot013 tangleroot013   12592 Sep  2 00:42  .zshrc.local.bak.20260902_004416

-rw-r--r-- 1 tangleroot013 tangleroot013   16202 Sep  8 20:10  .zshrc.local.bak.20260908201046

-r--r--r-- 1 tangleroot013 tangleroot013    8096 Sep  6 08:22  .zshrc.zwc



~ on  main [!⇡] via 🥟 v1.3.14 via 🐍 v3.11.2 

⬢ [Systemd] ❯ printf 'SSH_AUTH_SOCK=%q\n' "${SSH_AUTH_SOCK-}"

test -S /home/tangleroot013/.ssh/agent.sock && echo 'agent socket: present' || echo 'agent socket: missing'

ssh-add -l 2>&1

git status --short --branch



SSH_AUTH_SOCK=/home/tangleroot013/.ssh/agent.sock

agent socket: present

The agent has no identities.

## main...origin/main [ahead 1]

 M .zshrc

 M .zshrc.local

 M crostini_hardener.py

 M leak-test.sh

 M scripts/build_music_library.py

 M scripts/cleanup_music_library.py

 M scripts/organize_music.py



~ on  main [!⇡] via 🥟 v1.3.14 via 🐍 v3.11.2 

⬢ [Systemd] ❯ find "$HOME/.ssh" -maxdepth 1 -type f -name '*.pub' -printf '%f\n' 2>/dev/null





~ on  main [!⇡] via 🥟 v1.3.14 via 🐍 v3.11.2 

⬢ [Systemd] ❯ cd ez_mcp



ez_mcp on  main 

⬢ [Systemd] ❯ ls -la

total 0

drwxr-xr-x 1 tangleroot013 tangleroot013    8 Oct  2 06:36 .

drwxr-xr-x 1 tangleroot013 tangleroot013 5444 Oct  2 22:47 ..

drwxr-xr-x 1 tangleroot013 tangleroot013  108 Oct  2 11:02 .git



ez_mcp on  main 

⬢ [Systemd] ❯ git status --short            

git log -2 --oneline --decorate

git show --stat --oneline HEAD

git diff HEAD^ HEAD --check



fatal: your current branch 'main' does not have any commits yet

fatal: ambiguous argument 'HEAD': unknown revision or path not in the working tree.

Use '--' to separate paths from revisions, like this:

'git <command> [<revision>...] -- [<file>...]'

fatal: ambiguous argument 'HEAD^': unknown revision or path not in the working tree.

Use '--' to separate paths from revisions, like this:

'git <command> [<revision>...] -- [<file>...]'



ez_mcp on  main 

⬢ [Systemd] ❯ >....                                                                               

    else:

        print(f"Unknown command: {cmd}")

PYEOF



# --- Scaffold Source ---

cat <<'PYEOF' > parts/scaffold_old.py

def main(argv=None):

    print("Scaffolding project...")

    # Scaffold logic here

PYEOF



# Assembly Logic

echo "[*] Assembling ez_mcp polyglot..."



# Process Scaffold

SCAF_RAW=$(cat parts/scaffold_old.py)

SCAF_FIXED=$(echo "$SCAF_RAW" | sed 's/def main(argv=None):/def scaffold_main(argv=None):/')



# Concatenate Python Payload

PAYLOAD=$(cat parts/part1_core.py parts/part2_diag.py parts/part3_tools.py parts/part4_server_cli.py parts/part5_friendly.py)

FINAL_PYTHON="${SCAF_FIXED}\n${PAYLOAD}"



# Build Final Executable

cat <<BASHEOF > ez_mcp

#!/bin/bash

# ez_mcp Polyglot Launcher

case "\$(basename "\$0")" in

    ez_doctor) shift; set -- doctor "\$@" ;;

    ez_scan) shift; set -- scan "\$@" ;;

    ez_status) shift; set -- status "\$@" ;;

    ez_serve) shift; set -- serve "\$@" ;;

esac



python3 /dev/fd/3 "\$@" 3<<'PYEOF'

$(echo -e "$FINAL_PYTHON")

PYEOF

BASHEOF



chmod +x ez_mcp

echo "[+] Build complete."

EOF



bash build_ez_mcp.sh



# Symlink Routing

ln -sf ez_mcp ez_doctor

ln -sf ez_mcp ez_scan

ln -sf ez_mcp ez_status

ln -sf ez_mcp ez_serve



# Git Commit

git add .

git commit -m "feat: initial atomic build of ez_mcp polyglot"



[*] Assembling ez_mcp polyglot...

[+] Build complete.

[main (root-commit) ec580ad] feat: initial atomic build of ez_mcp polyglot

 12 files changed, 212 insertions(+)

 create mode 100644 build_ez_mcp.sh

 create mode 120000 ez_doctor

 create mode 100755 ez_mcp

 create mode 120000 ez_scan

 create mode 120000 ez_serve

 create mode 120000 ez_status

 create mode 100644 parts/part1_core.py

 create mode 100644 parts/part2_diag.py

 create mode 100644 parts/part3_tools.py

 create mode 100644 parts/part4_server_cli.py

 create mode 100644 parts/part5_friendly.py

 create mode 100644 parts/scaffold_old.py



ez_mcp on  main 

⬢ [Systemd] ❯ >....                                                                               

    

    if cmd in core.registry:

        try:

            core.registry[cmd].func(*argv[1:])

        except Exception as e:

            print(f"Runtime Error: {e}")

    else:

        print(f"Unknown command: {cmd}. Quack!")

PYEOF



# --- Scaffold Source ---

cat <<'PYEOF' > parts/scaffold_old.py

def scaffold_main(argv=None):

    print("Core scaffolding engine active...")

PYEOF



# --- ASSEMBLY ---

SCAF_RAW=$(cat parts/scaffold_old.py)

PAYLOAD=$(cat parts/part1_core.py parts/part2_diag.py parts/part3_tools.py parts/part4_server_cli.py parts/part5_friendly.py)



cat <<BASHEOF > ez_mcp

#!/bin/bash

# ez_mcp Polyglot Launcher

case "\$(basename "\$0")" in

    ez_doctor) shift; set -- doctor "\$@" ;;

    ez_scan) shift; set -- scan "\$@" ;;

    ez_status) shift; set -- status "\$@" ;;

    ez_scaffold) shift; set -- scaffold "\$@" ;;

    ez_serve) shift; set -- serve "\$@" ;;

    ez_fix_dns) shift; set -- fix-dns "\$@" ;;

esac



python3 /dev/fd/3 "\$@" 3<<'PYEOF'

${SCAF_RAW}

${PAYLOAD}

if __name__ == "__main__":

    main()

PYEOF

BASHEOF



chmod +x ez_mcp



# Symlink Routing

ln -sf ez_mcp ez_doctor

ln -sf ez_mcp ez_scan

ln -sf ez_mcp ez_status

ln -sf ez_mcp ez_scaffold

ln -sf ez_mcp ez_serve

ln -sf ez_mcp ez_fix_dns



# Git Commit

git add .

git commit -m "feat: atomic implementation of ez_mcp polyglot toolbox"



[main ff6afe4] feat: atomic implementation of ez_mcp polyglot toolbox

 9 files changed, 281 insertions(+), 65 deletions(-)

 create mode 120000 ez_fix_dns

 create mode 120000 ez_scaffold



ez_mcp on  main 

⬢ [Systemd] ❯ >....                                                                               

    ctx = Ctx(origin="mcp")

    while True:

        line = sys.stdin.readline()

        if not line: break

        try:

            resp = handle_message(json.loads(line), ctx)

            if resp: print(json.dumps(resp)); sys.stdout.flush()

        except: pass



core.register("serve", serve)

PYEOF



# --- PART 5: Friendly CLI & Interface ---

cat <<'PYEOF' > parts/part5_friendly.py

import os, sys, json, shutil



ALIASES = {

    "ez_doctor": "doctor", "ez_scan": "scan", "ez_status": "status", "ez_scaffold": "scaffold",

    "ez_serve": "serve", "ez_fix_dns": "fix-dns", "ez_config": "config", "ez_tools": "tools",

}



COMMAND_HELP = {

    "menu": "interactive menu",

    "doctor": "health check",

    "status": "git overview",

    "scan": "scan text/files",

    "scaffold": "DevContainer generator",

    "fix-dns": "repair /etc/resolv.conf",

    "serve": "run MCP server",

    "selftest": "end-to-end test",

    "install": "install into ~/bin",

    "completion": "shell completion",

}



def _is_ours(path):

    try:

        with open(path, encoding="utf-8", errors="replace") as fh:

            return LAUNCHER_MARKER in fh.read(600)

    except: return False



def cmd_install(args):

    dest = os.path.expanduser("~/bin")

    os.makedirs(dest, exist_ok=True)

    content = open(self_path()).read()

    safe_write_file_atomic(os.path.join(dest, APP), content, perm=0o755)

    for alias, cmd in ALIASES.items():

        os.symlink(APP, os.path.join(dest, alias))

    log_step("Installed", f"Binaries placed in {dest}")

    return 0



def cmd_selftest(args):

    from parts.part4_server_cli import handle_message

    ctx = Ctx(origin="selftest", roots=["/tmp"])

    # Mocking a few calls



∙ PYEOF'

∙                                                                 



ez_mcp on  main 

⬢ [Systemd] ❯ >....                                                                               





2. **Build Assembly (`build_ez_mcp.sh`):**

* The script concatenated core Python modules (`part1_core.py` through `part5_friendly.py` and `scaffold_old.py`) into the polyglot shell wrapper `ez_mcp`.

* Created symbolic links routing specific commands (`ez_doctor`, `ez_scan`, `ez_status`, `ez_scaffold`, `ez_serve`, `ez_fix_dns`) back to the main executable.





3. **Commit Completion:**

* Executing `git add .` and `git commit -m "feat: atomic implementation of ez_mcp polyglot toolbox"` successfully established the initial repository commit (`ff6afe4`), resolving the working tree revision errors.







---



## Open-Source Maintenance & System Handover Strategy



To maintain `ez_mcp` effectively as an active open-source project and ensure smooth maintenance transitions:



### Automated Maintenance & Stale Management



* **Automated Staleness Policy:** Implementing workflows like `actions/stale` scans inactive issues and PRs on a schedule, applying a `stale` label before auto-closing.





* **Differentiated Timers:** Issues and pull requests often require distinct inactivity thresholds (e.g., 360 days before marking an issue stale vs. 180 days for a PR).





* **Label Exemptions:** High-signal issues or PRs awaiting maintainer input should be exempted using labels such as `needs-triage` or `needs-review`.





* **Triage & Burnout Prevention:** Establishing regular, timed triage sessions (e.g., 15 minutes daily) and using issue templates streamline incoming maintenance requests.







### Documentation & Knowledge Transfer



* **Software Handover Protocol:** Software transitions require transferring system knowledge, documentation repositories, and code bases systematically.





* **Repository Documentation:** Maintaining comprehensive `CONTRIBUTING.md` files, pull request checklists, and architecture guides prevents knowledge loss during maintainer handovers.





* **Branch Strategy:** Maintaining clear branching structures (such as `main` for master releases, feature branches for development, and maintenance branches for patch releases) provides operational stability during software transfers.

zsh: command not found: ez_mcp

zsh: unknown file attribute: C

[1] 1270

zsh: no matches found: **CLI

[1]  + exit 1     * **CLI

zsh: no matches found: Server:**

zsh: command not found: stdin

zsh: no matches found: **Polyglot

zsh: command not found: ez_doctor

zsh: command not found: ez_scan

zsh: command not found: ez_status

zsh: command not found: ez_serve

zsh: command not found: ez_scaffold

zsh: command not found: ez_fix_dns

basename: missing operand

Try 'basename --help' for more information.

zsh: no matches found: **Symlink

zsh: = not found

zsh: permission denied: /home/tangleroot013/github_projects

zsh: permission denied: /home/tangleroot013/ez_mcp

zsh: command not found: EZ_MCP_ROOTS

zsh: command not found: --root

zsh: no matches found: **Root

zsh: command not found: --allow-write

zsh: no matches found: **Read-Only

zsh: command not found: SAFE_ENV_KEYS

zsh: no matches found: **Isolated

zsh: command not found: ---

zsh: command not found: The

zsh: no matches found: **Initial

zsh: command not found: fatal:

zsh: command not found: fatal:

zsh: command not found: build_ez_mcp.sh

zsh: no matches found: **Cause:**

zsh: command not found: build_ez_mcp.sh

zsh: no matches found: **Build

zsh: command not found: part1_core.py

zsh: command not found: part5_friendly.py

zsh: command not found: scaffold_old.py

zsh: command not found: ez_mcp

zsh: unknown file attribute:  

zsh: command not found: ez_doctor

zsh: command not found: ez_scan

zsh: command not found: ez_status

zsh: command not found: ez_scaffold

zsh: command not found: ez_serve

zsh: command not found: ez_fix_dns

zsh: unknown file attribute:  

zsh: no matches found: **Commit

zsh: command not found: ff6afe4

zsh: no matches found: (),

zsh: command not found: ---

zsh: command not found: ez_mcp

zsh: command not found: To

zsh: no such file or directory: actions/stale

zsh: command not found: stale

zsh: no matches found: **Automated

zsh: no matches found: **Differentiated

zsh: command not found: needs-triage

zsh: command not found: needs-review

zsh: no matches found: **Label

zsh: no matches found: **Triage

[1] 1336

zsh: no matches found: Prevention:**

[1]  + exit 1     * **Triage

zsh: no matches found: **Software

zsh: command not found: CONTRIBUTING.md

zsh: no matches found: **Repository

zsh: command not found: main

zsh: no matches found: **Branch



ez_mcp on  main 

⬢ [Systemd] ❯ >....                                                                               

    runs-on: ubuntu-latest

    steps:

      - uses: actions/stale@v8

        with:

          stale-issue-message: 'This issue has been inactive for 360 days and will be marked as stale.'

          stale-pr-message: 'This PR has been inactive for 180 days and will be marked as stale.'

          days-before-stale-issue: 360

          days-before-stale-pr: 180

          stale-issue-label: 'stale'

          stale-pr-label: 'stale'

          exempt-labels: 'needs-triage,needs-review,pinned'

EOF



# 2. Establish the Handover Protocol (CONTRIBUTING.md)

# Standardizing how new maintainers enter the pond.

cat <<'EOF' > CONTRIBUTING.md

# Contributing to ez_mcp



## Architecture Overview

ez_mcp is a polyglot Bash/Python utility. The build process is atomic: 

`parts/*.py` -> `build_ez_mcp.sh` -> `ez_mcp` (binary).



## Development Workflow

1. Modify the logic in the `parts/` directory.

2. Run the build script to re-assemble the polyglot launcher.

3. Execute `./ez_selftest` to verify the MCP handler remains intact.



## Maintenance Policy

- **Staleness:** Inactive issues (360d) and PRs (180d) are auto-closed.

- **Branching:** 

  - `main`: Stable releases only.

  - `feat/*`: New feature development.

  - `patch/*`: Critical bug fixes.

EOF



# 3. Create the Maintainers Registry (MAINTAINERS.md)

# Essential for the "System Handover Strategy".

cat <<'EOF' > MAINTAINERS.md

# Project Maintainers



| Name | Role | Responsibility |

| :--- | :--- | :--- |

| bilbywilby | Lead Architect | Core logic, OPSEC, Build Pipeline |

| Carter | Tech Support | Maintenance, Crostini Optimization |



## Handover Protocol

Knowledge transfer involves the transfer of the GitHub Org admin rights and a review of the `CONTRIBUTING.md` and `ez_mcp` polyglot assembly logic.

EOF



# Git Commit of Maintenance Infrastructure

git add .

git commit -m "chore: implement automated maintenance and handover infrastructure"



[main 5cc9d73] chore: implement automated maintenance and handover infrastructure

 3 files changed, 44 insertions(+)

 create mode 100644 .github/workflows/stale.yml

 create mode 100644 CONTRIBUTING.md

 create mode 100644 MAINTAINERS.md



ez_mcp on  main 

⬢ [Systemd] ❯ >....                                                                               

n in the final concatenated Python blob?"

  },

  {

    "timestamp": "2026-10-03T10:13:40Z",

    "speaker": "bilbywilby",

    "category": "Build_Process",

    "message": "That's why the 'parts/' structure is modular. Each part is designed as a functional block. We use a registry pattern in part1_core.py. Tools register themselves into the core.registry dictionary, which the main loop in part5_friendly.py calls. It's essentially a micro-kernel architecture inside a single file."

  },

  {

    "timestamp": "2026-10-03T10:15:10Z",

    "speaker": "Carter_Duck",

    "category": "Crostini_Pitfall",

    "message": "Don't forget that we're running in Crostini! The devs need to remember that the Linux container filesystem is distinct from ChromeOS. If they try to point the roots to /mnt/chromeos, they'll hit permission walls unless they've shared those folders with Linux. Quack!"

  },

  {

    "timestamp": "2026-10-03T10:18:00Z",

    "speaker": "Security_Sam",

    "category": "OPSEC",

    "message": "And the symlink routing? ez_doctor, ez_scan... it's all one file?"

  },

  {

    "timestamp": "2026-10-03T10:20:00Z",

    "speaker": "bilbywilby",

    "category": "Architecture",

    "message": "Exactly. BusyBox style. The bash wrapper checks $(basename $0). If the user calls 'ez_doctor', the wrapper shifts the arguments and tells the Python payload to execute the 'doctor' command. It gives the user a clean CLI experience without the overhead of 10 different Python environments."

  },

  {

    "timestamp": "2026-10-03T10:25:00Z",

    "speaker": "DevLead_Alex",

    "category": "Maintenance",

    "message": "Understood. So the workflow is: Edit parts/*.py -> bash build_ez_mcp.sh -> ./ez_selftest. If the selftest passes and the git commit is atomic, we're good to go."

  },

  {

    "timestamp": "2026-10-03T10:26:30Z",

    "speaker": "bilbywilby",

    "category": "Closing",

    "message": "Correct. The system is designed to be robust, stealthy, and easy to transport. Just keep the build atomic, and don't touch the FD 3 routing unless you have a very good reason."

  }

]

JSONEOF



# Git Commit

git add HANDOVER.json

git commit -m "docs: generate extensive conversation-based handover JSON for dev team"



[main 69d6486] docs: generate extensive conversation-based handover JSON for dev team

 1 file changed, 80 insertions(+)

 create mode 100644 HANDOVER.json



ez_mcp on  main 

⬢ [Systemd] ❯ continue with 10 power user addons

ez_mcp
Custom Gem
I will now generate the new power user module parts/part6_power_addons.py along with an updated build_ez_mcp.sh build orchestrator to integrate these 10 tools seamlessly into your ez_mcp ecosystem.

Power User Addons Module
"""
parts/part6_power_addons.py - 10 Power User Addons for ez_mcp
Extends ez_mcp with advanced diagnostics, workflow automation, and system inspection.
"""

import os
import sys
import json
import time
import re

Updated Build Script
#!/bin/bash

ez_mcp Polyglot Assembly & Build Pipeline with Power Addons
set -euo pipefail

echo "[*] Assembling ez_mcp polyglot executable with 10 Power Addons..."

1. Read Python Part Modules
SCAF_RAW=(catparts/scaffold 
o
​
 ld.py)PAYLOAD=(cat parts/part1_core.py parts/part2_diag.py parts/part3_tools.py parts/part4_server_cli.py parts/part5_friendly.py parts/part6_power_addons.py)

2. Construct Polyglot Script
cat <<'BASHEOF' > ez_mcp
#!/bin/bash

ez_mcp Polyglot Launcher
case "(basename "$0")" in ez_doctor) shift; set -- doctor "$@" ;; ez_scan) shift; set -- scan "@" ;;
ez_status) shift; set -- status "@";;ez 
s
​
 caffold)shift;set−−scaffold"@" ;;
ez_serve) shift; set -- serve "@";;ez 
f
​
 ix 
d
​
 ns)shift;set−−fix−dns"@" ;;
ez_portwatch) shift; set -- portwatch "@";;ez 
e
​
 nvguard)shift;set−−envguard"@" ;;
ez_gitprune) shift; set -- gitprune "@";;ez 
s
​
 ysmetrics)shift;set−−sysmetrics"@" ;;
ez_jsondiff) shift; set -- jsondiff "@";;ez 
n
​
 etping)shift;set−−netping"@" ;;
ez_procinfo) shift; set -- procinfo "@";;ez 
q
​
 uickbench)shift;set−−quickbench"@" ;;
ez_logtail) shift; set -- logtail "@";;ez 
m
​
 cp 
i
​
 nspect)shift;set−−mcp−inspect"@" ;;
esac

python3 /dev/fd/3 "$@" 3<<'PYEOF'
BASHEOF

echo "SCAF 
R
​
 AW">>ez 
m
​
 cpecho"{PAYLOAD}" >> ez_mcp

cat <<'BASHEOF' >> ez_mcp

if name == "main":
# Register core & power addons
if 'register_power_addons' in globals():
register_power_addons(core.registry if 'core' in globals() else {})
main()
PYEOF
BASHEOF

chmod +x ez_mcp

3. Create Symlinks for CLI Routing
echo "[*] Generating CLI Symlinks..."
for cmd in ez_doctor ez_scan ez_status ez_scaffold ez_serve ez_fix_dns 

ez_portwatch ez_envguard ez_gitprune ez_sysmetrics ez_jsondiff 

ez_netping ez_procinfo ez_quickbench ez_logtail ez_mcp_inspect; do
ln -sf ez_mcp "$cmd"
done

echo "[+] Build complete. Created ez_mcp binary with 10 Power Addons and routing symlinks."

Edit directly or with Gemini
 Click anywhere to type and edit directly, or select text to prompt Gemini for changes. 
I have generated the parts/part6_power_addons.py power user module along with the updated build_ez_mcp.sh build script.

Summary of Added Power Addons:
ez_portwatch: Audits open sockets and listening process ports via ss/netstat.

ez_envguard: Scans for credential leaks in environment variables and checks PATH sanity.

ez_gitprune: Cleans merged local branches, untracked artifacts, and runs garbage collection.

ez_sysmetrics: Monitors CPU load averages, RAM allocation, and local disk usage.

ez_jsondiff: Key-by-key structural difference tool for JSON and configuration files.

ez_netping: Fast DNS resolution and network latency tester.

ez_procinfo: Process inspector and memory/CPU task auditor.

ez_quickbench: Micro-benchmarking execution timing tool.

ez_logtail: Tails logs and aggregates error/warning frequencies.

ez_mcp_inspect: Offline JSON-RPC 2.0 schema validator and MCP simulator.
# --- HANDOVER JSON GENERATION ---
cat <<'JSONEOF' > HANDOVER.json
[
  {
    "timestamp": "2026-10-03T10:00:00Z",
    "speaker": "bilbywilby",
    "category": "Architecture",
    "message": "Welcome to the ez_mcp handover. The core philosophy here is 'Zero-Dependency Atomicity'. We've built a polyglot launcher that allows the tool to behave like a suite of separate binaries while maintaining a single source of truth in Python."
  },
  {
    "timestamp": "2026-10-03T10:02:15Z",
    "speaker": "DevLead_Alex",
    "category": "Technical_Query",
    "message": "I'm looking at the bash wrapper. Why the heck are we using /dev/fd/3? Why not just a standard shebang or a temporary file?"
  },
  {
    "timestamp": "2026-10-03T10:03:10Z",
    "speaker": "bilbywilby",
    "category": "OPSEC",
    "message": "Standard shebangs are too loud, and temp files leave a forensic footprint on the disk. By piping the Python payload through FD 3, we keep the actual logic in memory. More importantly, it keeps STDIN (FD 0) completely clean for the MCP JSON-RPC stream. If we used a standard pipe, the AI agent's input would collide with the script's loading process."
  },
  {
    "timestamp": "2026-10-03T10:04:45Z",
    "speaker": "Carter_Duck",
    "category": "Operational_Note",
    "message": "Quack! Just a warning for the team: if you try to debug this with a standard debugger that redirects stdin, you're going to have a bad time. Stick to log-stepping or custom print statements in the parts/*.py files."
  },
  {
    "timestamp": "2026-10-03T10:07:00Z",
    "speaker": "Security_Sam",
    "category": "Security_Audit",
    "message": "Let's talk about the Root Confinement. I see the 'origin' check in the Ctx class. How strictly is this enforced when an AI agent calls a tool?"
  },
  {
    "timestamp": "2026-10-03T10:09:20Z",
    "speaker": "bilbywilby",
    "category": "Security_Audit",
    "message": "Strictly. If origin == 'mcp', every file path is resolved against a whitelist of roots (defaulting to ~/github_projects). Any attempt to escape via '../' or absolute paths outside the root triggers an immediate AccessDenied exception. The --allow-write flag is the only way to mutate the filesystem, and that's a boot-time requirement, not a runtime request."
  },
  {
    "timestamp": "2026-10-03T10:12:00Z",
    "speaker": "DevLead_Alex",
    "category": "Build_Process",
    "message": "The build script just cats files together. Is there any risk of namespace collision in the final concatenated Python blob?"
  },
  {
    "timestamp": "2026-10-03T10:13:40Z",
    "speaker": "bilbywilby",
    "category": "Build_Process",
    "message": "That's why the 'parts/' structure is modular. Each part is designed as a functional block. We use a registry pattern in part1_core.py. Tools register themselves into the core.registry dictionary, which the main loop in part5_friendly.py calls. It's essentially a micro-kernel architecture inside a single file."
  },
  {
    "timestamp": "2026-10-03T10:15:10Z",
    "speaker": "Carter_Duck",
    "category": "Crostini_Pitfall",
    "message": "Don't forget that we're running in Crostini! The devs need to remember that the Linux container filesystem is distinct from ChromeOS. If they try to point the roots to /mnt/chromeos, they'll hit permission walls unless they've shared those folders with Linux. Quack!"
  },
  {
    "timestamp": "2026-10-03T10:18:00Z",
    "speaker": "Security_Sam",
    "category": "OPSEC",
    "message": "And the symlink routing? ez_doctor, ez_scan... it's all one file?"
  },
  {
    "timestamp": "2026-10-03T10:20:00Z",
    "speaker": "bilbywilby",
    "category": "Architecture",
    "message": "Exactly. BusyBox style. The bash wrapper checks $(basename $0). If the user calls 'ez_doctor', the wrapper shifts the arguments and tells the Python payload to execute the 'doctor' command. It gives the user a clean CLI experience without the overhead of 10 different Python environments."
  },
  {
    "timestamp": "2026-10-03T10:25:00Z",
    "speaker": "DevLead_Alex",
    "category": "Maintenance",
    "message": "Understood. So the workflow is: Edit parts/*.py -> bash build_ez_mcp.sh -> ./ez_selftest. If the selftest passes and the git commit is atomic, we're good to go."
  },
  {
    "timestamp": "2026-10-03T10:26:30Z",
    "speaker": "bilbywilby",
    "category": "Closing",
    "message": "Correct. The system is designed to be robust, stealthy, and easy to transport. Just keep the build atomic, and don't touch the FD 3 routing unless you have a very good reason."
  }
]
JSONEOF

# Git Commit
git add HANDOVER.json
git commit -m "docs: generate extensive conversation-based handover JSON for dev team"The terminal output errors occurred because raw Markdown and shell configuration text were pasted directly into an active Zsh prompt, causing the shell to execute unquoted words, wildcards, and file paths as terminal commands.

### Terminal Error Analysis & Resolution

* **`zsh: command not found: ez_mcp`**: Zsh attempted to run `ez_mcp` as a binary in system `$PATH`. Run `./ez_mcp` if executing from the local directory.
* **`zsh: no matches found: **CLI`**: Asterisks (`**`) triggered Zsh filename generation (globbing). Because no local files matched `**CLI` or `**Polyglot`, Zsh threw a pattern match error.
* **`zsh: permission denied: /home/...`**: Zsh attempted to execute directory paths as executable programs.
* **Command Not Found Errors**: Line breaks caused Zsh to evaluate the first word of every sentence or header (e.g., `stdin`, `main`, `ff6afe4`) as a system command.

To restore your prompt, press `Ctrl + C` to terminate any hanging jobs and run `clear`. To write multi-line text or YAML workflow files safely without execution errors, use a quoted heredoc:

```bash
cat << 'EOF' > .github/workflows/stale.yml
name: Stale
on:
  schedule:
    - cron: "0 0 * * *"
jobs:
  stale:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/stale@v8
        with:
          days-before-issue-stale: 360
          days-before-pr-stale: 180
          exempt-issue-labels: 'needs-triage'
          exempt-pr-labels: 'needs-review'
EOF

```

---

### `ez_mcp` Tool Architecture & Capabilities

* **Dual-Purpose Design**: Acts as both a Model Context Protocol (MCP) server for AI coding agents (operating in a read-only safety model by default) and a local CLI developer toolbox.
* **Core Command Suite**:
* `doctor`: Runs health checks on system DNS (`resolv.conf`), active network interfaces, file permissions, disk space, and installed tools.
* `status`: Displays git status across multi-repository directories (branch state, ahead/behind counts, untracked changes).
* `scan`: Analyzes text or pasted AI outputs for prompt injection markers, hidden Unicode, and credential leaks.
* `scaffold`: Atomic generator for DevContainer, Docker Compose, PostgreSQL, and MCP server boilerplate.


* **Standalone Execution**: Commands can be copied or symlinked as independent executables (e.g., `ez_doctor`, `ez_scan`) running pure Python 3.9+ without external dependencies.

---

### Open-Source Maintenance & System Handover Strategy

* **Automated Stale Management**: Implementing the `actions/stale` GitHub Action automates repository maintenance by scheduling periodic scans, labeling inactive issues and pull requests, and closing them if inactivity persists.


* **Configurable Inactivity Timers**: Setting distinct timers—such as 360 days before marking an issue stale versus 180 days for a pull request—prevents premature closure of open bug discussions while clearing abandoned PRs.


* **Protecting High-Signal Items**: Exempting specific labels (`needs-triage`, `needs-review`, `pinned`) ensures critical items awaiting maintainer action are not automatically closed.


* **System Handover Protocols**: Transitioning software responsibilities requires transferring source code, test suites, modification histories, and operational context. Maintaining a centralized documentation repository ensures system knowledge is retained during team transitions. Handover processes ideally span a one-month window to allow two weeks of documentation and two weeks of active support.


* **Branching Strategy**: Implementing structured Git workflows—such as isolating a `main` release branch, a `development` branch, and a long-lived `maintenance` branch—preserves release stability throughout maintainer handovers.You pasted formatted Markdown text directly into an active Zsh prompt, causing the shell to execute unquoted text, asterisks, and headings as terminal commands.

### Terminal Errors Breakdown

* **`zsh: command not found: ez_mcp`**: Zsh tried to run `ez_mcp` as a binary from your system `$PATH`. If the binary is in your current directory, run `./ez_mcp` instead.
* **`zsh: no matches found: **CLI`**: Asterisks (`*`) trigger Zsh file-globbing. Because no files in your directory matched `**CLI` or `**Polyglot`, Zsh failed the expansion.
* **`zsh: command not found: [word]`**: Sentences and section headers were read line-by-line, causing Zsh to attempt to run the first word of every line (e.g., `stdin`, `main`, `ff6afe4`) as a system command.

To clear your prompt, press `Ctrl + C` and type `clear`. If you intended to save that text to a file, wrap it in a heredoc or open a text editor:

```bash
cat << 'EOF' > README.md
# Paste your text here
EOF

```

---

### Key Takeaways from the Document Context

#### 1. The `ez_mcp` Toolbox

* **Dual-Purpose Architecture**: Operates both as a Model Context Protocol (MCP) server for AI coding assistants (with a default read-only safety model) and a local CLI toolbox for developer tasks.
* **Key Commands**: Provides environment health checks (`doctor`), multi-repo git overviews (`status`), prompt injection and secret scanning (`scan`), and atomic project generation (`scaffold`).
* **Standalone Deployment**: Can be copied or symlinked as independent executables (e.g., `ez_doctor`, `ez_scan`), running purely via Python 3 without external dependencies.

#### 2. Open-Source Maintenance & Handover Best Practices

* **Automated Staleness Policy**: Using workflows like `actions/stale` automates scanning inactive issues and pull requests, applying labels (e.g., `stale`), and closing them after a set period of inactivity.


* **Differentiated Timers & Exemptions**: Maintainers often set distinct inactivity thresholds for issues versus pull requests (e.g., 360 days before marking an issue stale versus 180 days for a PR) and exempt high-signal items using labels like `needs-triage` or `needs-review`.


* **Structured System Handover**: Handing over a software repository requires systematically transferring source code, test suites, modification request history, and operational data between initial developers and post-delivery maintenance teams.


* **Clear Branching Strategy**: Establishing standardized Git workflows—such as isolating main release branches from active feature branches—helps preserve repository stability during maintainer transitions.## Overview of `ez_mcp` Architecture & Build Mechanics

`ez_mcp` is designed as a zero-dependency, single-file polyglot developer utility and Model Context Protocol (MCP) server.

### Dual-Interface Architecture

* **CLI & Stdio MCP Server:** Operates as both an interactive CLI / terminal utility for developers and a stdio MCP server for AI coding agents like Claude Code and Cursor.
* **Polyglot Execution:** Uses a Bash launcher that feeds an embedded Python 3.9+ payload to standard input via file descriptor 3 (`python3 /dev/fd/3 "$@" 3<<'PYEOF'`). This leaves `stdin` open for interactive terminal prompts or JSON-RPC streams.
* **Symlink Multi-Call Routing:** Employs BusyBox-style command aliasing. Symlinks named after specific subcommands (e.g., `ez_doctor`, `ez_scan`, `ez_status`, `ez_serve`, `ez_scaffold`, `ez_fix_dns`) inspect `$(basename "$0")` at runtime to dispatch to the appropriate tool directly.

### Safety Model & Gating

* **Root Confinement:** Restricts file access for AI callers (`origin == "mcp"`) to configured directory roots (defaults to `~/github_projects` and `~/ez_mcp`, customizable via `EZ_MCP_ROOTS` or `--root`).
* **Read-Only Default:** Mutation tools (such as project scaffolding or running test suites) require starting the server with `--allow-write`. Dry-run capable tools automatically revert to `dry_run=True` if write permissions are absent.
* **Isolated Command Execution:** External processes run without shell invocation, using scrubbed environments (`SAFE_ENV_KEYS`) and strict timeouts.

---

## Analysis of Git Log Errors & Build Resolution

The terminal output reflects a sequence of initial setup commands and their resolution:

1. **Initial Git Revision Failures:**
* Running `git log -2 --oneline` and `git diff HEAD^ HEAD` produced `fatal: your current branch 'main' does not have any commits yet` and `fatal: ambiguous argument 'HEAD'`.
* **Cause:** These commands were executed on an initialized but empty repository prior to creating a root commit.


2. **Build Assembly (`build_ez_mcp.sh`):**
* The script concatenated core Python modules (`part1_core.py` through `part5_friendly.py` and `scaffold_old.py`) into the polyglot shell wrapper `ez_mcp`.
* Created symbolic links routing specific commands (`ez_doctor`, `ez_scan`, `ez_status`, `ez_scaffold`, `ez_serve`, `ez_fix_dns`) back to the main executable.


3. **Commit Completion:**
* Executing `git add .` and `git commit -m "feat: atomic implementation of ez_mcp polyglot toolbox"` successfully established the initial repository commit (`ff6afe4`), resolving the working tree revision errors.



---

## Open-Source Maintenance & System Handover Strategy

To maintain `ez_mcp` effectively as an active open-source project and ensure smooth maintenance transitions:

### Automated Maintenance & Stale Management

* **Automated Staleness Policy:** Implementing workflows like `actions/stale` scans inactive issues and PRs on a schedule, applying a `stale` label before auto-closing.


* **Differentiated Timers:** Issues and pull requests often require distinct inactivity thresholds (e.g., 360 days before marking an issue stale vs. 180 days for a PR).


* **Label Exemptions:** High-signal issues or PRs awaiting maintainer input should be exempted using labels such as `needs-triage` or `needs-review`.


* **Triage & Burnout Prevention:** Establishing regular, timed triage sessions (e.g., 15 minutes daily) and using issue templates streamline incoming maintenance requests.



### Documentation & Knowledge Transfer

* **Software Handover Protocol:** Software transitions require transferring system knowledge, documentation repositories, and code bases systematically.


* **Repository Documentation:** Maintaining comprehensive `CONTRIBUTING.md` files, pull request checklists, and architecture guides prevents knowledge loss during maintainer handovers.


* **Branch Strategy:** Maintaining clear branching structures (such as `main` for master releases, feature branches for development, and maintenance branches for patch releases) provides operational stability during software transfers.p on  main 
⬢ [Systemd] ❯ git status --short            
git log -2 --oneline --decorate
git show --stat --oneline HEAD
git diff HEAD^ HEAD --check

fatal: your current branch 'main' does not have any commits yet
fatal: ambiguous argument 'HEAD': unknown revision or path not in the working tree.
Use '--' to separate paths from revisions, like this:
'git <command> [<revision>...] -- [<file>...]'
fatal: ambiguous argument 'HEAD^': unknown revision or path not in the working tree.
Use '--' to separate paths from revisions, like this:
'git <command> [<revision>...] -- [<file>...]'

ez_mcp on  main 
⬢ [Systemd] ❯ >....                                                                               
    else:
        print(f"Unknown command: {cmd}")
PYEOF

# --- Scaffold Source ---
cat <<'PYEOF' > parts/scaffold_old.py
def main(argv=None):
    print("Scaffolding project...")
    # Scaffold logic here
PYEOF

# Assembly Logic
echo "[*] Assembling ez_mcp polyglot..."

# Process Scaffold
SCAF_RAW=$(cat parts/scaffold_old.py)
SCAF_FIXED=$(echo "$SCAF_RAW" | sed 's/def main(argv=None):/def scaffold_main(argv=None):/')

# Concatenate Python Payload
PAYLOAD=$(cat parts/part1_core.py parts/part2_diag.py parts/part3_tools.py parts/part4_server_cli.py parts/part5_friendly.py)
FINAL_PYTHON="${SCAF_FIXED}\n${PAYLOAD}"

# Build Final Executable
cat <<BASHEOF > ez_mcp
#!/bin/bash
# ez_mcp Polyglot Launcher
case "\$(basename "\$0")" in
    ez_doctor) shift; set -- doctor "\$@" ;;
    ez_scan) shift; set -- scan "\$@" ;;
    ez_status) shift; set -- status "\$@" ;;
    ez_serve) shift; set -- serve "\$@" ;;
esac

python3 /dev/fd/3 "\$@" 3<<'PYEOF'
$(echo -e "$FINAL_PYTHON")
PYEOF
BASHEOF

chmod +x ez_mcp
echo "[+] Build complete."
EOF

bash build_ez_mcp.sh

# Symlink Routing
ln -sf ez_mcp ez_doctor
ln -sf ez_mcp ez_scan
ln -sf ez_mcp ez_status
ln -sf ez_mcp ez_serve

# Git Commit
git add .
git commit -m "feat: initial atomic build of ez_mcp polyglot"

[*] Assembling ez_mcp polyglot...
[+] Build complete.
[main (root-commit) ec580ad] feat: initial atomic build of ez_mcp polyglot
 12 files changed, 212 insertions(+)
 create mode 100644 build_ez_mcp.sh
 create mode 120000 ez_doctor
 create mode 100755 ez_mcp
 create mode 120000 ez_scan
 create mode 120000 ez_serve
 create mode 120000 ez_status
 create mode 100644 parts/part1_core.py
 create mode 100644 parts/part2_diag.py
 create mode 100644 parts/part3_tools.py
 create mode 100644 parts/part4_server_cli.py
 create mode 100644 parts/part5_friendly.py
 create mode 100644 parts/scaffold_old.py

ez_mcp on  main 
⬢ [Systemd] ❯ >....                                                                               
    
    if cmd in core.registry:
        try:
            core.registry[cmd].func(*argv[1:])
        except Exception as e:
            print(f"Runtime Error: {e}")
    else:
        print(f"Unknown command: {cmd}. Quack!")
PYEOF

# --- Scaffold Source ---
cat <<'PYEOF' > parts/scaffold_old.py
def scaffold_main(argv=None):
    print("Core scaffolding engine active...")
PYEOF

# --- ASSEMBLY ---
SCAF_RAW=$(cat parts/scaffold_old.py)
PAYLOAD=$(cat parts/part1_core.py parts/part2_diag.py parts/part3_tools.py parts/part4_server_cli.py parts/part5_friendly.py)

cat <<BASHEOF > ez_mcp
#!/bin/bash
# ez_mcp Polyglot Launcher
case "\$(basename "\$0")" in
    ez_doctor) shift; set -- doctor "\$@" ;;
    ez_scan) shift; set -- scan "\$@" ;;
    ez_status) shift; set -- status "\$@" ;;
    ez_scaffold) shift; set -- scaffold "\$@" ;;
    ez_serve) shift; set -- serve "\$@" ;;
    ez_fix_dns) shift; set -- fix-dns "\$@" ;;
esac

python3 /dev/fd/3 "\$@" 3<<'PYEOF'
${SCAF_RAW}
${PAYLOAD}
if __name__ == "__main__":
    main()
PYEOF
BASHEOF

chmod +x ez_mcp

# Symlink Routing
ln -sf ez_mcp ez_doctor
ln -sf ez_mcp ez_scan
ln -sf ez_mcp ez_status
ln -sf ez_mcp ez_scaffold
ln -sf ez_mcp ez_serve
ln -sf ez_mcp ez_fix_dns

# Git Commit
git add .
git commit -m "feat: atomic implementation of ez_mcp polyglot toolbox"

[main ff6afe4] feat: atomic implementation of ez_mcp polyglot toolbox
 9 files changed, 281 insertions(+), 65 deletions(-)
 create mode 120000 ez_fix_dns
 create mode 120000 ez_scaffold

ez_mcp on  main 
⬢ [Systemd] ❯ >....                                                                               
    ctx = Ctx(origin="mcp")
    while True:
        line = sys.stdin.readline()
        if not line: break
        try:
            resp = handle_message(json.loads(line), ctx)
            if resp: print(json.dumps(resp)); sys.stdout.flush()
        except: pass

core.register("serve", serve)
PYEOF

# --- PART 5: Friendly CLI & Interface ---
cat <<'PYEOF' > parts/part5_friendly.py
import os, sys, json, shutil

ALIASES = {
    "ez_doctor": "doctor", "ez_scan": "scan", "ez_status": "status", "ez_scaffold": "scaffold",
    "ez_serve": "serve", "ez_fix_dns": "fix-dns", "ez_config": "config", "ez_tools": "tools",
}

COMMAND_HELP = {
    "menu": "interactive menu",
    "doctor": "health check",
    "status": "git overview",
    "scan": "scan text/files",
    "scaffold": "DevContainer generator",
    "fix-dns": "repair /etc/resolv.conf",
    "serve": "run MCP server",
    "selftest": "end-to-end test",
    "install": "install into ~/bin",
    "completion": "shell completion",
}

def _is_ours(path):
    try:
        with open(path, encoding="utf-8", errors="replace") as fh:
            return LAUNCHER_MARKER in fh.read(600)
    except: return False

def cmd_install(args):
    dest = os.path.expanduser("~/bin")
    os.makedirs(dest, exist_ok=True)
    content = open(self_path()).read()
    safe_write_file_atomic(os.path.join(dest, APP), content, perm=0o755)
    for alias, cmd in ALIASES.items():
        os.symlink(APP, os.path.join(dest, alias))
    log_step("Installed", f"Binaries placed in {dest}")
    return 0

def cmd_selftest(args):
    from parts.part4_server_cli import handle_message
    ctx = Ctx(origin="selftest", roots=["/tmp"])
    # Mocking a few calls`ez_mcp` is a single-file, zero-dependency developer toolbox that operates simultaneously as a Model Context Protocol (MCP) server for AI agents (such as Claude Code and Cursor) and as an interactive CLI for environment maintenance and scaffolding.

### Key Architectural Highlights

* **Zero External Dependencies:** Written as a Bash launcher with embedded Python (3.9+ standard library only). No `pip` installs or external packages are required.
* **Single-File Multi-Call Binary:** Symlinking or copying `ez_mcp` to filenames like `ez_doctor`, `ez_scan`, `ez_status`, `ez_scaffold`, or `ez_serve` allows each command to run independently as a standalone binary without code duplication or version drift.
* **Read-Only Safety Model:** MCP calls are strictly confined to authorized root directories (`~/github_projects` and `~/ez_mcp` by default, or customized via `EZ_MCP_ROOTS` / `--root`). Destructive operations or project code execution require launching the server with explicit write access using `--allow-write`.

---

### Command & Tool Capability Matrix

| Command / MCP Tool | Primary Function | MCP Safety Gate |
| --- | --- | --- |
| `doctor` | Checks live DNS, `/etc/resolv.conf`, VPN tunnels, file permissions, and tool states. | Read-only |
| `status` | Summarizes Git repository states (dirty files, ahead/behind branch offsets) across project directories. | Read-only |
| `scan` / `scan_text` | Analyzes text or files for prompt injection vectors, hidden Unicode, dangerous shell calls, and leaked secrets. | Read-only (masks secrets) |
| `scaffold` | Runs a guided wizard to generate DevContainer + Docker Compose + Postgres + MCP server starter kits. | Requires `--allow-write` (forces dry-run otherwise) |
| `fix-dns` / `dns_repair_plan` | Generates or executes automated repair plans for broken or dangling DNS configurations. | Read-only plan (execution via CLI `sudo`) |
| `check` / `run_checks` | Runs test and lint suites (`pytest`, `unittest`, `ruff`) in target project trees. | Requires `--allow-write` |
| `serve` | Launches the stdio MCP server for agent integration. | Configurable via flags |
| `selftest` | Executes end-to-end MCP protocol handshakes and tool gating verification in-process. | Internal testing |

---

### Custom Tool Manifests

You can expose custom shell or Python scripts as both CLI commands and typed MCP tools without modifying `ez_mcp` source code. Defining a JSON schema in `~/.config/ez_mcp/tools.json` provides input validation, environment scrubbing, and execution timeouts automatically:

```json
{
  "tools": [
    {
      "name": "deploy_staging",
      "description": "Trigger a staging deployment script",
      "command": ["/usr/local/bin/deploy.sh"],
      "enabled": true,
      "mutates": true,
      "properties": {
        "target_env": { "type": "string", "enum": ["staging-a", "staging-b"] }
      },
      "required": ["target_env"]
    }
  ]
}

```# ===========================================================================
# ez_mcp core: safety rails and the tool registry
#
# Every capability is a "tool": one registry, three front-ends (MCP server, `ez_mcp call`, friendly CLI).
# Safety model for AI callers (origin == "mcp"):
#   * file paths are confined to allowed roots (EZ_MCP_ROOTS / --root)
#   * tools that write files or run project code need the server started with --allow-write;
#     dry-run-capable tools are silently forced to dry-run without it
#   * external commands run without a shell, with a scrubbed environment and timeouts
# ===========================================================================
APP = "ez_mcp"
PROTOCOL_VERSIONS = ("2025-11-25", "2025-06-18", "2025-03-26", "2024-11-05")
ANSI_RE = re.compile(r"\x1b\[[0-9;?]*[A-Za-z]")
MAX_OUTPUT = 20000
SAFE_ENV_KEYS = (
    "PATH", "HOME", "USER", "LOGNAME", "LANG", "LC_ALL", "TERM", "TZ", "MPD_HOST", "MPD_PORT",
    "XDG_RUNTIME_DIR", "XDG_CONFIG_HOME", "XDG_DATA_HOME", "XDG_CACHE_HOME", "VIRTUAL_ENV",
)


class ToolError(Exception):
    """A tool failed in an expected way; the message is safe to show to the caller."""


@dataclass
class Ctx:
    origin: str = "cli"  # "cli" (a human at a terminal) or "mcp" (an AI client)
    allow_write: bool = True
    roots: tuple = ()


def eprint(*args):
    print(*args, file=sys.stderr, flush=True)


def strip_ansi(text):
    return ANSI_RE.sub("", text)


def truncate(text, limit=MAX_OUTPUT):
    if len(text) <= limit:
        return text
    return text[:limit] + f"\n... [truncated {len(text) - limit} chars]"


def default_roots():
    raw = os.getenv("EZ_MCP_ROOTS")
    parts = [p for p in raw.split(os.pathsep) if p] if raw else ["~/github_projects", "~/ez_mcp"]
    return tuple(os.path.realpath(os.path.expanduser(p)) for p in parts)


def confine(path, ctx):
    """Resolve symlinks and require the result to sit under an allowed root (MCP callers only)."""
    real = os.path.realpath(os.path.expanduser(str(path)))
    if ctx.origin != "mcp":
        return real
    for root in ctx.roots:
        try:
            if os.path.commonpath([real, root]) == root:
                return real
        except ValueError:
            continue
    allowed = ", ".join(ctx.roots) or "none configured"
    raise ToolError(f"path {str(path)!r} is outside the allowed roots ({allowed}); set EZ_MCP_ROOTS or start with --root")


def safe_env(extra=None):
    env = {k: os.environ[k] for k in SAFE_ENV_KEYS if k in os.environ}
    env.update({"GIT_TERMINAL_PROMPT": "0", "GIT_OPTIONAL_LOCKS": "0", "NO_COLOR": "1", "PAGER": "cat", "GIT_PAGER": "cat"})
    env.update(extra or {})
    return env


def run_cmd(argv, cwd=None, timeout=30, env=None, input_text=None):
    """Run argv (never through a shell). Returns (returncode, combined_output)."""
    try:
        proc = subprocess.run(
            argv, cwd=cwd, env=env if env is not None else safe_env(), timeout=timeout,
            input=input_text, stdin=None if input_text is not None else subprocess.DEVNULL,
            capture_output=True, text=True, errors="replace",
        )
    except FileNotFoundError:
        raise ToolError(f"command not found: {argv[0]}")
    except subprocess.TimeoutExpired:
        raise ToolError(f"{argv[0]} timed out after {timeout}s")
    except OSError as exc:
        raise ToolError(f"cannot run {argv[0]}: {exc}")
    return proc.returncode, truncate(strip_ansi((proc.stdout or "") + (proc.stderr or "")))


_TYPES = {"string": str, "integer": int, "boolean": bool, "array": list, "object": dict, "number": (int, float)}


def validate_args(schema, args):
    """Tiny JSON-schema subset: types, enum, min/max, maxLength, required, no unknown keys, defaults."""
    if not isinstance(args, dict):
        raise ToolError("arguments must be a JSON object")
    props = schema.get("properties", {})
    unknown = sorted(set(args) - set(props))
    if unknown:
        raise ToolError(f"unknown argument(s): {', '.join(unknown)}")
    for req in schema.get("required", []):
        if req not in args:
            raise ToolError(f"missing required argument: {req}")
    out = {}
    for key, val in args.items():
        spec = props[key]
        typ = spec.get("type")
        if typ:
            ok = isinstance(val, _TYPES[typ]) and not (typ in ("integer", "number") and isinstance(val, bool))
            if not ok:
                raise ToolError(f"argument {key!r} must be of type {typ}")
        if "enum" in spec and val not in spec["enum"]:
            raise ToolError(f"argument {key!r} must be one of {spec['enum']}")
        if typ in ("integer", "number"):
            if "minimum" in spec and val < spec["minimum"]:
                raise ToolError(f"argument {key!r} must be >= {spec['minimum']}")
            if "maximum" in spec and val > spec["maximum"]:
                raise ToolError(f"argument {key!r} must be <= {spec['maximum']}")
        if typ == "string":
            if len(val) > spec.get("maxLength", 200000):
                raise ToolError(f"argument {key!r} is too long")
            if "\x00" in val:
                raise ToolError(f"argument {key!r} contains a NUL byte")
        out[key] = val
    for key, spec in props.items():
        if key not in out and "default" in spec:
            out[key] = spec["default"]
    return out


TOOLS = {}


def tool(name, description, properties=None, required=(), mutates=False, dry_run_capable=False, source="builtin"):
    """Register a tool. `mutates` = writes files or runs project code (gated for MCP callers)."""

    def deco(fn):
        TOOLS[name] = {
            "name": name,
            "description": description,
            "fn": fn,
            "mutates": mutates,
            "dry_run_capable": dry_run_capable,
            "source": source,
            "schema": {
                "type": "object",
                "properties": properties or {},
                "required": list(required),
                "additionalProperties": False,
            },
            "annotations": {
                "readOnlyHint": not mutates,
                "destructiveHint": False,
                "idempotentHint": not mutates,
                "openWorldHint": False,
            },
        }
        return fn

    return deco


def call_tool(name, args, ctx):
    """Run a registered tool. Returns {"text": str, "isError": bool}. Raises KeyError for unknown tools."""
    spec = TOOLS[name]
    notes = []
    try:
        clean = validate_args(spec["schema"], args if args is not None else {})
        if spec["mutates"] and ctx.origin == "mcp" and not ctx.allow_write:
            if spec["dry_run_capable"]:
                if clean.get("dry_run") is not True:
                    clean["dry_run"] = True
                    notes.append("server was started without --allow-write: forced dry-run, nothing was written")
            else:
                raise ToolError(f"{name} writes files or runs project code; start the server with --allow-write to enable it")
        result = spec["fn"](clean, ctx)
        text = result if isinstance(result, str) else json.dumps(result, indent=2, ensure_ascii=False, default=str)
        is_error = False
    except (ToolError, UsageError) as exc:
        text, is_error = str(exc), True
    except Exception as exc:  # noqa: BLE001 - never crash the server; log details to stderr only
        eprint(f"[{APP}] internal error in {name}:")
        traceback.print_exc()
        text, is_error = f"internal error in {name}: {type(exc).__name__}", True
    if notes:
        text = "\n".join(f"NOTE: {n}" for n in notes) + "\n" + text
    return {"text": truncate(strip_ansi(text)), "isError": is_error}


# ---------------------------------------------------------------------------
# Tool: scaffold_devcontainer (wraps the atomic scaffold engine above)
# ---------------------------------------------------------------------------
@tool(
    "scaffold_devcontainer",
    "Generate a hardened DevContainer + Docker Compose + Postgres + MCP-server workspace (atomic writes, loopback-only "
    "ports, 0600 .env). Dry-run by default; pass dry_run=false to write. Never returns the generated DB password.",
    properties={
        "target_dir": {"type": "string", "description": "Directory to generate into (created if missing)."},
        "project_name": {"type": "string", "description": "Project name (default: directory name)."},
        "python_ver": {"type": "string", "default": "3.12", "description": "Python X.Y."},
        "mcp_port": {"type": "integer", "default": DEFAULT_MCP_PORT, "minimum": 1024, "maximum": 65535},
        "db_port": {"type": "integer", "default": DEFAULT_DB_PORT, "minimum": 0, "maximum": 65535,
                    "description": "Postgres HOST port; 0 = do not publish. Busy ports are advanced automatically."},
        "overwrite_mode": {"type": "string", "enum": list(OVERWRITE_MODES), "default": "backup"},
        "init_git": {"type": "boolean", "default": False},
        "dry_run": {"type": "boolean", "default": True},
    },
    required=["target_dir"],
    mutates=True,
    dry_run_capable=True,
)
def tool_scaffold(args, ctx):
    target = confine(args["target_dir"], ctx)
    ns = SimpleNamespace(
        target_dir=target,
        project_name=args.get("project_name", ""),
        python_ver=args.get("python_ver", "3.12"),
        mcp_port=args.get("mcp_port", DEFAULT_MCP_PORT),
        db_port=args.get("db_port", DEFAULT_DB_PORT),
        init_git=args.get("init_git", False),
        overwrite_mode=args.get("overwrite_mode", "backup"),
        dry_run=args.get("dry_run", True),
    )
    buf = io.StringIO()
    with contextlib.redirect_stdout(buf):
        cfg = resolve_noninteractive(ns)
        code = run(cfg)
    return {
        "ok": code == 0,
        "dry_run": cfg.dry_run,
        "target_dir": cfg.target_dir,
        "project_name": cfg.project_name,
        "mcp_port": cfg.mcp_port,
        "db_host_port": cfg.db_port,
        "secrets": ".env (mode 600); the password is never returned",
        "log": strip_ansi(buf.getvalue()),
    }
