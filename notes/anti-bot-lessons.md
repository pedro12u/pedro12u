# What I learned automating anti-bot-protected forms

I build automation that has to work in production against systems actively trying to block bots. The wins are boring; the failures are where the real lessons are. Here are a few that changed how I approach anti-bot work.

## 1. It's the IP, not the browser

I once spent days perfecting browser stealth - fingerprints, WebRTC, canvas, the works. The captcha still failed. The breakthrough came from a boring test: same code, different network.

- Datacenter IP → captcha solved perfectly, token rejected server-side.
- Residential IP → same automation passed without even showing a challenge.

For score-based anti-bot, IP reputation often dominates browser fingerprint. A perfectly solved captcha from a flagged IP is worthless. Fix the IP before you touch the fingerprint.

## 2. The rendered DOM lies - read the raw HTML

A form looked like it had one captcha. I solved it; the submit was silently rejected, every time, with no error.

So I stopped trusting the browser and curl'd the raw HTML. There was a second captcha, hidden by the page's JavaScript. The server required both - the browser was hiding half the problem from me.

When a browser flow mysteriously fails: diff the raw HTML against the rendered DOM. JS hides fields, injects client-side gates, and lies about what the server actually checks.

## 3. Sometimes the browser is the problem - drop to raw HTTP

That same form had a client-side validation gate that blocked any programmatic submit. Instead of fighting the page's JavaScript, I threw the browser away and replayed the whole flow as raw HTTP: session cookie → fetch captcha assets → solve → POST.

Result: faster, more reliable, and no headless browser to detect. The browser is a tool, not a requirement. Once you understand the request flow, raw HTTP usually beats fighting the DOM.

## 4. Reverse the internal API to kill UI fragility

Behind most modern portals is a clean JSON API and a session token issued right after the captcha passes. Instead of scraping brittle rendered pages, I solve the human gate once, capture the token, and talk to the API directly - structured data, zero DOM selectors, far less to break.

## The meta-lesson

There's no permanent bypass - anti-bot is an arms race. The real skill isn't a magic trick; it's a disciplined diagnostic loop: capture the actual request, find what the server truly validates, and attack that, not what the page shows you.

— Pedro Toscano · Automation & Anti-Bot Engineer
