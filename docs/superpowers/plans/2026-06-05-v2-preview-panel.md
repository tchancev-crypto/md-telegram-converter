# V2 Preview Panel Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the output textarea with a rendered HTML preview + collapsible raw HTML block, and split "Copy" into "Copy formatted" and "Copy HTML".

**Architecture:** Single file edit — `index.html`. Two tasks: (1) HTML markup + CSS, (2) JavaScript handlers. The `convert()` function is untouched. The preview renders newlines as `<br>` for display while `output.value` keeps raw `\n` for the bot API.

**Tech Stack:** HTML5, CSS3, vanilla JS. No dependencies.

---

## File Structure

Only one file changes:
- Modify: `/Users/ilya/Documents/md-telegram-converter/index.html`

---

### Task 1: HTML markup + CSS

**Files:**
- Modify: `/Users/ilya/Documents/md-telegram-converter/index.html`

Changes: replace right pane content, update button markup, update CSS.

- [ ] **Step 1: Replace the right pane markup**

Find and replace this block (lines 114–117):

```html
    <div class="pane">
      <label for="output">Output (HTML)</label>
      <textarea id="output" readonly placeholder="HTML output will appear here..."></textarea>
    </div>
```

Replace with:

```html
    <div class="pane">
      <label>Preview</label>
      <div id="preview"></div>
      <details>
        <summary>HTML</summary>
        <textarea id="output" readonly></textarea>
      </details>
    </div>
```

- [ ] **Step 2: Replace the buttons**

Find and replace (lines 121–123):

```html
  <div class="actions">
    <button id="convertBtn">Convert</button>
    <button id="copyBtn">Copy</button>
  </div>
```

Replace with:

```html
  <div class="actions">
    <button id="convertBtn">Convert</button>
    <button id="copyFormattedBtn">Copy formatted</button>
    <button id="copyHtmlBtn">Copy HTML</button>
  </div>
```

- [ ] **Step 3: Replace CSS for the right pane and buttons**

Find and replace these CSS rules:

```css
    textarea[readonly] { background: #fafafa; color: #444; }
```

Replace with:

```css
    textarea[readonly] { background: #fafafa; color: #444; }

    #preview {
      height: 320px;
      overflow-y: auto;
      padding: 0.75rem;
      border: 1px solid #ddd;
      border-radius: 8px;
      background: #fafafa;
      color: #444;
      line-height: 1.5;
      font-size: 0.875rem;
    }

    details {
      margin-top: 0.5rem;
      font-size: 0.8rem;
      color: #888;
    }

    details summary { cursor: pointer; }

    details textarea {
      width: 100%;
      height: 120px;
      font-family: 'Menlo', 'Consolas', monospace;
      font-size: 0.8rem;
      margin-top: 0.25rem;
    }
```

Also find and replace the old `#copyBtn` CSS:

```css
    #copyBtn {
      background: #e5e7eb;
      color: #333;
    }
    #copyBtn:hover { background: #d1d5db; }
```

Replace with:

```css
    #copyFormattedBtn, #copyHtmlBtn {
      background: #e5e7eb;
      color: #333;
    }
    #copyFormattedBtn:hover, #copyHtmlBtn:hover { background: #d1d5db; }
```

- [ ] **Step 4: Open in browser and verify structure**

```bash
open /Users/ilya/Documents/md-telegram-converter/index.html
```

Check:
- Right pane shows empty `div#preview` area with a border (same height as the left textarea)
- Below it: collapsed `▶ HTML` disclosure widget
- Click `▶ HTML` → opens a small textarea
- Three buttons: Convert, Copy formatted, Copy HTML

- [ ] **Step 5: Commit**

```bash
cd ~/Documents/md-telegram-converter
git add index.html
git commit -m "feat: v2 replace output textarea with preview panel and two copy buttons"
```

---

### Task 2: JavaScript handlers

**Files:**
- Modify: `/Users/ilya/Documents/md-telegram-converter/index.html`

Changes: update convertBtn handler, remove old copyBtn handler, add copyFormattedBtn and copyHtmlBtn handlers.

- [ ] **Step 1: Update the convertBtn handler**

Find and replace:

```js
    document.getElementById('convertBtn').addEventListener('click', () => {
      const input = document.getElementById('input').value;
      document.getElementById('output').value = convert(input);
    });
```

Replace with:

```js
    document.getElementById('convertBtn').addEventListener('click', () => {
      const html = convert(document.getElementById('input').value);
      document.getElementById('preview').innerHTML = html.replace(/\n/g, '<br>');
      document.getElementById('output').value = html;
    });
```

Note: `html.replace(/\n/g, '<br>')` converts newlines to `<br>` for HTML rendering in the preview div. `output.value` keeps the raw `\n` string — that's what the Telegram Bot API expects.

- [ ] **Step 2: Replace the old copyBtn handler with two new handlers**

Find and remove the entire old copyBtn handler:

```js
    document.getElementById('copyBtn').addEventListener('click', () => {
      const ta = document.getElementById('output');
      const text = ta.value;
      if (!text) return;

      const btn = document.getElementById('copyBtn');

      const onSuccess = () => {
        btn.textContent = 'Copied!';
        setTimeout(() => { btn.textContent = 'Copy'; }, 2000);
      };

      const fallback = () => {
        ta.select();
        document.execCommand('copy');
        onSuccess();
      };

      if (navigator.clipboard) {
        navigator.clipboard.writeText(text).then(onSuccess).catch(fallback);
      } else {
        fallback();
      }
    });
```

Replace it with both new handlers:

```js
    document.getElementById('copyFormattedBtn').addEventListener('click', () => {
      const preview = document.getElementById('preview');
      if (!preview.innerHTML) return;

      const btn = document.getElementById('copyFormattedBtn');
      const range = document.createRange();
      range.selectNode(preview);
      window.getSelection().removeAllRanges();
      window.getSelection().addRange(range);
      document.execCommand('copy');
      window.getSelection().removeAllRanges();
      btn.textContent = 'Copied!';
      setTimeout(() => { btn.textContent = 'Copy formatted'; }, 2000);
    });

    document.getElementById('copyHtmlBtn').addEventListener('click', () => {
      const ta = document.getElementById('output');
      const text = ta.value;
      if (!text) return;

      const btn = document.getElementById('copyHtmlBtn');

      const onSuccess = () => {
        btn.textContent = 'Copied!';
        setTimeout(() => { btn.textContent = 'Copy HTML'; }, 2000);
      };

      const fallback = () => {
        ta.select();
        document.execCommand('copy');
        onSuccess();
      };

      if (navigator.clipboard) {
        navigator.clipboard.writeText(text).then(onSuccess).catch(fallback);
      } else {
        fallback();
      }
    });
```

- [ ] **Step 3: Verify with Node.js that convert() is unchanged**

```bash
cd ~/Documents/md-telegram-converter
node -e "
const fs = require('fs');
const html = fs.readFileSync('index.html', 'utf8');
eval(html.match(/function convert\(text\) \{[\s\S]+?\n    \}/)[0]);
const cases = [
  ['**bold**', '<b>bold</b>'],
  ['*italic*', '<i>italic</i>'],
  ['~~struck~~', '<s>struck</s>'],
  ['\`code\`', '<code>code</code>'],
  ['[x](https://t.me)', '<a href=\"https://t.me\">x</a>'],
  ['line one\n\nline two', 'line one\n\nline two'],
];
let ok = 0;
cases.forEach(([i,e]) => { const r = convert(i); r===e ? ok++ : console.log('FAIL',i,'→',r); });
console.log(ok+'/'+cases.length+' passed');
"
```

Expected output:
```
6/6 passed
```

- [ ] **Step 4: Smoke test in browser**

```bash
open /Users/ilya/Documents/md-telegram-converter/index.html
```

Test sequence:
1. Paste into input: `**Заголовок**\n\nЭто *курсив* и ~~зачёркнутый~~ текст.\nСсылка: [n8n](https://n8n.io)`
2. Click **Convert**
3. Preview shows: bold "Заголовок", blank line, italic "курсив", strikethrough "зачёркнутый", clickable "n8n" link
4. Click `▶ HTML` → reveals raw HTML in textarea
5. Click **Copy formatted** → button shows "Copied!" briefly
6. Open Telegram Desktop, paste → formatting preserved (bold/italic visible)
7. Click **Copy HTML** → button shows "Copied!" briefly
8. Paste in a text editor → shows `<b>Заголовок</b>` etc.
9. Empty input → click Convert → preview is empty → both copy buttons do nothing

- [ ] **Step 5: Commit**

```bash
cd ~/Documents/md-telegram-converter
git add index.html
git commit -m "feat: v2 wire preview and dual copy buttons"
```
