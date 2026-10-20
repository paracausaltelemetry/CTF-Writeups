---
title: Letter
summary: TryHackMe OSINT room using a damaged envelope, handwritten note, and newspaper clipping to identify a historic Penmarc'h rescue figure.
date: 2026-07-01
tags: [TryHackMe, OSINT, GEOINT, Historical Research, Image Analysis]
difficulty: easy
os: N/A
url: https://tryhackme.com/room/letter
---

# CTF Room: Letter
- [Link to room](https://tryhackme.com/room/letter)
- **Difficulty:** Easy
- **Category:** OSINT, Historical Research, Image Analysis
- **OS:** N/A

## 1. Brief
Letter is a TryHackMe OSINT room built around a damaged envelope, a newspaper clipping, and a personal note.

The task files included:

```text
_MACOSX/
letter.png
Newspaper_clipping.png
Note.txt
```

The goal was to identify the delivery postcode and recover the full name and age of the person referenced by the note.

## 2. Envelope Clues
Starting with `letter.png`, the visible clues were limited but useful:

- The stamp/label showed `Lettre Verte`, pointing to France.
- The recipient line looked like it began with `Edouard G`.
- `SNSM` was visible underneath, suggesting a French maritime rescue connection.

That made a French maritime or lifeboat-related investigation likely from the beginning.

## 3. Note Translation
The note was addressed to Edouard. In English, the useful meaning was:

```text
The sender found an old newspaper clipping in their grandparents' attic.
Edouard's great-grandfather was not even old enough to get a driving licence when he distinguished himself.
He was the youngest member of the team, and clearly courageous.
The sender says he would be proud to see Edouard on the water too.
```

The key OSINT clues were:

- A young great-grandfather
- Courage or distinction
- Water or maritime activity
- A family connection to Edouard

## 4. Newspaper Clipping
Using image search on `Newspaper_clipping.png`, I identified the newspaper as `L'Ouest-Eclair`, a regional French newspaper associated with Rennes.

The relevant archived issue was available on Gallica:

```text
https://gallica.bnf.fr/ark:/12148/bpt6k4965688
```

The clipping matched reporting about a maritime disaster off the coast of Finistere near Penmarc'h in 1925. The article described two fishing boats in distress, lifeboats launched to rescue them, and a major loss of life.

The important detail for the room was not the full article text, but the location and event: the Penmarc'h disaster of 23 May 1925.

## 5. Delivery Postcode

#### What is the postal code of the delivery address on the envelope?
Once Penmarc'h became the likely delivery area, the postcode was straightforward to confirm.

### Answer
```text
29760
```

## 6. Identifying The Person
Researching the Penmarc'h disaster led to local-history pages about the event, including:

```text
https://kbcpenmarch.franceserv.com/la-catastrophe-du-23-mai-1925-selon-la-presse-locale.html
```

The note said the ancestor was the youngest member of the team and was not old enough to drive. Searching around the 1925 Penmarc'h lifeboat disaster surfaced Yves-Marie Gourlaouen, a 15-year-old cabin boy connected to the rescue effort.

This matched the note's clues:

- He was young enough for the driving-licence clue to make sense.
- He was connected to the maritime rescue story.
- He was distinguished for bravery.

## 7. Flag

#### What is the flag?
The required format used the person's name, surname, and age, with only the first letter of the name and surname capitalised.

### Answer
```text
||THM{Yves-Marie_**************||
```

## 8. Summary
The envelope established a French maritime context through `Lettre Verte` and `SNSM`. The note pointed to a young, courageous ancestor connected to the water.

Image searching the newspaper clipping led to the Penmarc'h disaster of 23 May 1925. From there, local history references identified the likely person as Yves-Marie Gourlaouen, aged 15.
