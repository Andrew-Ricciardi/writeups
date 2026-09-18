# Malicious websites: the patterns behind the popup

Notes from reading about how scam and malware pages actually work, and how browser-level blockers such as Malwarebytes
Browser Guard get in the way. Everything here is defensive: it is a recognition guide, written so you can look at a
suspicious page and say "I know what this is trying to do". Sources are linked at the bottom. Current as of September
2026, which matters more than usual here, because the specific campaigns turn over every few months even when the
shapes do not.

## The lures worth recognising

**1. Fake human verification (the paste-and-run family, best known as ClickFix).** The page shows a reCAPTCHA or
Cloudflare style "verify you are human" box, except the instruction is to press Win+R, paste something and hit Enter.
The page is using you as the delivery mechanism. Malwarebytes research from July 2026 describes campaigns using fake
Google and Cloudflare checks, fake "unauthorised login" warnings, a Google Meet prompt claiming an audio driver needs
fixing, and a QR code generator page, all ending in a command the visitor runs themselves, and all leading to one of
seven malware families plus a previously undocumented loader.

Two variants worth knowing by name, because they defeat the habit of only distrusting the Run box:

- **FileFix** puts the paste into the File Explorer address bar instead. There is no downloaded file, so there is no
  Mark of the Web tag, so SmartScreen and the "this file came from the internet" warning never fire.
- **TerminalFix**, documented by Microsoft in August 2026, is the same fake Cloudflare check on the front. What lands
  is not a stealer but a custom reverse TCP tunnel, which makes the machine a way into whatever network it sits on. It
  chains DLL sideloading, a payload hidden inside a PNG, and encrypted WebSocket traffic to stay quiet.

Why it works: nothing needs to be exploited. The payload arrives because the visitor types it in. Hosting is cheap and
stretchy too, the same research found payloads on repurchased expired domains, Cloudflare Pages, R2 buckets,
compromised legitimate sites, raw IP hosting, and the Deno runtime abused as the thing that executes the stealer.

**2. Notification bait.** "Click Allow to confirm you are not a robot." Allow is a browser notification permission, not
a check. Once granted, the site can push notifications that look like antivirus warnings, delivery notices or bank
alerts, and the browser itself becomes the delivery channel.

**3. Fake updates and fake players.** Chrome needs updating, your Flash is out of date, the video needs a codec, the
antivirus expired. The download is the malware, or a signed binary that pulls one.

**4. Tech support scam and browser locker.** A loud page, a siren, "your computer is infected, call this number".
Some of them fight back with a loop that keeps reopening the tab or a fullscreen request, which is social engineering
with a bit of script behind it.

**5. Malvertising and redirect chains.** A real ad slot on a real site redirects through several domains, and the
landing page depends on who you are. Cloaking means the same URL shows a scam to a normal visitor and a blank page to a
reviewer or a scanner, which is why these have to be caught at request time and not by looking at the screenshot.

**6. Skimmers on checkout pages.** An injected script reads the payment form fields as they are typed and posts them
somewhere else. Nothing looks wrong to the shopper.

**7. Cryptojacking.** Quiet mining in a tab, usually inside a script bundle nobody looks at.

**8. Obfuscation and hidden frames.** Long base64 blobs, `eval`, `atob`, `fromCharCode`, `document.write` chains,
invisible iframes, or a page whose entire HTML response is a few characters and a redirect. Steganography belongs here
too: both the GhostPoster extensions and TerminalFix carried their real payload inside image files, so the JavaScript
you can read is only the part that unpacks it. Obfuscation is not proof of malice, but it is definitely a reason to
stop and read.

**9. Pages written at your AI assistant rather than at you.** If you browse with an agentic browser or a browsing
assistant, the page has a second reader, and that reader follows instructions. Hidden text, off-screen elements or
text baked into an image can tell the agent to exfiltrate what it is holding or to visit an attacker's form. Brave's
research called indirect prompt injection a systemic problem across the whole category rather than one product's bug,
and OpenAI has said publicly it is unlikely to ever be fully solved. Treat an agent reading an untrusted page as a
second attack surface with your session attached to it.

## Nobody is only attacked on sketchy sites

Worth stating plainly, because every list like the one above reads as "avoid bad neighbourhoods". In May 2026 over 700
education and technology sites running Ghost were compromised through a SQL injection flaw (CVE-2026-26980), their
admin API keys stolen, and ClickFix injected into ordinary posts and pages. The lure arrives on a site you had every
reason to trust. The tell is the instruction, not the domain.

## Five kinds of site I read for fun, from a safe distance

These are a hobby for me. The pitches are a genre of their own and reading them is genuinely interesting. Installing
anything from one is a separate decision, and that's the part I skip.

**1. Hardware anonymisers.** Sites selling "change your HWID, clean your SMBIOS, randomise your serials". To touch
identifiers that live in firmware and drivers, the tool needs a kernel driver or a loader with admin rights, and you
cannot audit either one. So the deal on offer is "give me ring 0 and I will change a string". A fair few of them are
also partly snake oil, because some of those identifiers get re-read at a higher privilege level than the tool can
reach, or simply reset on reboot.

**2. Game cheat and mod menu sites.** Same shape, worse odds. The download is a driver or an injector running with
admin, from a vendor whose entire product is defeating a check somebody else owns. When the anti-cheat catches up the
account goes, and when the vendor turns out to be a stealer the machine goes with it. Their forums are full of people
finding out which of the two they got.

**3. Cracked software and "pre-activated" installers.** The busiest of the five and the least necessary. An installer
patched to skip licensing has already had its integrity thrown away, so the patched copy can carry anything, and the
sites hosting them make their money from the download page rather than from the file.

**4. Fake AI tool pages.** Every wave of new AI tools drags a wave of lookalike sites behind it: an "installer" for the
popular tool of the month, a free voice or video generator, a model download that wants a command pasted first. A Cloud
Security Alliance research note from March 2026 covers typosquatted packages and fake install guides riding AI brand
names and ending in infostealers, and the campaign behind it is a good illustration: SEO poisoning pushed fake Gemini
CLI and Claude Code install pages above the real ones, and the "installer" was an in-memory PowerShell stealer that
took browser credentials, session cookies, OAuth tokens, CI/CD credentials and VPN details. The lure is the thing
everybody wants and nobody has yet, which is why it keeps working, and the install step is where it stops being a
webpage problem and becomes a machine problem.

**5. Browser extensions and store lookalikes.** The GhostPoster campaign, found by LayerX and Koi Security in January
2026, got 17 extensions into the real Chrome, Firefox and Edge stores under ordinary names, a YouTube downloader, an
ad blocker, a right-click translator, and passed 840,000 installs before it came down. Some of the listings had been
sitting there since 2020. The code hid inside logo images and pulled its payload at runtime. The same reporting covers
ZIP "installers" wearing the name of software people already trust, security tools included, where a sideloaded DLL
does the actual work, one of which impersonated Malwarebytes itself. The store listing and the familiar brand are the
entire trick, which is why the only real check is who published it and whether you went looking for it yourself.

What all five have in common: they ask you to run something with more privilege than you would hand a stranger, from
someone with no real incentive to be straight with you, in order to beat a check that is not yours. That is exactly why
they are interesting to read and a bad idea to install. When I look at one it is offline, in a throwaway VM with
snapshots, no personal accounts, and I check the signature on any driver before letting it load.

## What Browser Guard does about it

From the vendor's own description of the extension, the free tier covers:

- blocking malicious sites and scam or fraud pages, including tech support style pages, before the page gets to do
  anything
- ad, popup and cookie banner removal, plus trackers
- credit card skimmer protection at checkout
- search hijacking prevention
- clipboard protection, a warning when a page writes to your clipboard
- a warning layer on downloads

Advanced phishing protection, heuristic protections, compromised site protection and breach notifications sit behind a
Malwarebytes subscription rather than in the free tier, which is worth knowing if you are relying on it.

The clipboard warning is the one that maps directly onto lure 1. A ClickFix page has to put the command on your
clipboard before it tells you to paste, so a prompt saying "this page copied something" arrives at exactly the moment
the story falls apart.

The rest of the mechanics are what you would expect from a browser extension in the current extension model: domain and
URL reputation lookups against blocklists, pattern and behaviour heuristics on page content and requests, and in-page
warnings when something matches. That is also why it catches things a screenshot review would miss: the decision
happens at request time, on the real page, not on a cached copy. It also means the coverage is only as fresh as the
lists and the heuristics, so it is a seatbelt rather than a reason to click.

## How I look at a suspicious page without catching anything

1. Never open the live page on a machine you care about. VM or disposable environment, no shared clipboard with your
   real work.
2. Pull the HTML, do not execute it. `curl -sL <url> -o page.html` in a throwaway VM, then read it offline with scripts
   stripped in your head, not in a browser.
3. Remember that what curl gets is not what a victim sees. Bare curl is the exact profile cloaking serves a blank page
   to. If the response is suspiciously empty, try again with a real browser user agent and a plausible referer before
   concluding there is nothing there, and treat any difference between the two as a finding in itself.
4. Grep for the tells: `atob`, `eval`, `fromCharCode`, `unescape`, `document.write`, `iframe src=`, `window.location`,
   `fetch(` or `XMLHttpRequest` pointed at raw IP addresses, base64 blobs several hundred characters long, and anything
   that reads an image back out as data rather than displaying it.
5. Check the clipboard writes specifically: `navigator.clipboard.writeText`, `document.execCommand('copy')`, or a
   hidden textarea being selected. On a verification page that is the whole attack in one line.
6. Check where it wants to send you: every host in the file, and whether any of them are raw IPs, freshly registered
   lookalike domains, or free hosting subdomains.
7. Be careful with public scanners. urlscan.io and VirusTotal are useful, but a submission is a public act: the
   operator can see their URL was scanned, and any token or identifier in the link is now published. Submit the bare
   host if you can, not the full tracking URL.
8. Defang everything you write down. `hxxps://example[.]com` so nobody, including you, accidentally clicks it later.
9. Do not run the "verification" step, ever. If a page asks you to paste something into a terminal, a Run box or a File
   Explorer address bar, that page is the malware, and there is nothing left to investigate.

## If you already pasted it

Assume it ran, because it did, and assume it took credentials before anything else.

1. Disconnect the machine from the network. Do not "just check something first".
2. From a different, clean device, change the passwords for anything that browser was signed into, email first, and
   revoke active sessions and OAuth tokens rather than only changing the password. Stealers take session cookies, and a
   new password does not invalidate a stolen session on its own.
3. Rotate anything a developer machine holds: cloud keys, CI/CD credentials, SSH keys, VPN profiles, package registry
   tokens.
4. Turn on or re-check MFA on the accounts that matter, and look for MFA methods or recovery addresses you did not add.
5. Treat the machine as untrusted. Persistence hides in scheduled tasks, Run keys and startup folders, and a reverse
   tunnel like TerminalFix leaves nothing on screen at all. A scan that finds nothing is not a clean bill of health.
   Flatten and reinstall is the honest answer for anything you care about.
6. If it is a work machine, tell whoever handles security immediately, before cleaning anything. What you might wipe is
   what tells them how far it went.

## Red flags, at a glance

| Thing you see | What it usually means |
| --- | --- |
| Asked to paste a command to "verify" | ClickFix, the payload is you |
| Asked to paste into File Explorer's address bar | FileFix, and Mark of the Web will not save you |
| "Allow notifications to continue" | Notification spam funnel |
| Fullscreen infection warning with a phone number | Tech support scam |
| Update prompt from a site you did not go to | Fake update dropper |
| Same link, different page for different people | Cloaking, to evade scanners |
| Checkout page loading a script from an odd host | Card skimmer |
| HTML that is mostly one obfuscated blob | Something is hidden |
| An image being read as data instead of displayed | Payload hidden by steganography |
| Hidden or off-screen text addressed to an assistant | Prompt injection aimed at your AI browser |
| Any tool that needs a driver you must whitelist | An unauditable kernel download |
| Familiar extension or installer you did not go looking for | Impersonation, or a hijacked original |

## Sources

- Malwarebytes Browser Guard product page, feature list and free versus premium split:
  https://www.malwarebytes.com/browserguard
- "Fake Google and Cloudflare verification pages spread multiple malware families", Malwarebytes Threat Intel,
  2 July 2026, the seven families, the lures and the hosting:
  https://www.malwarebytes.com/blog/threat-intel/2026/07/fake-google-and-cloudflare-verification-pages-spread-multiple-malware-families
- "TerminalFix looks like ClickFix, but delivers a very different payload", Malwarebytes, September 2026:
  https://www.malwarebytes.com/blog/news/2026/09/terminalfix-looks-like-clickfix-but-delivers-a-very-different-payload
- "700+ education and tech websites hijacked in huge ClickFix malware campaign", Malwarebytes, May 2026:
  https://www.malwarebytes.com/blog/bugs/2026/05/700-education-and-tech-websites-hijacked-in-huge-clickfix-malware-campaign
- FileFix and the File Explorer address bar, BleepingComputer:
  https://www.bleepingcomputer.com/news/security/filefix-attack-weaponizes-windows-file-explorer-for-stealthy-powershell-commands/
- "Malicious GhostPoster browser extensions found with 840,000 installs", BleepingComputer:
  https://www.bleepingcomputer.com/news/security/malicious-ghostposter-browser-extensions-found-with-840-000-installs/
- GhostPoster full scope, LayerX: https://layerxsecurity.com/blog/browser-extensions-gone-rogue-the-full-scope-of-the-ghostposter-campaign/
- Browser store and fake installer campaigns, including the fake Malwarebytes ZIP:
  https://mallory.ai/stories/019bd57d-6bfc-76b4-aec0-f7f060355e1d
- "AI Developer Tool Impersonation: Typosquatting, Fake Install Guides, and InfoStealer Delivery", Cloud Security
  Alliance research note, 10 March 2026:
  https://labs.cloudsecurityalliance.org/research/csa-research-note-ai-tool-impersonation-installfix-typosquat/
- SEO poisoning impersonating Gemini CLI and Claude Code, EclecticIQ:
  https://blog.eclecticiq.com/seo-poisoning-campaign-leverages-gemini-and-claude-code-impersonation-to-deliver-infostealer
- Prompt injection against agentic browsers, The Register:
  https://www.theregister.com/2025/10/28/ai_browsers_prompt_injection/
- OpenAI on prompt injection being unlikely to be fully solved, Fortune:
  https://fortune.com/2025/12/23/openai-ai-browser-prompt-injections-cybersecurity-hackers/
- Malwarebytes Labs blog for ongoing campaign writeups: https://www.malwarebytes.com/blog

*I have not tested Browser Guard against a live malicious page myself. The detection notes above are from the vendor's
description and public research, and the inspection steps are how I read a page offline.*
