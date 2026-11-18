You are building a small Android Termux app called **gchat**: a Flask chat server with one web page, talking to Google's Gemini API. It is the phone copy of the "Google Cloud" chat in my Mac app. Write every file complete, ready to save and run, with no placeholders and no "rest stays the same".

## What it does
- A chat with Gemini models through the Gemini API (`https://generativelanguage.googleapis.com/v1beta`), API key in the `x-goog-api-key` header.
- The model list comes live from `GET /v1beta/models`: only models whose name contains `gemini` and that support `generateContent`; skip tts, image, embedding, robotics and computer-use models. The default is `gemini-3.8-flash` if present, else the first flash model.
- Answers stream into the page (`:streamGenerateContent?alt=sse`), shown as Markdown with code blocks and a Copy button on each code block.
- **Several chats:** a list on the left (on a narrow phone screen it slides in from a button). NEW CHAT, RENAME, CLOSE (delete, after one confirm). The newest chat is on top. Each chat is saved as JSON in `~/.local/share/gchat/chats/` and comes back after a restart. Gemini receives the whole conversation.
- **File picker:** a paperclip button beside the input, plus paste and drag-and-drop. Several files at once, shown as small chips with an x before sending. Images and PDFs go to Gemini as `inline_data` (base64, with their mime type); text and code files are inlined as text. Limit 20 MB per file, with a clear message. On Android the paperclip must open the system picker (`<input type="file" multiple>`, no `accept` restriction that would hide files).
- **Credit line** at the top: "Google Cloud credit: $X left · this chat $Y · expires 2.12.2026". The base is $225.31 on 18.11.2026. Every reply's `usageMetadata` (prompt and candidates tokens) is priced with a small table of USD per million tokens per model (one dict at the top of server.py, easy to edit, unknown models use the flash price), appended to `~/.local/share/gchat/ledger.jsonl` and subtracted from the base. Label it an estimate.
- The input box grows with the text and has a scroll bar. Enter sends, Shift+Enter makes a new line. A STOP button cancels a running answer.
- Look: dark, plain, monospace labels in capitals, no emoji, no alarm symbols. Readable on a 6-inch portrait phone screen first, with margins on all sides; also fine on a large screen.

## The key
Never in the code or the repository. Read it from the `GEMINI_API_KEY` environment variable, else from `~/.config/gchat/key` (file mode 600). If neither exists, the first run of `gchat` asks for it in the terminal (input hidden), saves it there and goes on. The page never shows the key; logs show at most the first 4 characters.

## Files (exactly these)
1. `server.py`: Flask app served by **waitress**, falling back to the Flask dev server with a printed warning if waitress is missing. Port: prefer 8850, else the next 15, else any free port. It listens on 127.0.0.1 only. Routes: `/` (the page), `/api/models`, `/api/chats` (list, create, rename, delete), `/api/chat/<id>` (get), `/api/send` (POST: chat id, model, text, files as base64 → SSE stream), `/api/stop`, `/api/credit`. Errors from Gemini come back as one plain line in the chat, never a stack trace.
2. `index.html`: the whole page (HTML, CSS, JS in one file, no CDN, works offline except for the API calls). A small Markdown renderer written inline.
3. `install-termux.sh`: the installer, run by `curl -fsSL https://raw.githubusercontent.com/markoboskoauroville/GCLOUD_CHAT_TERMUX/main/install-termux.sh | bash`. It must:
   - start with `#!/data/data/com.termux/files/usr/bin/bash` and `set -e`; use colour only when `[ -t 1 ]`;
   - print a dependency table first (`python`, `git`, `pip` packages flask, waitress, requests), each `ok` or `MISSING`;
   - `pkg install -y python git` if missing, then `pip install flask waitress requests` (no venv: Termux pip installs directly);
   - clone the repo into `~/GCLOUD_CHAT_TERMUX`, or `git pull --ff-only` if it is already there;
   - write the command `$PREFIX/bin/gchat` by writing `gchat.new` and renaming it over the old one (never `cat >` over a running script), and remove orphaned `.new` files first;
   - work when piped through `bash`: any question to the user reads from `/dev/tty`, not stdin;
   - end with plain lines: how to start (`gchat`), and that `termux-setup-storage` is needed once, by hand, only for saving files.
4. `gchat`: the launcher (bash). It asks for the key if missing; runs `termux-wake-lock` (with `2>/dev/null || true`); starts the server; opens the page by trying in order `am start -a android.intent.action.VIEW -d <url>`, `termux-open-url`, `xdg-open`/`open`; then shows a **raw single-key console** in the terminal:
   `o` open the page again · `m` show models · `k` change the key · `l` show the last 20 log lines · `u` update (`git pull --ff-only` in the repo, then restart the server) · `r` restart the server · `q` quit.
   Read keys with `stty raw -echo` / `dd bs=1 count=1`. Save `stty -g` first and **arm the trap BEFORE the first stty**, restoring the terminal and stopping the server on EXIT, INT, TERM and HUP; `termux-wake-unlock` on the way out. Print the URL and the single-key menu on one short screen.
5. `version.py`: `VERSION = 1`. The page footer and the console show it. Versions are whole numbers.

## Before you answer
Check every file against this list, then answer with each file in its own fenced code block, with its file name on the line just above the block. Nothing else between the blocks except one line per file saying what it is.
