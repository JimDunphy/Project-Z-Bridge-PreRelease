# Rspamd Analysis and Training Workflow

This guide is for administrators and mail operators using Project Z-Bridge with the classic Zimbra Web Client (ZWC) UI on top of Stalwart.

The feature is designed for one practical job: quickly find messages with interesting spam scores, then export the original message source for Rspamd rule analysis and training.

## What The Feature Adds

When `Preferences -> General -> Other Settings -> Rspamd Analysis` is enabled:

- The reading pane shows an `Rspamd <score>` badge for messages that contain `X-Spamd-Result`.
- Hovering or clicking that badge opens the detailed Rspamd symbol breakdown.
- The message list flag column can show a compact tooltip from `X-Spam-Status`.

Project Z-Bridge also adds source-export actions to mail row context menus:

- `Save Original` for one selected message.
- `Save Originals as Tar` for multiple selected messages.

The hover tooltip intentionally stays short:

```text
SPAM, score=18.42, required=15.00
HAM, score=-2.10, required=15.00
No X-Spam-Status
```

## Why It Helps

Admins often need to review a large Junk folder without opening every message. The list hover gives enough information to spot:

- High-scoring spam that is useful as a positive training sample.
- Low-scoring junk that may need new rules.
- Ham incorrectly moved to Junk.
- Messages that users moved manually, where the original scanner verdict is still useful.

The source-export actions then make it easy to collect `.eml` files for external Rspamd rule writing, testing, or corpus review.

## User Training: Quick Score Review

1. Open ZWC.
2. Go to `Preferences -> General -> Other Settings`.
3. Check `Rspamd Analysis`.
4. Save preferences.
5. Open a mail folder such as `Junk`.
6. Move the mouse over the flag/star area of a row.
7. Read the compact `X-Spam-Status` tooltip.

If a message does not have `X-Spam-Status`, the tooltip shows:

```text
No X-Spam-Status
```

This is expected for some trusted, whitelisted, imported, or locally generated messages.

## User Training: Detailed Message Review

1. Open the message in the reading pane.
2. Look for the `Rspamd <score>` badge in the expanded message header.
3. Hover or click the badge.
4. Review the symbol list, score, required score, and summary.

This detailed view uses `X-Spamd-Result`. It is separate from the message-list tooltip, which uses `X-Spam-Status`.

## Exporting Raw Messages For Training

Single message:

1. Select one message.
2. Right-click the row.
3. Click `Save Original`.
4. The browser downloads a `.eml` file.

Multiple messages:

1. Select multiple message rows.
2. Right-click the selected rows.
3. Click `Save Originals as Tar`.
4. The browser downloads `original-messages-N.tar`.
5. Extract the tar file and use the contained `.eml` files for external tooling.

Example:

```sh
tar -tvf original-messages-11.tar
tar -xf original-messages-11.tar -C rspamd-training-samples/
```

The browser handles local download filename collisions. If `original-messages-11.tar` already exists, browsers such as Chrome normally save the next file as `original-messages-11 (1).tar`, then `original-messages-11 (2).tar`.

## Recommended Admin Workflow

Use this pattern for false-positive and false-negative review:

1. Open `Junk`.
2. Hover rows in the flag column and look for `HAM` or low scores.
3. Multi-select suspicious false positives.
4. Use `Save Originals as Tar`.
5. Review or feed the `.eml` files into your Rspamd analysis workflow.
6. Repeat in `Inbox` for missed spam with high scores or suspicious symbols.

For rule writing, keep the raw `.eml` files outside the browser download folder in a dated working directory, for example:

```text
rspamd-training/
  2026-04-24-junk-false-positive/
  2026-04-24-inbox-missed-spam/
```

## Operational Notes

- `.eml` files contain full original message source, including headers and MIME body parts. Treat them as sensitive data.
- Tar exports intentionally contain only `.eml` files. There is no JSON sidecar or bridge metadata file.
- The compact tooltip depends on `X-Spam-Status`.
- The detailed reading-pane badge depends on `X-Spamd-Result`.
- In conversation rows, the bridge uses the first message id available for the row. If you need a specific message inside a conversation, open or search at message granularity before exporting.
- The Rspamd hover and detailed header UI are per-user and controlled by the `Rspamd Analysis` preference.
- The source-export actions are bridge mail actions. They are documented here because they are part of the Rspamd training workflow.

## Troubleshooting

If the tooltip never appears:

1. Confirm `Rspamd Analysis` is checked in `Preferences -> General`.
2. Hard-refresh the browser tab after saving the preference.
3. Confirm the row has a usable message id.
4. Confirm the message source has `X-Spam-Status`.

If `Save Original` or `Save Originals as Tar` does not download:

1. Confirm the user is still logged in.
2. Try a hard refresh and repeat the right-click action.
3. Check browser download settings.
4. Check bridge logs for `rest message source download` or `rest message source tar download`.
