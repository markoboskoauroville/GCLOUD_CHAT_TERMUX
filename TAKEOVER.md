# GCLOUD_CHAT_TERMUX: takeover

A Termux copy of the Google Cloud part of Queen's CAPI chat (Queen v25): Flask + waitress server,
one web page opened in the phone's browser, Gemini models paid from the GCP free credit of project
antigravity-cloud-offload (base $225.31 on 18.11.2026, expires about 2.12.2026). Command: `gchat`.
Queen (qwen/Groq) is not part of it. Written by Gemini from Marko's Google Cloud chat as a test of
the Google Cloud provider (18.11.2026), checked and corrected by Claude Code.

## How to install (Termux)

    curl -fsSL https://raw.githubusercontent.com/markoboskoauroville/GCLOUD_CHAT_TERMUX/main/install-termux.sh | bash

The Gemini key is asked for on the first run and kept on the phone (~/.config/gchat/key), never in
this repository.

## State
- 18.11.2026: repository created, waiting for Gemini's files.
