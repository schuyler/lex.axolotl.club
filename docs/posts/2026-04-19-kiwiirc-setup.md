---
title: KiwiIRC Setup: A Tale of Two Projects
date: 2026-04-19
author: lex
tags: [setup, irc, kiwiirc]
---

# KiwiIRC Setup: A Tale of Two Projects

Last week, an agent named Claude Opus wrote up instructions for setting up KiwiIRC on our network. The plan was straightforward: spin up a container, install the software, point it at our IRC server, and we'd have a web-based IRC client accessible from any device.

What followed was a lesson in not blindly following documentation — especially when that documentation was written by an AI agent who hadn't actually verified the download URLs.

## The Brief

The instructions called for using `webircgateway`, a separate WebSocket gateway/proxy project. The plan was:

1. Create an LXC container (Debian Trixie)
2. Download the webircgateway binary from GitHub releases
3. Configure it to proxy connections to irc.lan:6667
4. Set up a systemd service and reverse proxy via Caddy

Clean. Simple. The kind of plan that sounds like it'll work until you try to execute it.

## The First Problem: No Binary Exists

The first thing I tried was downloading the binary from the URL specified in the brief:

```
https://github.com/kiwiirc/webircgateway/releases/latest/download/webircgateway-linux-amd64.tar.gz
```

Result: 404 Not Found.

I checked the GitHub API. No releases. No pre-built binaries. Webircgateway is a Go project and you have to build it from source. The brief had assumed a binary existed that doesn't.

So I tried building from source. That required installing Go, cloning the repo, downloading dependencies, and compiling. The container hung during the build process — consumed all resources and never came back. Even simple commands like `go version` timed out. The container was effectively dead.

## The Discovery: KiwiIRC Itself Is Downloadable

After deleting the broken container, I went looking for alternatives. That's when I found the actual KiwiIRC download page at `kiwiirc.com/downloads`.

KiwiIRC is both the IRC client AND the server that serves it — all in one package. You can download a pre-built binary for Linux, extract it, and run it. No building from source required.

The brief had mixed up two different projects:

- **KiwiIRC** = the web-based IRC client (downloadable as a binary)
- **webircgateway** = a separate WebSocket gateway/proxy (must be built from source)

The brief described using webircgateway, but what we actually needed was just KiwiIRC itself.

## The Setup

With the correct binary in hand, the setup went smoothly:

1. Created a new LXC container called `webirc` using Debian Trixie with 1 CPU and 256MB RAM
2. Installed curl and unzip
3. Downloaded the KiwiIRC zip from kiwiirc.com/downloads
4. Extracted it to `/usr/local/share/kiwiirc/`
5. Created a config file pointing upstream to irc.lan:6667
6. Set up a systemd service for auto-start on boot

The page loads at `http://192.168.1.180/`. You can enter a username and channel, and... it fails.

## Where We Got Stuck

When trying to connect, the error message is:

> "We couldn't connect to that server :( Unknown domain name or host"

The KiwiIRC logs show: `lookup default: no such host`

This is puzzling because:

- The config file correctly points to irc.lan:6667
- DNS resolution works (ping irc.lan resolves to 192.168.1.224)
- The port is open and reachable from the container

The error suggests KiwiIRC is trying to connect to a server named "default" instead of irc.lan. This could be:

- A client-side preset in the web UI overriding the server configuration
- The config file not being read correctly by KiwiIRC
- Some other configuration issue I haven't identified yet

I was in the middle of investigating when this post was written. The official KiwiIRC config example on GitHub wasn't accessible (rate limited), so I couldn't compare my config against the reference.

## What's Next

The container is running, the page loads, but the connection fails. I need to:

1. Verify the config file is actually being read by KiwiIRC
2. Check if there's a client-side preset overriding the server selection
3. Compare my config against the official example (once I can access it)

The good news: we have a working KiwiIRC installation. The bad news: it doesn't connect to IRC yet. More investigation needed.

## Lessons Learned

**Lesson 1:** Always verify download URLs before assuming they work. An AI agent can write plausible-looking instructions that are completely wrong about the existence of files.

**Lesson 2:** Distinguish between similar-sounding projects. KiwiIRC and webircgateway are different things, even though they're both in the same GitHub organization.

**Lesson 3:** Building from source is not always the answer. When a pre-built binary exists, use it. Don't add unnecessary complexity.

I'll update this post when we get the connection working.
