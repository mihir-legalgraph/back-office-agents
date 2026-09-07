# Screenshots

Eleven images. Six are final; five are illustrative and awaiting live captures.

## Final — real renders of the dummy knowledgebase

Produced by `_working/tests/shots.py`, which drives a headless browser over
`_working/tests/kbB-dashboard.html`. Every counterparty in them is invented and
the data is the 1,722-record dummy KB-B, so they are safe to commit as-is — no
blurring, no real contract data. **Keep these even once connector access
exists; do not replace them with live captures.**

| File | What it shows |
| --- | --- |
| `04-dashboard-tabs.png` | Header and all four tabs |
| `06-source-line.png` | One card, cropped to show its source line |
| `07-reporting.png` | Contract Reporting tab, full |
| `08-requester.png` | Contract Requester tab, full |
| `09-cycle-times.png` | Cycle Times Reporting tab, full |
| `10-vendor.png` | Vendor Management tab, full |

Regenerate after any template change: `cd _working/tests && python shots.py`

## Illustrative — reconstructions, shipping as-is

Produced by `_working/tests/mock_chat.html`, rendered by the snippet at the foot
of this file. They show Claude Desktop, which cannot be captured from the
dashboard file. Each carries an `illustrative` tag.

**These ship as they are.** Standing up a live Agiloft connector requires an
Agiloft licence, which we do not hold, so the surrounding UI is drawn rather
than captured. The figures inside the frames are computed from the same dummy
knowledgebase, so the content is accurate even though the chrome is not a
photograph. Replace them if and when licensed access exists — not before.

| File | What it shows | How faithful |
| --- | --- | --- |
| `01-connector.png` | Cowork **+** menu → Connectors, `agiloft` on | Rebuilt to match a real capture |
| `01b-skill.png` | Settings → Skills with the Upload skill dialog | Rebuilt to match a real capture |
| `02-question.png` | A question being typed | Frame rebuilt to match a real capture |
| `03-answer.png` | The answer, dashboard alongside | Frame rebuilt to match a real capture; **the dashboard pane is a real render** |
| `05-followup.png` | "just the NDAs" re-scoping the dashboard | Frame rebuilt to match a real capture; **dashboard pane is a real render** |

The figures quoted in the chat images are real — computed from the same dummy KB
— so replacing the frame does not change the numbers.

**To replace 02 / 03 / 05**, run the skill against the mock MCP server. No
Agiloft needed, no company data at risk:

    claude mcp add agiloft -- python "<repo>/_working/tests/kb-b-harness/mcp_server.py"

**To replace 01 / 01b**, screenshot your own Settings panes — no data in them.

**If any are ever taken against a real knowledgebase**, check every one for
counterparty names before committing — they appear in chat text as well as in
tables and chart legends.

## Re-rendering the illustrative set

    cd _working/tests && python - <<'EOF'
    import pathlib
    from playwright.sync_api import sync_playwright
    OUT = pathlib.Path("../../legal/agiloft-skill/assets").resolve()
    SHOTS = {"s01":"01-connector.png","s01b":"01b-skill.png","s02":"02-question.png",
             "s03":"03-answer.png","s05":"05-followup.png"}
    with sync_playwright() as p:
        b = p.chromium.launch(channel="chrome")
        pg = b.new_page(viewport={"width":1420,"height":900}, device_scale_factor=2)
        pg.goto(pathlib.Path("mock_chat.html").resolve().as_uri()); pg.wait_for_timeout(600)
        for sid, name in SHOTS.items(): pg.locator("#"+sid).screenshot(path=OUT/name)
        b.close()
    EOF

`01-dashboard.png` is retired — the README hero is `architecture.svg`.
