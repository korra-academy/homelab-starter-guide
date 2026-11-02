# Homelab Starter Guide — Your First Homelab on an Old Laptop

A free, step-by-step **homelab starter guide**: turn an old laptop into a Linux server, learn SSH, run your first Docker containers, and work through networking labs that map directly onto the skills IT certifications test. No new hardware, no cloud bill.

Maintained by [Korra Academy](https://korra.academy/) — the team behind **[Korra.Training](https://korra.training/)**, live certification prep with human instructors.

## Why a homelab

Reading about DNS is not the same as breaking DNS at 11pm and fixing it. A homelab is where the words in a CompTIA or Cisco study guide turn into things you have actually done. Hiring managers for service desk, junior sysadmin and junior network roles ask "what have you built?" — this is your answer.

## What you need

- **An old laptop** (any 64-bit machine from the last ten years). 8 GB RAM is comfortable; 4 GB works for everything in this guide. A laptop beats a desktop for a first lab: built-in screen, keyboard and a battery that doubles as a UPS.
- **A USB stick** (4 GB or more) to install from.
- **An Ethernet cable** to your router. Wi-Fi works, but wired is faster and far less confusing when you start doing networking labs.
- **Your everyday computer** to connect from.
- **A notebook** — paper or Markdown. See [Keep a lab journal](#keep-a-lab-journal).

## Step 1 — Install Linux

Use **Ubuntu Server LTS** or **Debian**. Both are free, widely documented, and what most tutorials and job adverts assume. Pick one and stay on it for your first three months.

1. Download the ISO and write it to the USB stick with [balenaEtcher](https://etcher.balena.io/) or [Rufus](https://rufus.ie/) (Windows).
2. Boot the laptop from USB (usually F12, F2, Esc or Del at power-on to pick the boot device).
3. Install with defaults. Say **yes** to installing the OpenSSH server when asked. Create a user with a strong password — not `admin`/`admin`.
4. After first login, update everything:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
5. Stop the laptop sleeping when you close the lid. Edit `/etc/systemd/logind.conf`, set `HandleLidSwitch=ignore`, then `sudo systemctl restart systemd-logind`.
6. Give it a fixed address. Easiest way: log into your router and create a **DHCP reservation** for the laptop's MAC address, so it always gets the same IP. Write that IP in your journal.

## Step 2 — SSH in and stop using the laptop's keyboard

From your everyday machine (Windows PowerShell, macOS Terminal or Linux shell):

```bash
ssh yourname@192.168.1.50
```

Then make it key-based, which is both safer and quicker:

```bash
ssh-keygen -t ed25519
ssh-copy-id yourname@192.168.1.50
```

On Windows, `ssh-copy-id` is missing: paste the contents of your `.pub` file into `~/.ssh/authorized_keys` on the server instead. Once keys work, disable password logins on the server (`PasswordAuthentication no` in `/etc/ssh/sshd_config`, then `sudo systemctl restart ssh`). Close the laptop lid. From here on it lives on a shelf.

## Step 3 — Docker and your first three containers

Docker lets you run services as isolated, disposable containers instead of installing them straight onto the OS. Install it following the [official Docker Engine docs for your distro](https://docs.docker.com/engine/install/), then add yourself to the `docker` group so you don't need `sudo` every time.

Start with these three, in this order:

| # | Container | What it teaches | Docs |
|---|---|---|---|
| 1 | **Uptime Kuma** | Ports, volumes, a web UI on a non-standard port, monitoring basics | [github.com/louislam/uptime-kuma](https://github.com/louislam/uptime-kuma) |
| 2 | **Pi-hole** | DNS end to end: how lookups work, what a resolver is, why ad domains vanish | [docs.pi-hole.net](https://docs.pi-hole.net/) |
| 3 | **Nginx** serving a static page | Web servers, reverse proxies, HTTP status codes, config files | [hub.docker.com/_/nginx](https://hub.docker.com/_/nginx) |

Write each one as a `compose.yaml` file in its own folder under `~/lab/`. Bring them up with `docker compose up -d`, check with `docker ps`, read logs with `docker logs <name>`. Then — deliberately — delete one and bring it back. Disposability is the point.

## Step 4 — Networking labs

Each lab maps to a CompTIA Network+ or Security+ objective. Do them in order; each builds on the last.

1. **Map your network.** Install `nmap` on the server and scan your own LAN (`nmap -sn 192.168.1.0/24`). Identify every device. Only ever scan networks you own.
2. **Subnet it on paper.** Take your LAN's address and CIDR, work out the network address, broadcast, first and last host and host count by hand. Check your answer with the included [subnet-calculator.html](subnet-calculator.html) — one file, works offline.
3. **Point one device at Pi-hole for DNS.** Watch the query log. Then break it: stop the container and see exactly how "the internet is down" looks when it's really DNS.
4. **Open a port, then close it.** Install `ufw`, allow only SSH and the ports your containers use, and prove it from another machine with `nmap`.
5. **Capture traffic.** Install `tcpdump` on the server, capture a DNS lookup and an HTTP request, and read the packets. Open the capture in Wireshark on your main machine.
6. **Put a service behind a reverse proxy.** Make Nginx forward `/kuma` to Uptime Kuma. This is how real deployments expose many services on one port.
7. **Back up and restore.** Tar the `~/lab/` folder and the Docker volumes to a USB drive, wipe a container, restore it. Untested backups don't exist.

## What to learn in what order

| Month | Focus | Maps to |
|---|---|---|
| 1 | Linux basics: files, permissions, `systemctl`, `journalctl`, package manager | CompTIA Linux+, A+ (OS objectives) |
| 2 | Docker and the three containers above | Day-one skills for junior sysadmin / DevOps |
| 3 | Networking labs 1–5 | Network+, CCNA (IP addressing, DNS, ports) |
| 4 | Firewall, reverse proxy, backups, log reading | Security+, Linux+ |
| 5+ | Pick one: a second node, a VPN (WireGuard), or a small Kubernetes (k3s) cluster | Cloud and platform roles |

## Keep a lab journal

For every session, write three lines: what you tried, what broke, what fixed it. After a month you have a troubleshooting log you can talk through in an interview, and a personal wiki you will actually use. A GitHub repo of Markdown files is ideal — it's also a portfolio.

## Common mistakes

- **Buying hardware first.** Rack servers are loud, hot and expensive. Everything above runs on a laptop worth nothing.
- **Installing services straight onto the OS.** Use containers. You will want to delete things.
- **Exposing the lab to the internet on day one.** Keep it on your LAN until you understand firewalls. When you need remote access, use a VPN such as WireGuard or Tailscale, not port forwarding.
- **Skipping the journal.** Six weeks later you won't remember why DNS stopped working.
- **Ten projects at 10%.** One container working and understood beats a dashboard of half-configured apps.

## FAQ

**What is a homelab?**
A homelab is a small computing setup at home used to learn and practise IT skills: running servers, networking, virtualisation and security tools on hardware you control. It can be a single old laptop or a rack of servers; the learning value comes from building and breaking things yourself.

**Can I use a Raspberry Pi instead of a laptop?**
Yes. A Raspberry Pi 4 or 5 runs everything in this guide. A laptop is usually free if you already own one and has more RAM and storage, which matters once you run several containers.

**Which Linux distribution is best for a homelab?**
Ubuntu Server LTS or Debian. Both are stable, free, and most documentation assumes one of them. Distribution choice matters far less than consistency — pick one and learn it properly.

**Do I need Docker for a homelab?**
Not strictly, but it is the standard way to run self-hosted services and it makes experiments disposable. Learning Docker early also directly matches what junior infrastructure and DevOps job adverts ask for.

**Is a homelab useful for CompTIA certifications?**
Yes. Network+, Security+ and Linux+ all test hands-on concepts — subnetting, DNS, firewalls, permissions, logs — that are far easier to remember once you have done them on a real machine. See the mapping table above and our [it-certification-roadmap](https://github.com/korra-academy/it-certification-roadmap).

**Does running a homelab cost much in electricity?**
A laptop with the lid closed typically draws 10–20 watts, roughly the same as an LED light bulb left on. Far less than a desktop or a rack server.

**Is it safe to expose my homelab to the internet?**
Not until you understand firewalls and authentication. Keep it on your home network, and use a VPN (WireGuard or Tailscale) for remote access rather than forwarding ports on your router.

---

### About Korra Academy

[Korra Academy](https://korra.academy/) runs live online training across eight doors: [Kids](https://korra.kids/), [Study](https://korra.study/), [Career](https://korra.career/), [Business](https://korra.business/), [Tutor](https://korra.one/), [Training](https://korra.training/), [Bot](https://korra.bot/) and [Codes](https://korra.codes/). Preparing for one of these exams with a real instructor beats grinding alone — start at [korra.training](https://korra.training/). MIT licensed.
