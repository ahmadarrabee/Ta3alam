# Deployment Specification

Baseline for the Northern Jordan hub, 2025 academic term. This is the intended deployment specification; verify actual hardware, installed versions, paths, and content on site. Cost estimates below are team planning figures, not current retail quotations.

## Hardware and indicative costs

| Component | Specification | Indicative cost |
| --- | --- | --- |
| Server | Repurposed laptop running Ubuntu; battery provides brief power resilience | Existing/reused hardware; no purchase price supplied |
| Router | Dual-band Wi-Fi router, Ethernet connection to server | Approximately 25 JOD |
| Storage | 512GB SSD plus 128GB USB backup drive | Approximately 15 JOD combined, as supplied by the project |
| Client devices | Shared laptops/tablets with a modern browser and Web Worker support | Existing/shared; no price supplied |

Indicative listed purchases total approximately 40 JOD, excluding unspecified costs. A 128GB USB cannot hold a full 512GB disk image: size actual Kolibri data and retained full snapshots before deployment, and provide additional/rotated drives as necessary.

## Software stack and version record

| Layer | Baseline | Verification |
| --- | --- | --- |
| Operating system | Ubuntu 22.04 LTS | `lsb_release -ds` |
| Learning service | Kolibri v0.14+; select a supported release compatible with deployment | Device > Info; `kolibri --version` using the installed environment |
| Scripting/runtime | Python 3.10+ | `python3 --version` |
| Portal and lab | Repository HTML/CSS/JavaScript; lab uses JavaScript Web Workers | Record deployed Git commit and browser version |
| Maintenance | Python standard library; Bash, systemd, `flock`, cron | Confirm installed utilities and configured root crontab |

These are baseline versions, not a claim that exact patch versions have been audited. Record exact Ubuntu, Kolibri, Python, browser, and maintenance package versions locally at commissioning and after updates. Kolibri content and packages are installed separately. See the README installation instructions and [operations](OPERATIONS.md).

## Network and services

- Router LAN: `192.168.1.0/24`; reserve or statically configure server IP `192.168.1.100`, outside conflicting DHCP assignments.
- Server connects by Ethernet. Learner devices join the local Wi-Fi. Disable client/AP isolation so they can reach the server.
- Disconnect WAN/internet during learning sessions; disable port forwarding, cloud sync, and external analytics. Protect router administration credentials.
- Portal: TCP 80, `http://192.168.1.100/custom_interface/index.html`.
- Kolibri: TCP 8080, `http://192.168.1.100:8080`.
- Permit these ports from the local learning subnet only. Keep data/USB backups outside the HTTP document root. These local HTTP services require a trusted, controlled LAN.

The README's Python HTTP server serves the repository root for supervised use so the sibling coding-lab link works. Use a dedicated static server and restricted document root for unattended operation. The portal heartbeat does not check Kolibri or internet access.

## Arabic educational channels

The project describes locally installed Arabic lessons, videos, exercises, and quizzes, emphasizing mathematics and science. The repository does not bundle channels or provide verified channel titles, IDs, or revisions. Do not treat this overview as a confirmed channel inventory. The digital-library portal card is currently a “Coming Soon” placeholder.

Before opening a session, the educational facilitator verifies available Arabic channels in Kolibri and records each channel's title, ID, version, language, subject/level, license, import date, and sample offline playback result in a local inventory. Import approved content before deployment or via prepared USB media. Confirm files are present locally, not merely catalog entries. The separate coding lab runs JavaScript despite its retained `python_intro.html` filename.

## Commissioning checks

Test Wi-Fi access on two client devices, both HTTP URLs, learner sign-in/progress, an Arabic lesson, and the coding lab without WAN or mobile data. Verify the actual Kolibri data path and service identity, complete a manual backup and isolated restore, set server time to `Asia/Amman`, and check daily 02:00 cron logging. Document results and responsible staff locally before relying on unattended backups.
