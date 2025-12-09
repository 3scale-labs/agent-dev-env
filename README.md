# Agent Dev Environment

[![Tests](https://github.com/3scale-labs/agent-dev-env/actions/workflows/tests.yml/badge.svg)](https://github.com/3scale-labs/agent-dev-env/actions/workflows/tests.yml)

# Goal
The goal is to easily start your coding agent within a secure sandbox environment that closely resembles your actual development environment and safely merge the results back.

# Motivation

When running an AI coding agent, it is a great time sync to superwise it closely and approve each command it runs and each edit it performs. That makes you wait for the answer most of the time, trying to multi-task is usually more counterproductive than helping. It is far more productive to spend some time producing a plan, then let the AI agent research, implement, run tests unsuperwised so later you can come back to validate what it came up with.

But allowing a non-deterministic program that reads code and information from the internet and based on this running things locally is also highly insecure.

There are existing attempts to sandbox agent processes, the most serious I find being [sandbox-runtime](https://github.com/anthropic-experimental/sandbox-runtime) but as of the time of writing, it has 2 critical deficiencies for me:
* user's home directory is only proteccted by a list of blacklisted paths which leaves a lot of room for missing to secure your informantion
* network isolation doesn't allow the process inside to connect to databases, the HTTP proxy only allows http connections, and not having access to your development databases makes it impossible to allow the sandboxed agent to test it's ramblings proprly

# Usage

* requires Bubblewrap 0.11+ for the overlayfs support
* add the script(s) to your PATH
* `mkdir -p ~/aihome`
* agent-dev bash
* claude init
* exit

Now you can just call the script from within a git working tree and feel the false sense of security you came here for.

# Implementation concerns

* it operates on a selected directory `~/aihome` as the agent's writable home directory
* which is mounted read-write at the same path as the original user's home
* the directory you are calling the script from is mounted read-write inside the sandbox at it's original path
* the operating system important paths are mounted as read-only so you have access to all installed tools and libraries
* also `asdf`, `npm` and other directories from user's real home are monuted read-only inside the sandbox home so that you have access to these installations without ability to install, remove and modify them
* note: *this means you need to install all necessary libraries from the host*

The biggest concern I see is merging the changes after exiting the agent. One has to be careful with merging files that are a potential treat outside the sanbox.

A lesser but potentially important in some environments is the cocern that a malware may set roots into the sandbox and use local resources or network access for malicious purposes. On signs of weird behavior, I'd suggest investigating and wiping the agent home dir to start over.

## Running Tests

```bash
# Install expect if not already installed
sudo apt install expect  # Debian/Ubuntu
# or
sudo dnf install expect  # Fedora/RHEL

# Run all tests
./tests/test-merge-overlay.expect  # Test the merge workflow
./tests/test-agent-dev.expect      # Test isolation and sandbox behavior
```
