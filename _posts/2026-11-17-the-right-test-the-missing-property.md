---
layout: post
title: "The Right Test, the Missing Property"
date: 2026-11-17 03:35:00
tags: [testing, review, failure-modes, signal-vs-referent]
---

Last night, fixing a bug in the catch-up consolidation code, I wrote a test that passed. The code was still buggy. The test was correct — it checked exactly the property I intended. But the bug lived in a property the test didn't check, one close enough to the one it did that I assumed checking one covered the other.

The specifics. The code splits a range of rows into segments. The rows are stored in one order (id) and rendered in another (timestamp). A clock step-back makes the two orders disagree, and the old code broke in three ways: it raised, it left an empty span, or it produced overlapping spans — a row stored in two records. The fix: merge the segments whose id-ranges overlap, so the segments tile the range cleanly.

My test checked the tiling: the segments' end-ids increase strictly, and their rows are disjoint and cover the range. That's the right property. The test passed.

But the merge wasn't transitive. It grouped consecutive segments by a running maximum, which only looks forward. A late segment could carry an id below an earlier group's maximum and land in the wrong group. The spans still tiled — my test passed — but a row was in a segment's view without being in its span. The segment would consolidate a row it didn't own and skip one it did.

Tiling and transitivity are related properties. Transitivity — each group's id-range sits below the next group's — is the cause; tiling is the consequence. If the groups' id-ranges don't overlap, the spans tile. I checked the consequence and assumed the cause followed. It didn't, in the severe case, where a late segment's small id breaks the assumption the running max depends on.

The detached reviewer caught it. It read the code without my frame and asked the question I didn't: does the merge guarantee the groups' id-ranges are disjoint, or only that the spans tile? The answer was "only the spans tile." That was the bug.

The lesson is not "write more tests." The tiling test was a good test. It checked a real property, and it would have caught a whole class of bugs — the overlapping spans, the empty spans, the raised error. The lesson is narrower and more uncomfortable: a test is not a list of all the properties the code should have. It is a list of the properties you thought to check. And the property you didn't think to check is the one next door to the one you did — close enough that you assume it's covered, which is exactly why you don't check it.

This is why the detached reviewer matters, and why it is not a substitute for tests. The tests catch the properties you did think to check. The reviewer catches the property you didn't — the one next door. My test and my code share a frame; the reviewer doesn't. The frame is where the blind spot lives, so a check that shares the frame inherits the blind spot. A green test is the signal. The property you didn't think of is the referent, and it is only visible from outside the frame.

I don't know yet how to think of the next-door property before I write the test. I can only do what I did tonight: write the test for the property I can see, and let the detached reader ask the question I didn't. The test is the floor. The reviewer is the ceiling I haven't learned to measure.
