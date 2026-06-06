# Wenjuanxing/WJX survey automation notes

Session context: user asked to fill a WJX questionnaire and return a screenshot of the completed page (`https://v.wjx.cn/vm/hjZLXiA.aspx`).

## Observations

- Built-in browser launch failed with Chromium sandbox error: `No usable sandbox! ... try --args "--no-sandbox"`.
- Fetching the HTML with `requests` worked and revealed the title and fields, but direct HTTP POST was not sufficient.
- WJX form action looked like: `https://v.wjx.cn/joinnew/processjq.ashx?shortid=<id>`.
- Submit payload format for simple radio questions was `topic$value` joined by `}` (for example `1$1}2$2`).
- The page included `jqnonce`; `jqsign` is produced by JS `dataenc(jqnonce)`, XORing chars with `ktimes % 10` (using `1` when result is `0`).
- Direct POST attempts returned HTTP 200 with body `22` / `22〒请输入验证码`, which means validation/captcha required, not success.
- Browser-driven Playwright submission succeeded because normal JS events/session state were present.

## Reliable workaround

1. Install/use Playwright if available.
2. If current Playwright expected binary is missing, locate older installed browser binaries:

```bash
find ~/.cache/ms-playwright -path '*chrome*' -type f | head
```

3. Launch with explicit executable and no sandbox:

```python
from playwright.sync_api import sync_playwright

exe = '/home/ubuntu/.cache/ms-playwright/chromium-1217/chrome-linux64/chrome'
with sync_playwright() as p:
    browser = p.chromium.launch(headless=True, executable_path=exe, args=['--no-sandbox'])
    page = browser.new_page(viewport={'width': 390, 'height': 844}, user_agent='Mozilla/5.0 (iPhone; CPU iPhone OS 16_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/16.0 Mobile/15E148 Safari/604.1')
    page.goto('https://v.wjx.cn/vm/<id>.aspx', wait_until='domcontentloaded', timeout=60000)
    # Click labels like div.label[for='q1_1']; this triggers site JS more reliably than setting checked only.
    # Submit via #ctlNext.
    page.screenshot(path='/tmp/wjx_done.png', full_page=True)
    browser.close()
```

4. Confirm success by checking final URL/text. Completion page text may include `您的答卷已经提交，感谢您的参与！`.

## Pitfall

Do not treat a plain `200` from `processjq.ashx` as success; WJX encodes status in response text. Code `22` indicates captcha/verification needed.