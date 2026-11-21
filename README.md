# FAQ: mounting WebDAV / NAS as a Windows drive letter

The questions below come from things people actually asked me, plus the ones I hit myself while setting this up on my own machines. Disclosure first: I work on ScsDrive (https://drive.scsoi.com/), so read the product mentions as biased. The mechanics below are not marketing and do not depend on who wrote them.

## "Map network drive" vs. a real drive letter — is it the same?

No, and this is the single biggest source of confusion.

Windows' own **Map network drive** (the WebClient service) creates a *network drive*. It shows up under **Network locations**, it is handled by a redirector, and software that insists on a local path will refuse it or treat it oddly.

A **real local volume** shows up under **This PC → Devices and drives** with a normal letter (D:, E: …) and behaves like an internal disk. That is the difference people are usually chasing when they say "I want it as a drive letter, not a network share".

## Does mounting need administrator rights?

Mounting does not. You mount with your own Windows account in your own session.

One caveat that is easy to gloss over: on the **first launch** the client sets up the open-source user-mode file-system driver it needs (Dokany, LGPL) by itself. It is not a kernel component and there is nothing for you to install or configure — but it does mean the very first start does more than a plain `subst` would.

## Will the drive still be there after a reboot? Will my backup service see it?

This is where I lost the most time, so here it is in one paragraph.

A mount belongs to **the Windows session that created it**. Two consequences follow:

- Services and scheduled tasks running in **Session 0** — the isolated session Windows keeps for system services — do **not** see the drive letter. A backup agent or script started that way will report a missing path even though the drive is right there in your desktop session. Run it in an interactive session, as the same user who owns the mount.
- After a reboot the drive letter is **not** there at boot. The client can re-create the mounts when you sign in again, so if a script runs before you log in, it will run before the mount exists.

## What if the network drops in the middle of a transfer?

The client re-establishes the connection on its own when the network comes back, and it restores its mounts at start-up. What it cannot promise is that a single file write interrupted mid-flight survives untouched — the same is true of any filesystem over a flaky link, and it is worth checking the file if the connection died during a write.

## Does it work with my NAS?

Anything that speaks **WebDAV** works, which covers Synology and QNAP with WebDAV enabled, Nextcloud, AList / OpenList, self-hosted WebDAV services, and cloud drives exposed through a WebDAV gateway. Each mounted service gets its own letter, so several can be mounted at once without fighting over letters.

## What does it cost, and what is the catch?

The trial is 7 days with the full feature set, and there are no ads and no forced subscription. The honest catch is the session scope described above: if your workflow depends on a service or scheduled task seeing the drive, plan for that. Using a limited-rights account on the NAS instead of the administrator account is a good habit regardless.

## Where to get it

Official site and downloads: https://drive.scsoi.com/

If you want the longer, measured write-up instead of a FAQ — including how the file semantics behave once WebDAV is mounted as a local volume — it is here: https://github.com/shuyingegit/scsdrive-webdav-file-semantics
