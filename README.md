# systemd — No Age Verification

> A fork of [systemd](https://github.com/systemd/systemd) with age verification infrastructure removed.

---

> [!NOTE]
> This only affects **systemd v260+**. Most Linux Operating Systems (OS) are alright as it comes with **systemd v259**.

## What Was Removed

Three commits have been reverted from upstream systemd `main`:

| Commit | Description |
|--------|-------------|
| `acb6624fa1` | userdb: add birthDate field to JSON user records (#40954) — merge commit |
| `7a858878a0` | userdb: add birthDate field to JSON user records |
| `c72860d5f6` | userdb: mark PII fields as sensitive in user records |

These commits introduce a `birthDate` field into systemd's userdb JSON user records, along with supporting parser infrastructure (`parse_birth_date`, `BIRTH_DATE_UNSET`, `allow_pre_epoch` calendar parsing).

---

## Why

Several US state laws and international regulations are driving OS-level age verification infrastructure into the Linux stack:

- **California AB-1043** (not yet enacted)
- **Colorado SB26-051**
- **Brazil Lei 15.211/2025**

In response, a coordinated effort is pushing age verification changes across multiple projects simultaneously — systemd userdb, xdg-desktop-portal ([PR #1922](https://github.com/flatpak/xdg-desktop-portal/pull/1922)), and Arch Linux's installer ([PR #4290](https://github.com/archlinux/archinstall/pull/4290)).

The concern is not that the contributors are malicious — they are established, publicly known developers. The concern is the **infrastructure itself**:

- Once `birthDate` is stored in systemd userdb, the xdg-desktop-portal age verification API can expose it to any Flatpak app with the right manifest permission.
- The API design allows apps to send fine-grained age gates (e.g. `[13, 14, 15, 16, 17, 18]`) to effectively extract a user's exact date of birth — this was acknowledged as a risk in the xdg-portal PR discussion itself.
- **The law is not yet enacted.** This infrastructure is being built preemptively.
- Users outside California and Colorado — the vast majority of Linux users worldwide — have no legal obligation and may be subject to conflicting privacy laws (e.g. GDPR in Europe) if this data is collected.

## Building

### Dependencies (Arch Linux)

```bash
pacman -S base-devel git meson ninja gperf python-jinja \
dbus openssl cryptsetup tpm2-tss \
  curl libidn2 p11-kit libfido2
```

### Build

```bash
git clone https://github.com/r4shsec/systemd-no-age-verification.git
cd systemd-no-age-verification
meson setup build/ --prefix=/usr
ninja -C build/
```

### Before Installing — Back Up First

```bash
sudo cp -r /usr/lib/systemd /usr/lib/systemd.bak
sudo cp /usr/bin/systemctl /usr/bin/systemctl.bak
```

### Install

```bash
sudo ninja -C build/ install
sudo systemctl daemon-reexec
```

### Rollback If Needed

```bash
sudo cp -r /usr/lib/systemd.bak /usr/lib/systemd
sudo cp /usr/bin/systemctl.bak /usr/bin/systemctl
sudo systemctl daemon-reexec
```

---

## Verification

After building, confirm no birthdate code was compiled in:

```bash
nm build/systemd | grep -i birth
nm build/userdbctl | grep -i birth
strings build/userdbctl | grep -i birth
```

All three should return **empty**.

Check a user record contains no `birthDate` field:

```bash
build/userdbctl user root
```

Or on an installed system:

```bash
userdbctl user $USER
```

---

## Related

- [xdg-desktop-portal PR #1922](https://github.com/flatpak/xdg-desktop-portal/pull/1922) — Age verification portal API (draft, open)
- [archinstall PR #4290](https://github.com/archlinux/archinstall/pull/4290) — Birth date field in Arch installer (locked, pending Arch legal review)
- [systemd issue #40974](https://github.com/systemd/systemd/pull/40954) — Added `birthDate` field to JSON user records (watch this)
- [QubesOS issue #10744](https://github.com/QubesOS/qubes-issues/issues/10744) — Qubes tracking age verification compliance requirements

---

## Distros That May Ship Without Age Verification

- **Artix Linux** — stated it will not comply
- **Void Linux** — source-based, full compile control
- **Gentoo** — USE flags give fine-grained control
- **Arch Linux** — pending official legal stance (see archinstall PR)

---

## Disclaimer

This fork is provided for privacy-conscious users who wish to opt out of age verification infrastructure before any legal mandate requires it. It does not condone circumventing any applicable law. Users are responsible for their own legal compliance.

This fork is not affiliated with the systemd project or any of its contributors.
