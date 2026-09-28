# React (Vite) contact form — Formspree alternative with AI spam filtering

Contact form for a Vite + React app, posting JSON to SmartForm AI.

## Setup

1. Get a form ID at https://usesmartform.com/dashboard.
2. Clone, install, configure, run:
   ```bash
   git clone https://github.com/yanghuai123456/smartform-example-vite-react.git
   cd smartform-example-vite-react
   npm install
   cp .env.example .env
   # edit .env → VITE_SMARTFORM_FORM_ID=f_your_real_id
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
    setStatus('Sending…');
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

- `POST {endpoint}/api/v1/f/{form_id}` — JSON or form-data, no API key.
- Response: `{ success, message, submission_id, is_spam, intent, next_url }`.

For the full contract, see https://usesmartform.com/docs.

## Deploy

```bash
npm run build         # static output in ./dist
npx vercel --prod     # or netlify deploy --prod, wrangler pages deploy ./dist
```

Set `VITE_SMARTFORM_FORM_ID` in your hosting dashboard's environment variables.

## License

MIT.
