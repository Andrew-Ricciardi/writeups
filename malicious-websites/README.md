# Malicious websites: the patterns behind the popup

Notes from reading about how scam and malware pages actually work, and how browser level blockers such as Malwarebytes
Browser Guard get in the way. Everything here is defensive: it is a recognition guide, written so you can look at a
suspicious page and say "I know what this is trying to do". Sources are linked at the bottom.

## The lures worth recognising

**1. Fake human verification (ClickFix).** The page shows a reCAPTCHA or Cloudflare style "verify you are human" box,
except the instruction is to press Win+R, paste something and hit Enter. The page is using you as the delivery
mechanism. Recent Malwarebytes research describes campaigns using fake Google and Cloudflare checks, fake "unauthorised
login" warnings, and a Google Meet prompt claiming an audio driver needs fixing, all ending in a command the visitor
runs themselves. A variant uses a QR code and a "scan this to verify" story to get the same paste into a phone or
desktop.

Why it works: nothing needs to be exploited. The payload arrives because the visitor types it in. Hosting is cheap and
stretchy too, the same research found payloads on repurchased domains, Cloudflare Pages, R2 buckets, compromised
legitimate sites and one abused JavaScript runtime.

**2. Notification bait.** "Click Allow to confirm you are not a robot." Allow is a browser notification permission, not
a check. Once granted, the site can push notifications that look like antivirus warnings, delivery notices or bank
alerts, and the browser itself becomes the delivery channel.

**3. Fake updates and fake players.** Chrome needs updating, your Flash is out of date, the video needs a codec, the
antivirus expired. The download is the malware, or a signed binary that pulls one.

**4. Tech support scam and browser locker.** A loud page, a siren, "your computer is infected, call this number".
Some of them fight back with a loop that keeps reopening the tab or a fullscreen request, wich is social engineering
with a bit of script behind it.

**5. Malvertising and redirect chains.** A real ad slot on a real site redirects through several domains, and the
landing page depends on who you are. Cloaking means the same URL shows a scam to a normal visitor and a blank page to a
reviewer or a scanner, which is why these have to be caught at request time and not by looking at the screenshot.

**6. Skimmers on checkout pages.** An injected script reads the payment form fields as they are typed and posts them
somewhere else. Nothing looks wrong to the shopper.

**7. Cryptojacking.** Quiet mining in a tab, usually inside a script bundle nobody looks at.

**8. Obfuscation and hidden frames.** Long base64 blobs, `eval`, `atob`, `fromCharCode`, `document.write` chains,
invisible iframes, or a page whose entire HTML response is a few characters and a redirect. Obfuscation is not proof of
malice, but it is definately a reason to stop and read.

## Hardware anonymisation sites, and why I dont trust them either

There is a whole category of site selling "hardware anonymisation": change your HWID, clean your SMBIOS, repair or
randomise serials, hide your machine from whoever is fingerprinting it. Some of it is marketed at privacy, most of it
gets bought by people trying to get around a game or platform ban, and the sites know it.

The trust problem here is structural. To change identifiers that live in firmware and drivers, the tool needs deeper
access to your machine than almost anything else you run: a kernel driver, a loader that asks for admin, something that
has to run before the rest of the system or it does not work at all. You cannot audit that. So the deal on offer is
"give me ring 0 and I will change a string in SMBIOS". That is a bad trade even when the vendor is honest, and there is
no verifiable reason to assume the vendor is honest.

Things I treat as red flags on these sites:

- a driver or loader you have to whitelist in your antivirus, or an instruction to turn protection off first
- tiers, subscriptions and "permanent" versus "temporary" fixes, since a permanent change is usually a firmware level
  change you can not undo
- no named developer, no company, no way to check the code, just a checkout page and a download
- claims that it defeats a specific anti-cheat or platform check, which is the same as saying the site's whole business
  depends on that company not fixing it
- activation or telemetry that has to phone home before the tool will do anything

There is also a boring technical point: plenty of these tools are partly snake oil. Several identifiers get re-read at a
higher privilege level than the tool can reach, or reset on reboot, or live in a component the tool does not touch. So
you can pay, hand over kernel access, break a driver, and still be exactly as fingerprinted as before.

If you are curious about one, treat it like any other untrusted download, only more so: a throwaway VM with snapshots,
never the machine you actually use, no personal accounts signed in, check who signed any driver before it loads, and
assume that anything you ran in there is compromised afterwards. For actual privacy work, the supported OS and browser
settings get you part of the way without handing a stranger ring 0, and raw IP adresses, cookies and browser
fingerprints are seperate problems with their own fixes.

## What Browser Guard does about it

From the vendor's own description of the extension, the free tier covers:

- blocking malicious sites and phishing attempts before the page gets to do anything
- scam and fraud blocking, including tech support style pages
- ad, popup and cookie banner removal, plus trackers
- credit card skimmer protection at checkout
- a warning layer on downloads

The mechanics are what you would expect from a browser extension in the current extension model: domain and URL
reputation lookups against blocklists, pattern and behaviour heuristics on page content and requests, and in page
warnings when something matches. That is also why it catches things a screenshot review would miss: the decision
happens at request time, on the real page, not on a cached copy.

## How I look at a suspicious page without catching anything

1. Never open the live page on a machine you care about. VM or disposable environment, no shared clipboard with your
   real work.
2. Pull the HTML, do not execute it. `curl -sL <url> -o page.html` in a throwaway VM, then read it offline with scripts
   stripped in your head, not in a browser.
3. Grep for the tells: `atob`, `eval`, `fromCharCode`, `unescape`, `document.write`, `iframe src=`, `window.location`,
   `fetch(` or `XMLHttpRequest` pointed at raw IP addresses, base64 blobs several hundred characters long.
4. Check where it wants to send you: every host in the file, and whether any of them are raw IPs, freshly registered
   lookalike domains, or free hosting subdomains.
5. Defang everything you write down. `hxxps://example[.]com` so nobody, including you, accidentally clicks it later.
6. Do not run the "verification" step, ever. If a page asks you to paste something into a terminal, that page is the
   malware, and there is nothing left to investigate.

## Red flags, at a glance

| Thing you see | What it usually means |
| --- | --- |
| Asked to paste a command to "verify" | ClickFix, the payload is you |
| "Allow notifications to continue" | Notification spam funnel |
| Fullscreen infection warning with a phone number | Tech support scam |
| Update prompt from a site you did not go to | Fake update dropper |
| Same link, different page for different people | Cloaking, evade scanners |
| Checkout page loading a script from an odd host | Card skimmer |
| HTML that is mostly one obfuscated blob | Something is hidden |
| "Permanent HWID change", driver must be whitelisted | An unauditable kernel tool, at best |

## Sources

- Malwarebytes Browser Guard product page, feature list and protections: https://www.malwarebytes.com/browserguard
- ClickFix campaigns abusing fake Google and Cloudflare checks, Malwarebytes research summarised: https://cybernoz.com/clickfix-scams-abuse-google-cloudflare-checks-to-deliver-7-malware-families/
- Malwarebytes Labs blog for ongoing campaign writeups: https://www.malwarebytes.com/blog

*I have not tested Browser Guard against a live malicious page myself. The detection notes above are from the vendor's
description and public research, and the inspection steps are how I read a page offline.*
