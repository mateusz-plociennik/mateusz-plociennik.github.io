---
title: "How I'm building a Christmas light show with AI agents"
date: 2026-11-17
draft: false
summary: "Two and a half weeks from \"I have no idea about controllers\" to hardware on order and a live show page: the history, the research approach and where the project stands."
tags: [christmas-lights, ai, wled, diy]
ShowToc: true
---

This year I decided to do a proper Christmas light display: addressable pixels, effects, eventually a show synced to music. I'm a software developer and an electronics hobbyist, but I'd never built anything like this. My first message to an AI agent included the sentence "I have no idea about controllers."

Two and a half weeks later, the power supplies, a controller and the pixels are ordered, and there's a public show page at [mattyslights.remotefalcon.com](https://mattyslights.remotefalcon.com). The code and research notes are in [the project repo](https://github.com/mateusz-plociennik/christmas-lights). <!-- TODO verify: repo is public before publishing -->

I want to be upfront about one thing: **this was AI-assisted work from the first day, and it most likely wouldn't have happened without AI.** Not because the problem is hard, but because the research would have taken more evenings than I have. This post covers how it went, how I work with the agents, and where things stand.

## How it started

On 1 November I gave Claude Code two photos of the house and asked for a DIY addressable-light design. I pointed it at [auschristmaslighting.com](https://auschristmaslighting.com), the Australian community forum, and asked for Australian suppliers, budget options, renders and an HTML document with pricing.

About fifteen minutes later I had a plan with lights drawn over the photos and three tiers: a budget one around 400 pixels on an ESP32 running [WLED](https://kno.wled.ge/), a mid tier with a dedicated controller and xLights, and a premium tier I was never going to buy. It also told me to buy Mean Well power supplies from a local shop rather than cheap unbranded ones, which stuck with me.

It wasn't perfect (about half the prices were estimates because shops blocked lookups), but it turned "where do I start" into a list of things to check.

## A week of narrowing down

The next sessions were in GitHub Copilot CLI. I didn't remember which tool I'd used the week before, so I asked the new agent to dig through my previous session history. It found the Claude Code conversation on disk and rebuilt the context from it. That's when I realised old AI sessions are research I can recover, not throwaway chats.

The plan changed several times:

- **Icicles, then not.** I wanted dense, per-pixel icicles along the roofline as a matrix. The agent read the seller's dimension image and pointed out the "dense" icicle was 15 cm between columns and 2.5 cm between pixels vertically: a shallow, stretched grid, not square pixels. That reshaped the design before I spent anything. Icicles also turned out to be expensive, so I switched to linear strings.
- **5V, then 24V.** The first string comparison defaulted to 5V. I asked "why 5V? I prefer 12V or 24V", and the agent agreed it should have prioritised the higher voltage. Higher voltage means less current for the same power, which matters over a long roofline. 24V seed strings also came out cheaper per pixel in that comparison.
- **Controller.** I asked about WLED versus a dedicated pixel controller and what forum members actually report. The answer: a WLED board *is* a pixel controller, just running open firmware, and WLED can later take data from xLights, so starting cheap doesn't mean throwing it away later. For a budget Wi-Fi setup, Gledopto ESP32 boards running WLED ended up on the shortlist.
- **Power.** I compared an RGBW and an RGB 24V seed listing and spotted something odd: the RGBW one said 0.33 W per pixel, but also 3 mA for RGB and 5 mA for white. The agent worked it through (three colour channels plus white at 24V comes to about 0.336 W) and revised its own power supply advice upwards, saying its earlier example would have been too small.

## How I work with the agents

Looking back, a few habits did most of the work.

**Start broad, with the community as the source.** I asked for tiers and options, not one answer, and pointed the agent at the forum where Australian hobbyists post real experience. Later I asked specifically for "user opinions", and the notes keep those separate from manufacturer claims.

**Push back.** "Why 5V?", "is that all?", "it says 0.33 W but then 3 mA…?" Short questions like these changed the answer more than once. The agents are good at research, but they settle on defaults, and they don't know my preferences unless I say them.

**Ask for receipts.** The research notes record sources with the date they were checked and what they don't prove. Prices are labelled by where they came from: a search result card, the selected product variant, or checkout. Owner measurements, anecdotes and manufacturer claims are kept apart. When the second agent recovered the first session, it also wrote a list of corrections to the earlier advice instead of inheriting it.

**Change the tool when the web blocks you.** AliExpress kept throwing CAPTCHAs at the agent, so I asked whether it could use my browser instead. With the Playwright browser extension it drove my logged-in Chrome, selected the actual product variants and read the real prices and "Sold By" lines across two stores. Hours of clicking I didn't have to do.

**Write everything down, in the repo.** Each session ends with notes for the next agent: a handoff document, and later a small markdown wiki in `docs/wiki/`. Its one rule is to keep dated evidence separate from recommendations and update an existing page rather than duplicate it. A new session starts by reading the previous session's notes. The agents don't remember anything between sessions, so the wiki does.

**Keep sessions short.** One narrow goal per session (suppliers, controllers, power, the show page) makes the result easier to check.

## The show page

[Remote Falcon](https://remotefalcon.com) is a free service that gives a light show a public page where visitors can request songs or vote. I wanted the page to look like the house, lit up.

The first drawings the agent made looked, in my words, "very 90s… like paint". I used an annotation tool to click on parts of the rendered page and leave notes ("roof doesn't stick to the wall", "palm trees are trimmed from the top"), plus a site plan. The agent eventually rebuilt the scene in real metres through a single camera with a proper vanishing point. That took a couple of hours of feedback, and it was also where the design firmed up.

Then the live page looked different from my local copy. Remote Falcon renders custom pages through a library that lowercases tag names, which breaks SVG gradients. Moving the drawings to image files fixed that, until the embedded images were stripped on save. So the agent read Remote Falcon's open-source UI, found its API and wrote a deploy script that uploads the images and saves the page in one command. On my own, a weekend would most likely have ended at "the images don't show".

## Where it stands now

The design, as drawn on the show page:

- the first-floor roofline in warm white;
- the ground floor as a single line of RGBW seed pixels following the wall; <!-- TODO verify: RGBW still the plan for the ground floor -->
- three palm trees wrapped in lights;
- a mega tree on the lawn, planned, with placement still to decide.

Ordered today:

- **2× Mean Well XLG-200-24-A**, 24V 200 W LED power supplies;
- **a GLEDOPTO controller**, the WLED-based Wi-Fi option from the shortlist; <!-- TODO verify: exact model and quantity; "WLED / Wi-Fi" is inferred -->
- **Ray Wu pixels**, 24V seed strings. <!-- TODO verify: which strings (RGB vs RGBW), how many pixels -->

The plan is to start with WLED effects and grow into a sequenced show with xLights and Falcon Player (FPP). Two 200 W supplies leave headroom over the roughly 200 W the agent estimated for 600 RGBW pixels at full white, though I'll set the real numbers from measurements. <!-- TODO verify: how the two PSUs will be split across strings/props -->

Nothing has been tested yet. Everything so far is research, a drawing and a shopping cart.

## What's next

- **Test every roll on arrival.** A single-pixel chase to check every pixel is individually addressable and in the right order. Forum owners report batch variance and mislabelled rolls.
- **Check WLED support for the RGBW pixels.** The chip on the RGBW listing isn't in WLED's LED type table, so colour order and white handling need testing. <!-- TODO verify: only relevant if the order includes the RGBW strings -->
- **Measure, then wire.** Full-white current and voltage at the far end of each string, then decide power injection and fusing from the measurements rather than rules of thumb.
- **The mains side.** The XLG supplies have bare mains leads. The project rule is no DIY mains work, so they get a compliant connection done by a licensed electrician or I use a different supply. <!-- TODO verify: how you're handling the mains side -->
- **Delivery.** AliExpress delivery estimates aren't promises, and December is close.
- **Weatherproofing.** An enclosure for the controller and power distribution.
- **The show itself.** Set up an FPP host, lay out the house in xLights, sequence some songs, and connect it all to Remote Falcon. The show is planned for December. <!-- TODO verify: dates, hours, FM radio -->
- **The mega tree.** Placement and parts.

I'll write again once the hardware arrives and I've measured something real.

## Why I'm sharing this

The interesting part isn't that an AI can find LED strings. It's that a hobby project I'd have put off for another year went from zero to hardware ordered in a couple of weeks of evenings, with research I trust enough to spend money on. That trust came from the process: pushing back, asking for sources, checking the arithmetic and keeping a written record. <!-- TODO verify: "put off for another year" -->

If you want the details, the full research notes, wiki and show page source are in [the repo](https://github.com/mateusz-plociennik/christmas-lights).
