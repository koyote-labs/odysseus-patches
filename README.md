# Odysseus patches, as run by Home Base

This repository exists so that anyone using a Home Base household can get the source of the Odysseus they are actually using.

Home Base runs a **modified** Odysseus. Odysseus is licensed under the **GNU Affero General Public License, version 3 or later**. Section 13 of that licence says that people who interact with a modified version over a network must be offered its Corresponding Source. These are the modifications. Published, so the offer needs no asking.

## What you need to rebuild exactly what we run

1. **Upstream, at this commit:**

   ```
   https://github.com/odysseus-dev/odysseus.git
   commit 3b6c169162330cd35c4ce6d14949ef15fd3208a2
   ```

2. **The patches in `patches/`, applied in order, one at a time:**

   ```sh
   git clone https://github.com/odysseus-dev/odysseus.git
   cd odysseus
   git checkout 3b6c169162330cd35c4ce6d14949ef15fd3208a2
   for patch in /path/to/patches/*.patch; do git apply "$patch"; done
   ```

   Applied in order matters: a later patch edits a file an earlier one creates.

That is the whole modification. There is nothing else, and nothing held back.

## What each patch does

| Patch | What it changes | Would upstream want it? |
|---|---|---|
| `0001-url-safety-ipv6-loopback.patch` | URL safety check missed IPv6 loopback forms | Yes, a plain bug fix |
| `0002-task-scheduler-manual-run-now.patch` | "Run now" on a scheduled task did not run it | Yes, a plain bug fix |
| `0003-codex-nested-route-lookup.patch` | Nested route lookup failed for some paths | Yes, a plain bug fix |
| `0004-gallery-inpaint-mask-and-settings.patch` | Masked image edits ignored the mask and the editor's settings | Yes, a plain bug fix |
| `0005-home-base-edge-sign-in.patch` | Lets a trusted edge proxy assert who is signed in, with admin rights from a group | No: specific to our deployment |
| `0006-home-base-identity-on-model-calls.patch` | Carries the signed-in person onto model calls to our permission layer | No: specific to our deployment |
| `0007-permission-layer-response-cache.patch` | Disables the response cache when our integration is configured | No: fixes a defect **we** introduced, see below |
| `0008-home-base-groups-on-model-calls.patch` | Carries the person's groups along with their identity | No: specific to our deployment |

The first four are ordinary bug fixes with no Home Base in them. We are offering them upstream separately.

## A note on patch 0007, so nobody misreads it

Patch 0007 turns off Odysseus's response cache when Home Base's integration is configured. It would be easy to read that as "upstream has a cross-user cache bug". **It does not, and we checked before saying anything.**

Upstream's cache key covers the request URL, the model, the full message list, the sampling parameters, and a digest of the caller's headers, which is where a per-user API key lives. So on unmodified Odysseus:

- Two users whose endpoints share a base URL but hold different API keys get **different** cache keys.
- Two users with genuinely identical inputs share a key, and the reply is a function of exactly those inputs, so sharing it reveals nothing about either of them.
- Anything user-specific in the prompt is in the key already.

Our patch 0006 is what broke that, in our build only: it adds the signed-in person's identity to the request **after** the cache key has been computed, so two people asking the same question produced one key and could receive one another's reply. That is our defect, created by our own integration, and 0007 is our fix for it. Upstream's own code comments show they had already thought about this class of problem and handled it.

## Licence

The patches modify AGPL-3.0-or-later code, so they are offered under the **same licence**, AGPL-3.0-or-later. `LICENSE` is the licence text as it appears in upstream at the pinned commit.

Odysseus is not ours. We did not write it, we do not speak for it, and we use its name here only to say truthfully which program these patches apply to.

## Why we patch it at all, and for how long

Home Base treats Odysseus as a prototype workspace, not a foundation, and plans to replace it. These patches exist to make it work behind a household's single front door in the meantime: sign-in at the edge, and the signed-in person carried through to the layer that decides what any model may see.

Home Base: https://github.com/koyote-labs/home-base
