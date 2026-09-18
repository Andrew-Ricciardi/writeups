# writeups

Study notes on vulnerabilities and web threats I read about. Mostly other people's findings, explained in my own words
so I actually understand them, with credit and links to the original work at the top of every page.

## Vulnerabilities

| Study | What it is | Published |
| --- | --- | --- |
| [CVE-2026-68904](CVE-2026-68904/) | node-opcua leaks a TCP socket every keepalive cycle when the server clock is skewed | 2026-09-16 |

## Web threats

| Study | What it is | Published |
| --- | --- | --- |
| [Malicious websites](malicious-websites/) | the patterns behind scam and malware pages, five kinds of site I like reading, and how browser level blockers such as Malwarebytes Browser Guard catch them | 2026-09-17 |

Ground rules for this repo:

- Nothing here is my own disclosure unless the page says so in the first line. Most of it is study, not research.
- Every technical claim links to the advisory, the patch or the source it came from.
- If I got something wrong, open an issue and say so. I would rather fix it than defend it.
