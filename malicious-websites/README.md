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

## Five kinds of site I read for fun, from a safe distance

These are a hobby for me. The pitches are a genre of their own and reading them is genuinely interesting. Installing
anything from one is a seperate decision, and thats the part I skip.

**1. Hardware anonymisers.** Sites selling "change your HWID, clean your SMBIOS, randomise your serials". To touch
identifiers that live in firmware and drivers, the tool needs a kernel driver or a loader with admin rights, and you
cannot audit either one. So the deal on offer is "give me ring 0 and I will change a string". A fair few of them are
also partly snake oil, because some of those identifiers get re-read at a higher privilege level than the tool can
reach, or simply reset on reboot.

**2. Game cheat and mod menu sites.** Same shape, worse odds. The download is a driver or an injector running with
admin, from a vendor whose entire product is defeating a check somebody else owns. When the anti-cheat catches up the
account goes, and when the vendor turns out to be a stealer the machine goes with it. Their forums are full of people
finding out which of the two they got.

**3. Cracked software and "pre activated" installers.** The busiest of the three and the least necessary. An installer
patched to skip licensing has already had its integrity thrown away, so the patched copy can carry anything, and the
sites hosting them make their money from the download page rather than from the file.

**4. Fake AI tool pages.** Every wave of new AI tools drags a wave of lookalike sites behind it: an "installer" for the
popular tool of the month, a free voice or video generator, a model download that wants a command pasted first. Research
published through 2026 describes typosquatted domains and fake installers riding the AI name of the week and ending in
infostealers. The lure is the thing everybody wants and nobody has yet, which is why it keeps working, and the install
step is where it stops being a webpage problem and becomes a machine problem.

**5. Browser extensions and store lookalikes.** One 2026 campaign got 17 extensions into the real Chrome, Firefox and
Edge stores under ordinary names, a YouTube downloader, an ad blocker, a right click translator, and passed 840,000
installs before it came down. The code hid inside images and pulled its payload at runtime. The same reporting covers
ZIP "installers" wearing the name of software people already trust, security tools included, where a sideloaded DLL
does the actual work. The store listing and the familiar brand are the entire trick, which is why the only real check is
who published it and whether you went looking for it yourself.

What all five have in common: they ask you to run something with more privilege than you would hand a stranger, from
someone with no real incentive to be straight with you, in order to beat a check that is not yours. That is exactly why
they are interesting to read and a bad idea to install. When I look at one it is offline, in a throwaway VM with
snapshots, no personal accounts, and I check the signature on any driver before letting it load.

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
   `fetch(` or `XMLHttpRequest` pointed at raw IP adresses, base64 blobs several hundred characters long.
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
| Any tool that needs a driver you must whitelist | An unauditable kernel download |
| Familiar extension or installer you did not go looking for | Impersonation, or a hijacked original |

## Sources

- Malwarebytes Browser Guard product page, feature list and protections: https://www.malwarebytes.com/browserguard
- ClickFix campaigns abusing fake Google and Cloudflare checks, Malwarebytes research summarised: https://cybernoz.com/clickfix-scams-abuse-google-cloudflare-checks-to-deliver-7-malware-families/
- Browser store and fake installer campaigns, including the extension campaign with 840,000+ installs: https://mallory.ai/stories/019bd57d-6bfc-76b4-aec0-f7f060355e1d
- AI tool impersonation and typosquatting research note, Cloud Security Alliance, March 2026: https://labs.cloudsecurityalliance.org/wp-content/uploads/2026/03/CSA_research_note_ai_tool_impersonation_installfix_typosquatting_20260310-csa-styled-2.pdf
- Malwarebytes Labs blog for ongoing campaign writeups: https://www.malwarebytes.com/blog

*I have not tested Browser Guard against a live malicious page myself. The detection notes above are from the vendor's
description and public research, and the inspection steps are how I read a page offline.*
