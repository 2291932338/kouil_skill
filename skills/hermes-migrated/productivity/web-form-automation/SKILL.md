---
name: web-form-automation
description: Automate web forms in a browser, submit them, and return completion screenshots; includes handling headless browser/sandbox issues and fallback HTTP submission analysis.
tags:
  - web
  - browser
  - forms
  - screenshots
  - playwright
---

# Web Form Automation

Use this skill when the user asks to fill out an online form/questionnaire/survey, submit a web page, or provide a screenshot of a completed/submitted page.

## Workflow

1. **Open the page in a real browser first.** Use browser tools if available. If Chrome fails in a container/VM with `No usable sandbox`, relaunch with `--no-sandbox` via Playwright or another browser automation tool.
2. **Inspect the form before submitting.** Identify required fields, field names/IDs, and whether the form asks for personal/sensitive information. If answers require the user's real preferences or private data and are not obvious, ask for the missing answers instead of inventing them.
3. **Fill with consistent, plausible answers only when appropriate.** For anonymous low-stakes surveys where the user simply asks to complete it and no personal facts are required, select internally consistent answers.
4. **Prefer browser-driven submission over direct HTTP posting.** Modern survey sites often require client-side anti-bot state, captcha, nonce/signature, timing, or event tracking. Use direct HTTP only as an analysis fallback, not as the main path unless verified.
5. **Take evidence screenshots.** Capture a filled-form screenshot before submit when useful, and always capture the completion/confirmation page after submit.
6. **Verify the screenshot.** Use OCR/vision or page text to confirm it shows completion/success, not just a filled page or captcha prompt.
7. **Return the screenshot natively.** In Feishu/Lark, include `MEDIA:/absolute/path/to/file` so the image is uploaded inline.

## Playwright fallback pattern

If built-in browser navigation fails or Chrome sandbox errors occur:

```python
from playwright.sync_api import sync_playwright

exe = '/home/ubuntu/.cache/ms-playwright/chromium-1217/chrome-linux64/chrome'  # adjust after locating installed browser
with sync_playwright() as p:
    browser = p.chromium.launch(
        headless=True,
        executable_path=exe,
        args=['--no-sandbox'],
    )
    page = browser.new_page(viewport={'width': 390, 'height': 844})
    page.goto(URL, wait_until='domcontentloaded', timeout=60000)
    # fill/click fields...
    page.screenshot(path='/tmp/form_done.png', full_page=True)
    browser.close()
```

If Playwright reports that the expected executable does not exist, search `~/.cache/ms-playwright/` for installed `chrome` or `chrome-headless-shell` binaries and pass `executable_path` explicitly.

## Pitfalls

- Browser tool auto-launch may fail in containers with `No usable sandbox`; do not stop there—use Playwright with `--no-sandbox`.
- A partial Playwright browser install can leave multiple versions present; `p.chromium.executable_path` may point to a missing new version while an older installed binary works.
- Direct POSTs that return captcha/error codes may still look like successful HTTP 200 responses. Confirm by interpreting response text and/or final page content.
- Do not promise to submit later; submit now if the user requested it and scope is clear.

## References

- `references/wjx-survey-automation.md` — notes from automating a Wenjuanxing/WJX survey, including anti-bot/captcha observations and Playwright workaround.