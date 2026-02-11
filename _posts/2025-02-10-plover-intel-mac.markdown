---
layout: post
title:  Plover, Python, and Spinning Wheels
date:   2026-02-10 13:37:00 -0800
categories: [macOS, Intel, Apple Silicon, arm64, x86_64, Universal]
---
Recently I've gotten into learning about stenography. Have you ever watched a legal drama and wondered how the court reporters are able to keep up with oral arguments yet don't seem to by tip-typing at their keyboards like their playing "Through the Fire and the Flames" by DragonForce? They do that through stenography where you hold down multiple keys at once like a chord in piano playing and software look through pre-defined chord dictionaries to change that into a words, phrases, emojis, or more. No more individually typing every letter!

tl;dr? There's a nice [3 minute YouTube video](https://youtu.be/62l64Acfidc) someone made for you.

In order to get started with stenography, you need to invest either in professional stenography machines which cost as much as the highest end Macbooks (multiple thousands of US dollars) or in hobbyist keyboards which cost ~$100 to $200. Normal keyboards are not preferable as they will usually be unable input more than 4-7 keys at the same time and so you'd have to use advanced techniques like arpeggio to work around this.

I opted for a nice hobbyist keyboard called the Stenoob and installed [Plover](https://github.com/opensteno/plover), a software for translating steno keyboard strokes into words like a full steno machine. It's available on macOS, Linux, and Windows.

Unfortunately, my first experience with Plover was immediately seeing it hang when opening up the newly installed application, seeing the lovely app icon just bounce in my dock forever.

Opening up `Console.app` in `Utilities`, I started streaming the log messages and saw this:

```
default	13:09:54.113432-0800	kernel	ASP: Library load (/Applications/Plover.app/Contents/Frameworks/Python.framework/Versions/3.13/bin/python3.13 -> /Users/abotkin/ffi6LScct) rejected: library load disallowed by system policy
error	13:09:54.113828-0800	syspolicyd	Unable (errno: 2) to read file at <private> for process path: <private> library path: <private>
error	13:09:54.113857-0800	syspolicyd	Disallowing load of <private> in 5315, <private>
default	13:09:54.113877-0800	kernel	ASP: Library load (/Applications/Plover.app/Contents/Frameworks/Python.framework/Versions/3.13/bin/python3.13 -> /private/var/folders/qd/b3kcwgkn1ql48gwb4wyv7yn80000gn/T/ffiIehdoT) rejected: library load disallowed by system policy
error	13:09:54.114255-0800	syspolicyd	Unable (errno: 2) to read file at <private> for process path: <private> library path: <private>
error	13:09:54.114281-0800	syspolicyd	Disallowing load of <private> in 5315, <private>
default	13:09:54.114300-0800	kernel	ASP: Library load (/Applications/Plover.app/Contents/Frameworks/Python.framework/Versions/3.13/bin/python3.13 -> /private/tmp/ffiQCNojM) rejected: library load disallowed by system policy
error	13:09:54.114708-0800	syspolicyd	Unable (errno: 2) to read file at <private> for process path: <private> library path: <private>
error	13:09:54.114737-0800	syspolicyd	Disallowing load of <private> in 5315, <private>
default	13:09:54.114756-0800	kernel	ASP: Library load (/Applications/Plover.app/Contents/Frameworks/Python.framework/Versions/3.13/bin/python3.13 -> /private/var/tmp/ffiJCJnRy) rejected: library load disallowed by system policy
error	13:09:54.115170-0800	syspolicyd	Unable (errno: 2) to read file at <private> for process path: <private> library path: <private>
error	13:09:54.115200-0800	syspolicyd	Disallowing load of <private> in 5315, <private>
default	13:09:54.115220-0800	kernel	ASP: Library load (/Applications/Plover.app/Contents/Frameworks/Python.framework/Versions/3.13/bin/python3.13 -> /Users/abotkin/ffiyIMJHa) rejected: library load disallowed by system policy
error	13:09:54.115618-0800	syspolicyd	Unable (errno: 2) to read file at <private> for process path: <private> library path: <private>
error	13:09:54.115648-0800	syspolicyd	Disallowing load of <private> in 5315, <private>
default	13:09:54.115669-0800	kernel	ASP: Library load (/Applications/Plover.app/Contents/Frameworks/Python.framework/Versions/3.13/bin/python3.13 -> /private/var/folders/qd/b3kcwgkn1ql48gwb4wyv7yn80000gn/T/ffiwW0pH5) rejected: library load disallowed by system policy
error	13:09:54.116060-0800	syspolicyd	Unable (errno: 2) to read file at <private> for process path: <private> library path: <private>
error	13:09:54.116089-0800	syspolicyd	Disallowing load of <private> in 5315, <private>
default	13:09:54.116111-0800	kernel	ASP: Library load (/Applications/Plover.app/Contents/Frameworks/Python.framework/Versions/3.13/bin/python3.13 -> /private/tmp/ffiI0mKMq) rejected: library load disallowed by system policy
```

Reviewing through the releases on GitHub, I found that the version I'd downloaded, Plover 5.1.0, had recently added code-signing and notarization to the macOS app due to the increasing security requirements in macOS for running apps from unidentified developers. Installing an earlier version, 5.0.0, worked just fine on my 2020 iMac 5k.

When macOS applications are code-signed and notarized (using the Developer ID certificate), Apple now requires that the app has the `Hardened Runtime` enabled for the application which [adds several security restrictions](https://developer.apple.com/documentation/security/hardened-runtime). As seen from the Console logs, those security policies were preventing some module in the Python setup of Plover from being able to load a library that was not in the code-signed binary. Given the description and the location of the library seemingly being in `/tmp`, I thought this was likely due to the module doing just-in-time compilation (JIT).

My first thought was perhaps Python was doing just-in-time compilation due to a module or library not being available in my computer's architecture (`x86_64` as Intel, while most Macs post-2020 are `arm64` Apple Silicon). Some investigation with [a helpful script](https://github.com/gregneagle/relocatable-python) uncovered that several third-party Python modules in the site-packages were not being built for both architectures (what macOS devs will call Universal or `universal2`). Working with Claude Code, I made changes to the Mac build scripts to help ensure we only used pre-built Python wheels that were `universal2` so that they contained both `x86_64` and `arm64` macOS support. There were some modules that did not have `universal2` wheels available. For these, we created code to auto-determine which modules were affected and changed the `pip` command to build those from source rather than install from the pre-built wheels.

After updating the GitHub Actions so we can run code-signing and notarization on our own fork with our own app bundle ID & developer ID certificate, I tried out the new app version, but still saw the same issue.

Reviewing the entitlements for Plover, Plover had already declared for `com.apple.security.cs.disable-library-validation`, but there are two other entitlements in the Hardened Runtime exceptions that come into play when JIT is involved: `com.apple.security.cs.allow-jit` and `com.apple.security.cs.allow-unsigned-executable-memory`

Adding in these additional exceptions and rebuilding, I tested again and found that this fixed the issue.

This fix to the Universal macOS support went out with Plover 5.2.2 on Feb 9th, 2026. If you'd like to see the PR that fixed this, you can review that [here](https://github.com/opensteno/plover/pull/1820)