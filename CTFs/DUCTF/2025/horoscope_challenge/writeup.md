# Horoscope Challenge (Gemini Protocol)
**Category:** Misc  

---

## Initial Exploration

The email from the descriptio hints at a “new and improved protocol”, this made me think: new member in a family of web protocols??? idk keep that in mind ill come back to it in a bit.

The challenge gave host/port pairs, so I decided to poke around with *netcat*.

```bash
nc chal.2025.ductf.net 30015
```

I typed a bunch of random text, pressed Enter… and got this every time:

```
2%
```

Same thing when I tried the US endpoint:

```bash
nc chal.2025-us.ductf.net 30015
```

Still just:

```
2%
```

At this point I expected some banner (like `HTTP/1.1 200 OK`, or even a usage message). That suggested the remote service was not a *plain-text protocol*.

---

### What does `2%` actually mean?

I figured out that  2%  was just random binary junk showing up as characters in my terminal. The server was expecting an `encrypted TLS handshake`, not `plain text`.
> Therefore, whatever runs on that port isn’t a cleartext ASCII protocol.
---

## Sanity Check: HTTP/2 

Because the prompt said “new and improved protocol,” I still wanted to rule out *HTTP/2* in case the server spoke cleartext *HTTP/2* (unlikely, but fast to try). *HTTP/2* connections start with a required magic preface:

```bash
printf "PRI * HTTP/2.0\r\n\r\nSM\r\n\r\n" | nc chal.2025.ductf.net 30015
```

Explanation:

- `PRI * HTTP/2.0` … `SM` → the *HTTP/2* client connection preface.
- `\r\n` sequences provide required *CRLF* line endings.
- The pipe (`|`) feeds that preface to `nc`, which connects over *TCP*.

**Result:** still just binary noise (`2%` earlier behavior). So.. not cleartext *HTTP/2*.

---

## Re-reading the Hint: Taurus… then Gemini?

The mail said:

> *Little Tommy has been born! He’s a Taurus just a month before matching his mum and dad!*

**Astrology order:** Taurus → Gemini ... ahh! ok lol its *Gemini* we have to use.

---

## TLS Handshake Check

Gemini always uses *TLS* so, as to confirm whether the service expects *TLS* i used *OpenSSL*:

```bash
openssl s_client -connect chal.2025.ductf.net:30015
```

What this does:

- Establishes a *TCP* connection.
- Performs a *TLS handshake*.
- Prints the certificate chain + session info.
- Drops you into an interactive session where *stdin/stdout* are wrapped in *TLS* (like `nc`, but encrypted).

The handshake succeded!! (TLSv1.3, self-signed cert). 

That confirmed the service is *TLS-only*, explaining the earlier `2%` garbage when I used raw `nc`.

---
## Using Gemini
Since *Gemini* uses a simple request format
```bash
gemini://host/path\r\n
```
I tried sending one directly over the *TLS* session:
```bash
printf "gemini://chal.2025.ductf.net/\r\n" | \
openssl s_client -connect chal.2025.ductf.net:30015
```
This did send the request, but the output was cluttered with handshake debug information from  *s_client*, making it hard to spot the server’s actual response.
By default, *s_client* prints lots of diagnostic output (certificate, handshake, session tickets, etc.) to *stdout* before (and while) exchanging data.

---

## Using `-quiet` 

Adding `-quiet` tells `s_client` to suppress most of the handshake chatter and behave more like a clean data pipe: what you send in goes out after the handshake, and what comes back is mostly the server’s application data. Much easier to parse, and in our case, it made the *Gemini* response (and flag) obvious.

```bash
printf "gemini://chal.2025.ductf.net/\r\n" | \
openssl s_client -connect chal.2025.ductf.net:30015 -quiet
```

The server replied with a valid *Gemini* status line + content:

```
depth=0 CN = chal.2025.ductf.net
verify error:num=18:self signed certificate
verify return:1
depth=0 CN = chal.2025.ductf.net
verify return:1
20 text/gemini;lang=en-US



# Welcome to the Wasteland Network

  

The year is 2831. It's been XXXX years since The Collapse. The old web is dead - corrupted by the HTTPS viral cascade that turned our connected world into a weapon against us.

But we survive. We adapt. We rebuild.

This simple Gemini capsule is one node in the new network we're building - free from the complexity that doomed the old internet. No JavaScript. No cookies. No tracking. Just pure, clean information exchange.

Some pages are struggling with corruption as we take further attacks.

  

## Navigation

  

=> /survival.gmi Survival Basics: First Steps in the New World

=> /salvaging.gmi Tech Salvaging: Safe Computing After the Fall

=> /community-hub.gmi Community Hub: Finding Other Survivors

=> /about-us.gmi About the Wasteland Network

  

## Daily Advisory
⚠️ ALERT: Increased bot activity reported in old HTTP sectors 44-48. Avoid all mainstream browser use in these digital quadrants.
⚠️ REMINDER: Always verify capsule certificates before sharing sensitive information. Trust no one who won't use Gemini protocol.
⚠️ WARNING: Protocol has sustainnnnnned damages. Corruption detected within [------]. ProceeX with cauXXXn

  

## Message of the Day

  

DUCTF{g3mini_pr0t0col_s4ved_us}

  

"The old web was a mansion with a thousand unlocked doors. The new web is a bunker with one good lock."

- Ada, Network Founder

Remember: Simple is safe. Complex is compromise.

  

## Update Log

  

* 2831-04-04: Added new communications relay points in sectors 7 and 9

* 2831-04-03: Updated survival maps for Western salvage zones

* 2831-04-01: Repaired node connection to Australian wasteland network
```

Status `20` in Gemini means “Success; here’s content,” and the `text/gemini` MIME type tells clients to render Gemini markup. The page contained lore text and the flag:

**Flag:** `DUCTF{g3mini_pr0t0col_s4ved_us}`