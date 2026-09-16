# writeups

Study notes on vulnerabilities I read about. Mostly other people's findings, explained in my own words so I actually
understand them, with credit and links to the original work at the top of every page.

| Study | What it is | Published |
| --- | --- | --- |
| [CVE-2026-68904](CVE-2026-68904/) | node-opcua leaks a TCP socket every keepalive cycle when the server clock is skewed | 2026-09-16 |

Ground rules for this repo:

- Nothing here is my own disclosure unless the page says so in the first line. Most of it is study, not research.
- Every technical claim links to the advisory, the patch or the source it came from.
- If I got something wrong, open an issue and say so. I would rather fix it than defend it.
