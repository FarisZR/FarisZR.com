---
title: My Projects
slug: "projects"
comments: false

links:
  - title: More Projects

  - title: agentbox-mcp
    description: A Rust MCP server that gives chat UIs such as ChatGPT tools for running commands, managing persistent terminal sessions, applying patches, loading skills, and proxying local MCP servers on a real Linux machine.
    image: github.webp
    website: https://github.com/FarisZR/agentbox-mcp

  - title: whisper-stt-gnome-extension
    description: A GNOME Shell extension for one-key speech-to-text through OpenAI-compatible transcription APIs, with a live voice graph and clipboard output.
    image: github.webp
    website: https://github.com/FarisZR/whisper-stt-gnome-extension

  - title: Privacy-OCI
    description: Automatically builds and publishes container images for privacy-respecting alternative frontends such as Quetre, Scribe, BreezeWiki, SimplyTranslate, whisper.cpp, and StickTock, checking upstream projects for updates every hour.
    image: github.webp
    website: https://github.com/FarisZR/Privacy-OCI

  - title: moodle-cloud-sync
    description: A self-hosted web app that syncs selected DHBW Moodle course files to Google Drive on demand or on a schedule, with detailed sync logs and error handling.
    image: github.webp
    website: https://github.com/FarisZR/moodle-cloud-sync
---

## Knocker, a knock-based access control service for your homelab

[![](https://github.com/FarisZR/knocker/raw/main/assets/knocker-banner.webp)](https://github.com/FarisZR/knocker)

Knocker is a self-hosted, knock-based access control service for homelabs. An authenticated request temporarily whitelists your IP, letting you keep services private without requiring a VPN client on every device.

It can protect HTTP services through a reverse proxy such as Caddy, or enforce access at the firewall level through its FirewallD integration. It also supports IPv4 and IPv6, configurable per-key TTLs, remote IP/CIDR whitelisting, and clients for the web, CLI, GNOME, and Android.

[Here's my launch blog post about it](https://fariszr.com/knocker-access-your-homelab-without-vpn/).

## DualMate

[![](https://dualmate.fariszr.com/dualmate-hero.jpg)](https://dualmate.fariszr.com)

DualMate is an Android app for DHBW students. It's an all-in-one companion for checking your schedule, canteen menus, grades, exams, and other important dates without jumping between different portals.

The app also supports offline timetable access, course details such as rooms and lecturers, meal filters, Dualis modules and credits, and home-screen widgets for upcoming classes and canteen meals.

I revived the app from an abandoned fork, added the new canteen and important-dates features, and rebuilt the entire schedule page.

[DualMate is also available on Google Play](https://play.google.com/store/apps/details?id=com.fariszr.dualmate).

## The Aosus Community

[Aosus](https://aosus.org) is the largest Arabic community for free and open-source software. Its goal is to spread FOSS in the Arab world and build a stronger culture around contributing to open-source projects.

[![](https://aosus.org/opengraph.jpg)](https://aosus.org)

The community now has more than 3,000 members, 10,000 posts, and 1,300 topics.

I joined Aosus in 2021 to work on a new relaunch of the project, starting with the merger of Aosus's Telegram community and another GNU/Linux group with more than 3,000 members.

I then worked on redesigning the community page and launching the new official website and blog. This also came with a complete overhaul of Aosus's technical infrastructure, which is now fully Dockerized and uses AI-assisted automation to self-heal parts of the infrastructure.

I also managed the [Aosus Writing Award](https://aosus.org/writing-contest) ([English summary](https://opencollective.com/aosus/projects/aosus-writing-contest)), the first award of its kind in the Arab world to support writers producing Arabic content about FOSS.

You can find ways to support Aosus [here](https://aosus.org/en/support-us), and you can contribute to Aosus's [projects on GitHub](https://github.com/aosus).

### My Achievements in Aosus

(Unfortunately, most of the links are only available in Arabic. It is an Arabic-focused project, after all.)

- Helped move Aosus to The Hack Foundation (Hack Club) as its fiscal sponsor, giving the project U.S. 501(c)(3) fiscal sponsorship and enabling tax-deductible donations in the U.S.
- [Created a page about the community and its projects](https://aosus.org/en).
- [Created a Matrix server for Aosus](https://aosus.org/931).
- Redesigned the Discourse forum; you can see older versions on the [Internet Archive](https://web.archive.org/web/*/aosus.org).
- Launched the [Aosus Writing Award](https://aosus.org/924), the first award of its kind in the Arab world.
- Helped [Aosus join Open Collective](https://aosus.org/1359), making it one of the first Arab projects on the platform and improving the project's financial transparency and independence.

## ARhackintosh

[ARhackintosh](https://هاكنتوش.com) is the first complete Arabic community for Hackintosh, with an open-source installation guide based on Dortania and a dedicated Kext Archive.

**If you know Arabic, ARhackintosh needs new maintainers. If you're interested, you can find more details [here](https://هاكنتوش.com/هاكنتوش-بالعربي-يبحث-عن-مساهمين-جدد/).**

[![](https://xn--mgbg4a8cpdl.com/wp-content/uploads/2022/09/link-preview.jpeg)](https://هاكنتوش.com)

(Yes, that's an actual wordmark. People can read it.)

I started ARhackintosh in 2020 to create a dedicated place for all things Hackintosh in Arabic.

It played a major role in helping me discover my interest in Linux and technical infrastructure, and it introduced me to Git and the wider FOSS ecosystem.

### ARtutorial

The biggest part of the project, and the one where I spent most of my development time, is [ARtutorial](https://tutorial.هاكنتوش.com).

With more than 4,000 words and 1,250+ commits, it is the largest Arabic guide to installing Hackintosh. It is completely [open source](https://github.com/ARhackintosh/ARtutorial).

I tried to keep the guide approachable without hiding the real complexity of installing a Hackintosh. It is genuinely complex, and hiding that complexity often leads to lost data or wasted time.

### Kext Archive

The Kext Archive collects crucial Kexts (kernel extensions, essentially macOS drivers), with direct links to developer repositories and downloads, handwritten descriptions in Arabic, and clear categories.

[![](https://xn--mgbg4a8cpdl.com/wp-content/uploads/2021/08/image-1536x870.jpg.webp)](https://xn--mgbg4a8cpdl.com/kextarchive/)

To my knowledge, it is still fairly unique; I have not seen a directly comparable archive in English.

The archive aims to remove the hassle of finding drivers and determining whether a download is up to date. Many Arabic Hackintosh resources are static video guides that cannot easily be corrected or updated after publication.

The Kext Archive provides a single place where users can find both common and more advanced Kexts, read what they do, and follow direct links to their upstream projects and current downloads. It now lists more than 50 drivers.