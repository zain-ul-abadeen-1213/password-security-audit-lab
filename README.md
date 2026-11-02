# Project 4 — Password Security Audit Lab

**Cyber Projects Sprint · Zain Tech Automation Solutions**

A hands-on password security lab on Kali Linux: self-created password hashes were
cracked offline with John the Ripper and Hashcat, and an online login-guessing test
was run against the lab VM's own SSH service with Medusa and Hydra. The lab closes
with the password policy written from its findings.

> **Scope & ethics:** Every test in this project was performed on my **own** test
> passwords, my **own** hashes, and my **own** virtual machine (127.0.0.1). Nothing
> belonging to anyone else was tested. This is the only way this work is ever done.

## Lab environment

| Item | Detail |
|---|---|
| Host | Kali Linux 2026.3 in VirtualBox (user `zain`) |
| Wordlist | `rockyou.txt` — 14,344,385 passwords (shipped pre-extracted in Kali's `wordlists` package) |
| Tools | John the Ripper, Hashcat, Medusa 2.3, Ncrack 0.7, Hydra 9.7 |

## Part 1 — Offline cracking (hashes)

Three test passwords were hashed with MD5 and stored in `hashes.txt`:

| Test password | MD5 hash |
|---|---|
| `karachi123` | `253acb401ca5ef629fb17e520cfc64e6` |
| `admin2024` | `8d4db54daf7d67db5f3c96e43f61c609` |
| `ZainTech@2026!Long` | `cbdb749587a1e3d2481e0ab3a06e4f3d` |

Commands used:

```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt /home/zain/hashes.txt
john --format=raw-md5 --show /home/zain/hashes.txt
hashcat -m 0 /home/zain/hashes.txt /usr/share/wordlists/rockyou.txt
hashcat -m 0 /home/zain/hashes.txt --show
```

### Crack-time table

| Password | Type | John | Hashcat | Verdict |
|---|---|---|---|---|
| `karachi123` | Common / weak | **Cracked — full rockyou scanned in ~2 s (5,951 Kp/s)** | **Cracked** (1/3 recovered) | Broken instantly |
| `admin2024` | Word + year pattern | Not found | Not found | Survived *this* list only — see note |
| `ZainTech@2026!Long` | Long passphrase (19 chars) | Not found | Not found | **Held** against the full 14.3M-word list |

Notes:
- John scanned the entire rockyou list in about **2 seconds**; Hashcat's full run on
  this VM's CPU (1,352 kH/s) took about **84 seconds** in total. On GPU hardware
  Hashcat is normally the faster tool — tool speed depends on the machine.
- `admin2024` was **not** cracked because it does not exist in rockyou (a leak from
  ~2010, which predates "2024" password patterns). That is a gap in the wordlist,
  not strength in the password — a word+year pattern is still predictable and weak.

## Part 2 — Online guessing (own VM only)

A 3-word list (`123456`, `password`, `zain123`) was tested against the SSH login of
the lab VM itself (`127.0.0.1`, user `zain`):

```bash
medusa -h 127.0.0.1 -u zain -P /home/zain/passlist.txt -M ssh
ncrack -p 22 --user zain -P /home/zain/passlist.txt 127.0.0.1
hydra -l zain -P /home/zain/passlist.txt ssh://127.0.0.1
```

| Tool | Result |
|---|---|
| Medusa 2.3 | **SUCCESS — `ACCOUNT FOUND: zain / zain123` in ~2 seconds** |
| Ncrack 0.7 | **Crashed (segmentation fault)** — tool failure, recorded honestly |
| Hydra 9.7 | **1 valid password found — `zain123`** |

### Key finding

The lab VM's **own real login password was `zain123`** — a name+digits password that
an online guessing tool found on its third try, in seconds. The weakest password in
this lab was not a test case; it was the machine's actual password. It was changed
to a long passphrase as the closing step of this lab.

## Findings summary

1. **Common passwords fall instantly** — a full 14.3M-word dictionary attack against
   MD5 took seconds (`karachi123`).
2. **Word+year patterns are not safety** — `admin2024` survived only because one
   particular old wordlist lacks it.
3. **Length and uniqueness win** — a 19-character self-made passphrase survived the
   entire rockyou list in both tools.
4. **Online guessing works when the password is guessable** — `zain123` was found by
   two independent tools in seconds against a live login.
5. **Tools fail too** — Ncrack crashed on this system; the test was completed with
   Hydra instead. Failures are recorded, not hidden.

## Password Policy (deliverable)

Written from this lab's findings:

1. **Minimum 12 characters** — 15+ preferred; a multi-word passphrase is best.
2. **No names, no years, no patterns** — `name123`, `word+2024` style passwords are
   guessed first.
3. **A different password for every account** — one leak must not open everything.
4. **MFA / two-step verification ON** wherever available — a stolen password alone
   should not be enough.
5. **Use a password manager** instead of reusing or writing passwords down.
6. **Change default passwords immediately** on routers, cameras, and new accounts.
7. **Never share passwords** — and never ask clients for theirs; audit by policy and
   demonstration on lab hashes, not by collecting real passwords.

## Screenshots

**John — hashes loaded, `karachi123` cracked, full rockyou run DONE in ~2 seconds:**

![John crack result](screenshots/john-crack-result.png)

**John `--show` — "1 password hash cracked, 2 left":**

![John show result](screenshots/john-show-result.png)

**Hashcat — Status: Exhausted, Recovered 1/3, 1,352 kH/s on the VM's CPU:**

![Hashcat run result](screenshots/hashcat-run-result.png)

**Hashcat `--show` — the cracked hash:password pair:**

![Hashcat show result](screenshots/hashcat-show-result.png)

**Medusa — `ACCOUNT FOUND: zain / zain123 [SUCCESS]` against my own VM's SSH:**

![Medusa success](screenshots/medusa-success.png)

**Hydra — "1 valid password found" (`zain123`):**

![Hydra success](screenshots/hydra-success.png)

## Lab notes

- On Kali 2026.3 the `wordlists` package ships rockyou **already extracted** as
  `/usr/share/wordlists/rockyou.txt`; the `rockyou.txt.gz` path used in many guides
  does not exist on this install.
- Kali's shell is zsh: an unquoted `!` inside a password triggers history expansion
  ("event not found"). Single quotes fix it — a small lesson in how shells read input.

---

*Part of the Cyber Projects Sprint by Zain — Zain Tech Automation Solutions ·
"Automate. Accelerate. Achieve." Educational lab work on my own systems only.*
