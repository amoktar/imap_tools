Export correspondents (contacts)
========================================================================================================================

This project can scan your IMAP mailbox and build a list of unique email addresses from messages you received (``From``) and sent (``To`` / ``Cc`` / ``Bcc``). Output is CSV or JSON.

Quick start (CLI)
------------------------------------------------------------------------------------------------------------------------

1. Install this repo so Python can import ``imap_tools`` (once per environment, from the repo root)::

       pip install -e .

   Alternatively: ``pip install imap-tools`` from PyPI if you do not need local changes.

2. Copy ``.env.example`` to ``.env`` in the repo root and set:

   * ``IMAP_HOST`` — IMAP server hostname
   * ``IMAP_PORT`` — usually ``993`` (IMAP over SSL)
   * ``IMAP_USER`` — login / email address
   * ``IMAP_PASSWORD`` — password or app-specific token
   * ``IMAP_OUTPUT`` — output file (e.g. ``contacts.csv``)
   * ``IMAP_FOLDERS`` (optional) — comma-separated folder names; if omitted, all folders except Trash/Junk-style names are scanned

3. From the repository root (so ``.env`` is found automatically)::

       python examples/export_contacts.py

4. When finished, the script prints how many unique addresses were written and the output path (default: ``contacts.csv`` in the current directory).

CLI options
------------------------------------------------------------------------------------------------------------------------

Run ``python examples/export_contacts.py --help`` for the full list. Common flags:

* ``--host``, ``--user``, ``--password``, ``--port`` — override ``IMAP_*`` env vars
* ``-o`` / ``--output`` — ``contacts.csv`` or ``contacts.json``
* ``--folders INBOX,Sent`` — limit which folders are scanned
* ``--exclude a@b.com,c@d.com`` — skip addresses (your login is excluded by default)
* ``--no-skip-noise`` — include noreply / mailer-daemon style addresses
* ``--env-file path/to/.env`` — use a specific env file

Configuration priority for each setting: CLI flag → environment variable → values loaded from ``.env``.

Example with explicit flags (no ``.env``)::

    python examples/export_contacts.py ^
        --host imap.example.com ^
        --user you@example.com ^
        --password your-app-password ^
        -o contacts.csv

On Unix/macOS, use ``\`` instead of ``^`` for line continuation.

Output columns (CSV)
------------------------------------------------------------------------------------------------------------------------

* ``email`` — normalized address
* ``name`` — display name when present
* ``inbound_count`` / ``outbound_count`` — how often the address appeared in each direction
* ``folders`` — semicolon-separated folder names where the address was seen
* ``first_seen`` / ``last_seen`` — ISO timestamps from message dates

JSON export uses the same fields; ``folders`` is a JSON array.

Python API
------------------------------------------------------------------------------------------------------------------------

.. code-block:: python

    from imap_tools import MailBox, collect_contacts, write_contacts_csv

    with MailBox('imap.example.com').login('you@example.com', 'password') as mailbox:
        contacts = collect_contacts(
            mailbox,
            exclude_emails=['you@example.com'],
            mark_seen=False,
        )
        write_contacts_csv('contacts.csv', contacts)

See ``imap_tools/contacts.py`` and ``examples/export_contacts.py`` for ``write_contacts_json``, folder filters, and other options.

Notes
------------------------------------------------------------------------------------------------------------------------

* Messages are fetched with ``headers_only=True`` and ``mark_seen=False`` in the CLI so mail is not marked read and bodies are not downloaded.
* Large mailboxes can take a long time; use ``--folders`` to narrow the scan.
* Do not commit ``.env``; it contains credentials.
