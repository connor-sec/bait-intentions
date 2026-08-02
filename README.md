# Bait Intentions

A USB Rubber Ducky payload for running honest USB drop tests (baiting).

Place a few of these around the office. Someone picks one up, plugs it in, and instead of getting pwned they get signed up for security awareness training — and a note lands in the IT helpdesk queue saying so. That it, it types a URL, it drafts an email. No shells, no persistence, no exfil.

## What actually happens
Someone plugs in the drive. In about three seconds:

1. The Ducky figures out whether it's on Windows or macOS and waits only as long as that machine needs to be ready for keystrokes.
2. It opens the default browser to your training portal's signup page and enrols the person.
3. It closes the browser and drops a pre-filled email to your helpdesk into their mail client.
4. It leaves that email sitting there, composed but unsent, for the person to read and send themselves.

Step 4 is deliberate, and we'll come back to it.

## Setup

You need a [USB Rubber Ducky (Mk II)](https://shop.hak5.org/products/usb-rubber-ducky) and [PayloadStudio](https://payloadstudio.hak5.org) or [PayloadStudioPro](https://payloadstudio.hak5.org/pro/). 

1. Open `payload.txt` in PayloadStudio.
2. Find the line that reads `EXTENSION PASSIVE_WINDOWS_DETECT`, retype it, and accept the autocomplete. This pulls in the official extension body, which is about forty lines that don't belong in version control. **The payload won't work without this step.**
3. Edit the three things at the top of the file (below).
4. Hit Generate Payload, drop the resulting `inject.bin` on the root of the MicroSD card.

One card does both operating systems. There's no separate Windows build and Mac build — the OS check happens at runtime, inside the one binary.

### The three things you need to change
Everything you'll want to touch is at the top of the file under a big `EDIT THESE THREE THINGS` banner.

**Your training portal.** Point `SIGNUP_URL` at your own signup page.

**Your helpdesk address and message.** Change `helpdesk@firm.com` and edit the subject and body text. This one has rules, because it's a `mailto:` URI being typed into a launcher:

- Spaces have to be written as `%20`. A literal space will split the argument and the whole thing falls apart.
- Use `%0A` for a line break.
- Commas, periods and exclamation marks are fine as-is.
- Keep it short. Every character in that line is a keystroke the Ducky physically types, so a chatty email is a slow payload.

**Timings.** `BROWSER_WAIT` and `MAIL_WAIT` are almost the entire runtime. Tune them down until things break on your slowest machine, then back off a little. Macs generally need more than Windows here.

You'll also want to check the signup form section. It assumes tabbing past a username and password field for SSO, then picking the third option in a dropdown. If your form is shaped differently, adjust the `DOWNARROW` count.

## Why it's fast

Keystroke injection payloads traditionally open with `DELAY 3000` — three seconds of dead air, because the Ducky can type faster than a freshly-plugged-in machine can listen, and three seconds is the number that works nearly everywhere.

This one doesn't guess. When a host finishes setting up a USB keyboard, it pushes lock-key LED state back to that keyboard. Hak5's `PASSIVE_WINDOWS_DETECT` extension watches for that signal and moves on the instant it arrives, which on a warm Windows box is around 25 milliseconds instead of 3000. It also tells us which OS we're on as a free side effect, which is where the cross-platform branching comes from.

A few smaller things:

- One launch instead of two. Typing the URL straight into the Run dialog or Spotlight beats opening a browser and then navigating.
- A pre-filled `mailto:` beats driving the mail client's UI. Tabbing between compose fields is exactly where these payloads break.
- The email body is deliberately terse. Trimming a polite greeting and sign-off cut 142 bytes off the compiled binary.
- `Optimize DELAY encoding` is worth enabling in PayloadStudio's compiler settings. `Always use new DELAY encoding` is not — it made our binary bigger.

Compiles to about 1 KB.

## Known limits

**macOS doesn't get the fast start.**
**It enumerates as an Apple keyboard.** 
**It doesn't press Send.** The send shortcut differs by mail client, and firing the wrong key combination into the wrong app is a good way to do something you didn't intend. Three options are commented at the bottom of the payload — uncomment one once you know what your fleet runs.

It has been argued that leaving it unsent may be the better design regardless. Someone reading a pre-written "I plugged in a USB drive I found" email and clicking Send themselves is a sharper lesson than a message that quietly vanishes.

**Timings are a starting point, not a truth.** Every number in the config block came from one setup. Test on the oldest, slowest machine in the building before you scatter anything.

## Before you use this

Only run drop tests against an organization that has authorized you to do so, in writing, with a defined scope. Get that sorted before the drives hit the carpet.

Give some thought to how the results get used, too. A drop test measures how easy it is for a normal person to be curious at work — it doesn't tell you anything about who's careless. Handing management a list of names is a good way to make sure nobody ever reports the next thing they plug in. Report the rate, not the roster.

## Credits

Built on DuckyScript 3.0. The readiness detection is Hak5's own [`PASSIVE_WINDOWS_DETECT`](https://github.com/hak5/usbrubberducky-payloads/tree/master/payloads/extensions) extension — worth reading [their writeup](https://shop.hak5.org/blogs/usb-rubber-ducky/detect-ready) on why the three second delay was never really necessary.

## License

The payload and documentation here are MIT licensed — see [LICENSE](LICENSE).

That covers this repository's own work only. DuckyScript and USB Rubber Ducky are Hak5 LLC intellectual property under the [Hak5 license](https://hak5.org/license), and the official payload repository is all-rights-reserved. This is why the extension is referenced by name in step 2 of Setup rather than pasted into `payload.txt` — the body isn't ours to redistribute, and you pull it from your own licensed PayloadStudio at build time. Please keep it that way if you fork this.

Full breakdown in [NOTICE](NOTICE). Not affiliated with Hak5.
