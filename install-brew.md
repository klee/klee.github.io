---
layout: default
title: Homebrew
subtitle: Installing KLEE with Homebrew
slug: getting-started
---

The current procedure for installing KLEE using Homebrew is outlined below.

1. **Install dependencies (Linux only):** Homebrew and KLEE require some additional dependencies on Linux.

   On Ubuntu:
   ```bash
   $ sudo apt-get install build-essential curl file git
   ```

   For Ubuntu>=16.04, some additional packages my be required (by KLEE) along with the ones mentioned above:
   ```bash
   $ sudo apt-get install gcc-multilib g++-multilib
   ```

2. **Install Homebrew:** The latest installation instructions will always be on the [homepage](https://brew.sh). Homebrew runs on macOS and Linux, but can be installed on Windows 10 using WSL.

   Currently, you can install Homebrew with:
   ```bash
   $ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

3. **Install KLEE:** You can now install KLEE with

   ```bash
   $ brew install klee
   ```

   This will install a pre-built binary package on macOS, but builds KLEE from source on Linux. Pull requests to improve or update the Homebrew KLEE package are welcome at the [Homebrew Core repository](https://github.com/Homebrew/homebrew-core).

4. **You're ready to go! Check the [Documentation]({{site.baseurl}}/docs) page to try KLEE.**

**NOTE:** For testing real applications (e.g. Coreutils), you may need to increase your system's open file limit (ulimit -n). Something between 10000 and 999999 should work. In most cases, the hard limit will have to be increased first, so it is best to directly edit the corresponding configuration file (e.g., `/etc/security/limits.conf`).<br/><br/>
