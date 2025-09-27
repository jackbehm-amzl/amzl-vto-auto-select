// ==UserScript==
// @name         VTO Kiosk: Auto-select
// @namespace    https://example.local
// @version      1.0
// @description  Automatically selects DKY6 from the site dropdown and submits/enters it
// @match        *://*/*
// @author       jackbehm
// @run-at       document-idle
// @grant        none
// ==/UserScript==

(function () {
  'use strict';
  const TARGET_CODE = 'DKY6'; // change here if needed

  // Run only once
  if (window.__autoDKY6_done) return;
  window.__autoDKY6_done = true;

  const until = (fn, {timeout = 15000, interval = 100} = {}) =>
    new Promise((resolve, reject) => {
      const start = Date.now();
      const tick = () => {
        try {
          const val = fn();
          if (val) return resolve(val);
          if (Date.now() - start >= timeout) return reject(new Error('timeout'));
          setTimeout(tick, interval);
        } catch (e) { reject(e); }
      };
      tick();
    });

  const setSelectValue = (sel, code) => {
    if (!sel) return false;
    const opt = Array.from(sel.options || []).find(o =>
      (o.textContent || '').trim() === code ||
      (o.value || '').trim().endsWith(code)
    );
    if (!opt) return false;
    sel.value = opt.value;
    // Fire Angular-friendly events
    ['input', 'change'].forEach(type => {
      sel.dispatchEvent(new Event(type, { bubbles: true }));
    });
    // Nudge with Enter as some flows listen for it
    sel.dispatchEvent(new KeyboardEvent('keydown', { key: 'Enter', code: 'Enter', keyCode: 13, which: 13, bubbles: true }));
    sel.blur?.();
    return true;
  };

  const clickLikelySubmit = () => {
    const btns = Array.from(document.querySelectorAll('button, input[type="button"], input[type="submit"], .btn, .button'));
    const cand = btns.find(b => {
      const t = (b.innerText || b.value || '').toLowerCase();
      return ['start', 'continue', 'next', 'submit', 'go'].some(k => t.includes(k));
    });
    cand?.click();
  };

  (async () => {
    try {
      // If there is a locale selector, prefer English first so labels are predictable
      const locSel = document.querySelector('#localeSelector');
      if (locSel) {
        const enOpt = Array.from(locSel.options || []).find(o =>
          (o.value || '').includes('en-US') || (o.textContent || '').toLowerCase().includes('english')
        );
        if (enOpt) {
          locSel.value = enOpt.value;
          ['input', 'change'].forEach(type => locSel.dispatchEvent(new Event(type, { bubbles: true })));
        }
      }

      // Wait for the site dropdown to exist and be populated
      const siteSel = await until(() => {
        const el = document.querySelector('#siteSelector');
        return el && el.options && el.options.length > 1 ? el : null;
      }, { timeout: 20000, interval: 150 });

      const ok = setSelectValue(siteSel, TARGET_CODE);
      if (!ok) {
        // Fallback: try again briefly in case options are still loading
        await new Promise(r => setTimeout(r, 500));
        setSelectValue(siteSel, TARGET_CODE);
      }

      // If the page expects a button press, try to click it
      clickLikelySubmit();
    } catch (e) {
      // Silent fail to avoid interfering with the page
      // console.warn('[Auto DKY6] failed:', e);
    }
  })();
})();
