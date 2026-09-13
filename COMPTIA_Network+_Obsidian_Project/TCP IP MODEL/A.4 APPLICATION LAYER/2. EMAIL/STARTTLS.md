---
tags:
created: 2026-09-11
author: Mattia Vacca
---


STARTTLS is the mechanism behind that "explicit" TLS variant I mentioned for SMTP — worth walking through properly since it shows up a lot on Network+.

**The core idea:** instead of opening a dedicated encrypted port from the first byte, the client connects on the normal plaintext port, and partway through the session issues a command telling the server "let's switch to TLS now." The TCP connection stays open, but a TLS handshake happens _inside_ it, and everything after that point is encrypted. It's an in-place upgrade rather than a separate secure channel.

**Typical flow (SMTP example):**

1. Client connects on port 587 (or 25), server sends plaintext greeting.
2. Client and server exchange capability info (EHLO).
3. Client sends `STARTTLS`.
4. Server responds ready.
5. TLS handshake runs over the same connection.
6. From here on, all SMTP commands (auth, mail data, etc.) are encrypted.

**Where it's used:**

- SMTP — port 587 (submission) or 25, `STARTTLS` per RFC 3207
- IMAP — port 143, per RFC 2595
- POP3 — port 110, per RFC 2595
- LDAP — port 389, via the StartTLS extension (RFC 2830/4511)
- FTP has a conceptually similar upgrade path (`AUTH TLS` command), but it's not technically called "STARTTLS" — different protocol, same idea.

**Explicit vs. implicit, the key distinction to keep straight:**

- **STARTTLS (explicit)**: connect plaintext, then upgrade. Same port used for both secure and insecure sessions.
- **Implicit TLS**: TLS handshake happens immediately, before any protocol data is exchanged, on a dedicated port. Examples: 465 (SMTPS), 993 (IMAPS), 995 (POP3S), 636 (LDAPS).
  
  **The security wrinkle worth knowing:** STARTTLS has a history of downgrade vulnerabilities — a man-in-the-middle can strip or tamper with the `STARTTLS` negotiation in transit, tricking the client into falling back to plaintext without either side noticing (sometimes called STRIPTLS attacks). There have also been "command injection" bugs in mail servers where text sent before the handshake got processed as if it came after. Because of that history, RFC 8314 (2018) pushed mail clients/servers toward preferring implicit TLS (port 465) over STARTTLS where possible, since there's no unencrypted negotiation window to attack.

If your professor asks for the one-line distinction: **STARTTLS upgrades an existing connection; implicit TLS encrypts from the start on a separate port.**