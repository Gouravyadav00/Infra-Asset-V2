# Offline Linux Patch Discovery & Installation (Server-Centric Model)

Everything happens on the server (your Windows machine).\
The agent does NOTHING in discovery. Zero.

------------------------------------------------------------------------

## Server vs Agent Responsibilities

### SERVER (Windows)

    ┌─────────────────────────────────────────────────────────┐
    │                 SERVER (Windows)                         │
    │                                                         │
    │  1. Download Packages.gz from Ubuntu mirrors (HTTP)     │
    │  2. Parse text files (pure Python - no Linux needed)    │
    │  3. Compare with agent inventory (DB query)             │
    │  4. Download .deb files (HTTP - just binary files)      │
    │  5. Store in apt-catalog/                               │
    │                                                         │
    │  Server needs: Internet access + Python                 │
    │  Server does NOT need: Linux, apt, dpkg                 │
    └─────────────────────────────────────────────────────────┘

### AGENT (Ubuntu - OFFLINE)

    ┌─────────────────────────────────────────────────────────┐
    │              AGENT (Ubuntu - OFFLINE)                    │
    │                                                         │
    │  1. Reports installed packages to server (dpkg-query)   │
    │  2. When patch job arrives, installs from server's      │
    │     apt-catalog via HTTP (LAN only, no internet)        │
    │                                                         │
    │  Agent needs: LAN access to server                      │
    │  Agent does NOT need: Internet                          │
    └─────────────────────────────────────────────────────────┘

The Packages.gz files are just gzipped text. The .deb files are just
binary files downloaded via HTTP. No Linux tools needed --- httpx
(Python HTTP library) handles it all on Windows.

------------------------------------------------------------------------

## What about jammy vs noble for the same package?

Yes, they are completely separate. Example with p7zip-full:

### Sync for jammy:

    → Packages.gz says: p7zip-full 16.02+dfsg-8 (jammy)
    → Agent "server-A" (jammy) has: p7zip-full 16.02+dfsg-7
    → Update found! Downloads .deb for jammy

### Sync for noble:

    → Packages.gz says: p7zip-full 16.02+dfsg-10 (noble)
    → Agent "server-B" (noble) has: p7zip-full 16.02+dfsg-8
    → Update found! Downloads different .deb for noble

They get stored in separate paths:

    apt-catalog/
      ubuntu/
        jammy/
          patches/
            p7zip-full/          ← jammy version
              pool/p7zip-full_16.02+dfsg-8_amd64.deb
              dists/jammy/main/binary-amd64/Packages
        noble/
          patches/
            p7zip-full/          ← noble version
              pool/p7zip-full_16.02+dfsg-10_amd64.deb
              dists/noble/main/binary-amd64/Packages

Two separate Patch records in DB:

    - kb_id=LP-p7zip-full, ubuntu_codename=jammy, version 16.02+dfsg-8
    - kb_id=LP-p7zip-full, ubuntu_codename=noble, version 16.02+dfsg-10

When you deploy, each agent gets the correct version for its codename.

------------------------------------------------------------------------

# Human-Readable Roadmap (shareable)

Here's how the same thing could be done manually, step by step, and what
our system automates:

## Manual Process (what a sysadmin would do)

### Step 1 --- Know what's installed

SSH into each Ubuntu server. Run:

    dpkg-query -W -f='${Package}\t${Version}\n'

Copy the output into a spreadsheet. Note which server runs jammy, which
runs noble.

Our system: Agents send this automatically via telemetry. Stored in
software_inventory table with OS codename from os_info.

------------------------------------------------------------------------

### Step 2 --- Check Ubuntu for updates

Open browser. Go to:

    http://archive.ubuntu.com/ubuntu/dists/jammy-security/main/binary-amd64/Packages.gz

Download it. Unzip. Open the text file (it's \~50MB of package
metadata).\
Search for packages from your spreadsheet. Note available versions.\
Repeat for -updates, universe, and for each codename (jammy, noble,
focal).\
That's 6 files per codename, 18 files for 3 codenames.

Our system: "Sync Indices" button does this in \~30 seconds. Downloads
all 18 files, parses them, filters to only your installed packages.

------------------------------------------------------------------------

### Step 3 --- Compare versions

For each package, compare your installed version vs available version.\
Ubuntu versions have special rules: 1:2.3.4-5ubuntu6 --- epoch,
upstream, revision.\
\~ sorts before everything, letters before digits, etc.\
Determine which packages actually have newer versions available.

Our system: debian_version_compare() implements the full Debian version
comparison spec. Runs automatically during sync.

------------------------------------------------------------------------

### Step 4 --- Decide what matters

From hundreds of available updates, decide which to apply.\
Security updates (from -security suite) are usually priority.\
Check how many servers are affected.

Our system: Shows a table with filters. "Security" badge, agent count,
source column. "Select All Security" button for bulk selection.

------------------------------------------------------------------------

### Step 5 --- Download .deb files

For each selected package, construct the download URL:

    http://archive.ubuntu.com/ubuntu/ + Filename field from Packages index

Download the .deb file. Verify SHA256 hash matches.\
Repeat for each codename --- jammy's openssl is different from noble's
openssl.

Our system: "Download Selected" button. Downloads each .deb, verifies
SHA256, stores locally.

------------------------------------------------------------------------

### Step 6 --- Create a local repository

Set up an APT repository structure:

Create pool/ directory, copy .deb into it.\
Generate Packages index file (run dpkg-scanpackages or write it
manually).\
Compress to Packages.gz. Create Release file.\
Host via HTTP server.

Our system: AptRepoService.add_deb() does this automatically --- pure
Python, no dpkg-scanpackages needed. Creates proper APT repo structure
that any apt-get client can consume.

------------------------------------------------------------------------

### Step 7 --- Configure agents and install

SSH into each server. Add source to /etc/apt/sources.list.d/.\
Run apt-get update then apt-get install `<package>`{=html}.\
Repeat for every server, every package.

Our system: Deploy patch from UI → agent auto-configures per-app APT
source → installs from local repo. No SSH, no internet on agent.

------------------------------------------------------------------------

# Architecture Diagram

                              INTERNET
                                 │
                    ┌────────────┴────────────┐
                    │   Ubuntu Mirrors        │
                    │  archive.ubuntu.com     │
                    │  security.ubuntu.com    │
                    └────────────┬────────────┘
                                 │ HTTP (Packages.gz + .deb)
                                 │
                    ┌────────────┴────────────┐
                    │   InfraKnit Server      │
                    │   (Windows, Port 9000)  │
                    │                         │
                    │  ┌───────────────────┐  │
                    │  │ Patch Discovery   │  │
                    │  │ Service           │  │
                    │  │                   │  │
                    │  │ • Parse indices   │  │
                    │  │ • Compare versions│  │
                    │  │ • Download .deb   │  │
                    │  │ • Build APT repo  │  │
                    │  └───────┬───────────┘  │
                    │          │              │
                    │  ┌───────┴───────────┐  │
                    │  │ apt-catalog/      │  │
                    │  │  ubuntu/          │  │
                    │  │   jammy/patches/  │  │
                    │  │   noble/patches/  │  │
                    │  │   focal/patches/  │  │
                    │  └───────┬───────────┘  │
                    │          │ Served via   │
                    │          │ HTTP /apt-   │
                    │          │ catalog/     │
                    └──────────┼──────────────┘
                               │
                    ┌──────────┴──────────┐
                    │     LAN (no internet needed)
                    │                     │
           ┌────────┴──────┐    ┌────────┴──────┐
           │ Agent (jammy) │    │ Agent (noble) │
           │               │    │               │
           │ apt-get install│   │ apt-get install│
           │ from server's │    │ from server's │
           │ apt-catalog   │    │ apt-catalog   │
           └────────────────┘    └────────────────┘

The key insight: the server is just an HTTP download proxy + APT repo
builder. It doesn't need Linux. It downloads files over HTTP and serves
them over HTTP. The agents are the only ones that need apt-get.
