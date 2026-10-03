# React (Vite) contact form â€?Formspree alternative with AI spam filtering

Contact form for a Vite + React app, posting JSON to SmartForm AI.

## What you're POSTing

The endpoint accepts a standard HTML form POST or JSON via AJAX. Two
kinds of fields:

**Your form fields** â€?`name`, `email`, `message`, whatever you
want. Every non-reserved field lands in your dashboard as a column in
the submissions table.

**Reserved fields** â€?names starting with `_` are interpreted by
the API, not stored:

| Field | Purpose |
|---|---|
| ``_gotcha`` | **Honeypot.** Keep it empty. Hidden from humans via CSS; bots fill it automatically. Any non-empty value silently drops the submission. Add this to every form. |
| ``_hp_email`` / ``_website`` / ``_url`` / ``_phone`` | Honeypot aliases for `_gotcha` (WordPress / WPForms / Contact Form 7 migrations). Same drop semantics. |
| ``_next`` | Same-origin URL to redirect to after a successful submission. Browser POST results in a 302 here. AJAX calls (with `Accept: application/json`) get the same value back as `next_url` in the JSON response. Only http(s) and in-site paths allowed. |
| ``_subject`` | Override the AI-generated email subject line. Max 200 chars; control characters stripped. |
| `X-Gotcha` header | Same as `_gotcha` for JSON requests where you can't add a hidden form field. |

Field names are Formspree-compatible â€?migrating from
`formspree.io/f/{form_id}` requires no renaming.

## Setup

1. Get a form ID at https://usesmartform.com/dashboard.
2. Clone, install, configure, run:
   ```bash
   git clone https://github.com/smartformai/smartform-example-vite-react.git
   cd smartform-example-vite-react
   npm install
   cp .env.example .env
   # edit .env â†?VITE_SMARTFORM_FORM_ID=your_real_id
   npm run dev
   ```
3. Open http://localhost:5173, submit, check your dashboard.

## The form

`src/ContactForm.tsx` is a controlled React component that POSTs JSON to the SmartForm endpoint
and shows an inline status message.

```tsx
import { useState } from 'react';

const FORM_ID = import.meta.env.VITE_SMARTFORM_FORM_ID;
const ENDPOINT = 'https://api.usesmartform.com/api/v1/f';

export function ContactForm() {
  const [status, setStatus] = useState('');
  if (!FORM_ID) return <p>Set VITE_SMARTFORM_FORM_ID in .env first.</p>;

  async function onSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setStatus('Sendingâ€?);
    const data = Object.fromEntries(new FormData(e.currentTarget));
    try {
      const r = await fetch(`${ENDPOINT}/${FORM_ID}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json', 'Accept': 'application/json' },
        body: JSON.stringify(data),
      });
      const body = await r.json();
      setStatus(`Sent! submission_id=${body.submission_id} intent=${body.intent}`);
    } catch (err: any) {
      setStatus(`Error: ${err.message}`);
    }
  }

  return (
    <form onSubmit={onSubmit}>
      <input name="name"  placeholder="Name"  required />
      <input name="email" type="email" placeholder="Email" required />
      <textarea name="message" placeholder="Message" required />
      <input type="text" name="_gotcha" tabIndex={-1} autoComplete="off"
             style={{ position: 'absolute', left: -9999 }} aria-hidden />
      <button type="submit">Send</button>
      <p>{status}</p>
    </form>
  );
}
```

## How the API works

- `POST {endpoint}/api/v1/f/{form_id}` â€?JSON or form-data, no API key.
- Response: `{ success, message, submission_id, is_spam, intent, next_url }`.

For the full contract, see https://usesmartform.com/docs.

## Deploy

```bash
npm run build         # static output in ./dist
npx vercel --prod     # or netlify deploy --prod, wrangler pages deploy ./dist
```

Set `VITE_SMARTFORM_FORM_ID` in your hosting dashboard's environment variables.


## FAQ

### Is there a free tier?

Yes. AI spam filtering is enabled by default on every plan. AI intent
classification and high-value lead detection require a paid plan (Pro
or Business) â€?the dashboard enforces this and returns HTTP 402 if
you try to enable them on a free workspace.

### Do I need an API key?

No. The form posts directly to a public endpoint using only an 8-char
form ID, which is non-enumerable. The example also includes a hidden
`_gotcha` honeypot field so naive bots cannot submit.

### Do I need a backend?
No. The form posts JSON to the public endpoint. The example is a plain React component with inline status, no build server required.

## Related examples
[Vite + Vue 3 contact form](https://github.com/smartformai/smartform-example-vite-vue) | [Angular contact form](https://github.com/smartformai/smartform-example-angular) | [smartform-js SDK](https://github.com/smartformai/smartform-js)


## License

MIT.

