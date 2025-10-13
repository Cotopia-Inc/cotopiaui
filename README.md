# Cotopia Shop — README

Version: 1.0.0  
Last updated: 2025-10-12

Table of contents
- Overview
- Features
- Tech stack & files
- Project structure
- Quick start (local)
- PayPal integration
  - Client-side Buttons (SDK)
  - Hosted buttons fallback
  - Server-side webhook (optional)
  - Recommended PayPal settings
- Web3 (MetaMask) integration
  - Supported flows
  - Security & network notes
  - Example donation/purchase flow
- Client-side cart & checkout flow
- Carousel, accessibility & responsiveness
- Performance & progressive enhancement
- Security considerations
- Deployment
- Environment variables & configuration
- Testing & debugging
- Extending the project
  - Add product management / admin
  - Add ERC-20 token payments (USDC)
  - Add currency conversion
- Troubleshooting
- License
- Acknowledgements

---

Overview
--------
Cotopia Shop is a lightweight static storefront that demonstrates a small inventory of downloadable templates with PayPal-powered checkout and optional Web3 (MetaMask) wallet connectivity for crypto donations or direct payments. The project contains plain HTML, CSS and vanilla JavaScript (no frameworks), with progressive enhancement: PayPal Buttons are used when available, falling back to hosted PayPal forms embedded per-product.

This repository is intended for small digital goods sellers, templates, and creators who want a simple front-end shop that can be hosted on static hosting (Netlify, Vercel, GitHub Pages) while optionally wiring in server-side webhooks for order verification.

Features
--------
- Static site (HTML/CSS/JS) — no build tools required
- Product carousel with keyboard, touch, and pointer support
- PayPal client-side Buttons (via SDK) for immediate payments
- Fallback to hosted PayPal buttons/forms if SDK not loaded
- Optional server webhook pattern for PayPal order verification & inventory/order tracking
- MetaMask / Web3 wallet connect function for donations and on-chain payments
- Accessible navigation and keyboard-friendly carousel
- Local client-side cart support (localStorage) ready for extension
- Responsive layout, CSS variables, and reduced-motion support

Tech stack & files
------------------
- Plain HTML: `index.html` (root page)
- Styles: `styles.css`
- Client JS: `scripts.js`
- Optional server stub (Node/Express): `server/` (webhook example)
- PayPal SDK (client-side) via script tag (requires client-id)
- Web3 injected provider (MetaMask) via `window.ethereum`

Project structure (example)
---------------------------
- index.html
- styles.css
- scripts.js
- README.md
- server/
  - package.json
  - index.js (Express webhook example)
  - README.md (server instructions)

Quick start (local)
-------------------
1. Clone or copy files to your project directory.
2. Edit `index.html`:
   - Replace PayPal SDK client id placeholder with your production or sandbox client id:
     - Example: `<script src="https://www.paypal.com/sdk/js?client-id=REPLACE_CLIENT_ID&currency=USD" defer></script>`
3. (Optional) Serve locally with a static server. Examples:
   - Python 3: `python -m http.server 8000` then open `http://localhost:8000`
   - Node: `npx serve .` or `npx http-server .`
4. Open the site in a browser and verify:
   - PayPal Buttons render for each product (if SDK loaded).
   - Connect MetaMask button prompts wallet connection (if MetaMask installed).
   - Carousel is keyboard and pointer/touch operable.

PayPal integration
------------------
This project supports two modes of PayPal integration:

1) Client-side PayPal Buttons (recommended)
- Load PayPal SDK in `index.html`:
  - For production: `https://www.paypal.com/sdk/js?client-id=YOUR_CLIENT_ID&currency=USD&intent=capture`
  - For sandbox/testing: `https://www.paypal.com/sdk/js?client-id=YOUR_SANDBOX_CLIENT_ID&currency=USD&intent=capture`
- The `scripts.js` contains `initPayPalButtons()` which:
  - Locates `.product` nodes
  - Reads product info from `data-*` attributes and `.product-price`
  - Renders a PayPal button into each `.paypal-button-container`
  - Creates an order with `actions.order.create()` and captures on `onApprove`
  - Handles `onError` and provides a graceful alert fallback

Notes:
- Use your sandbox client id for testing. Transactions in sandbox will not move real funds.
- Keep `intent=capture` for immediate payments or use `intent=authorize` if you want later capture.
- Use `purchase_units[].reference_id` or `custom_id` for internal product/SKU mapping.

2) Hosted button / HTML form fallback
- Each product includes an embedded PayPal hosted button form (older PayPal flow).
- If the SDK is unavailable, `initPayPalButtons()` will create a fallback "Buy with PayPal" button that submits the hosted form, opening PayPal in the configured target (often `_top`).

Server-side webhook (recommended for order verification)
- Relying on client-side capture only is convenient but not sufficient for secure fulfillment. Implement a server endpoint to:
  - Receive PayPal Webhook events (PAYMENT.CAPTURE.COMPLETED, CHECKOUT.ORDER.APPROVED, etc.)
  - Verify webhook signatures using PayPal's API (transmission verification)
  - Retrieve order details with PayPal API (orders/v2/orders/{id} or payments API)
  - Fulfill the order (send download link to buyer email, mark inventory, create order record)
- Example server stub is provided in `server/index.js`:
  - Use `express.json()` to parse PayPal webhook POSTs
  - Validate webhooks using PayPal verification endpoints or SDK (recommended)

Recommended PayPal settings
- Use REST API client credentials (client id + secret) to verify/call PayPal securely from the server (never put the secret in client-side code).
- Enable Webhooks in your PayPal developer dashboard; configure your server webhook URL and subscribe to relevant events:
  - CHECKOUT.ORDER.APPROVED
  - PAYMENT.CAPTURE.COMPLETED
  - PAYMENT.CAPTURE.DENIED
  - PAYMENT.AUTHORIZATION.CREATED (if using authorize)
- Use the sandbox environment for development and testing. Switch to live client-id and live webhook URL for production.

Web3 (MetaMask) integration
---------------------------
What is included
- `connectMetaMask()` in `index.html` (or `scripts.js`) detects `window.ethereum` and calls `eth_requestAccounts`.
- The project currently demonstrates wallet connectivity and a basic on-page confirmation alert.

Supported flows (examples you may implement)
- Donations (ETH): The simplest flow — user connects wallet and sends ETH to the shop's wallet address with a standard transaction.
- Direct purchase (ETH or ERC-20):
  - Option A (simple): Accept ETH transfers to a recipient address, verify payment off-chain (server monitoring) and deliver product manually or via automated system.
  - Option B (contract-based): Deploy a smart contract for purchase logic, add purchase functions for token payments, and call contract from client using Web3/Ethers. This is more secure and automatable but requires contract development, deployment, and auditing.

Security & network notes
- Never store private keys or sign transactions server-side unless using secure vaults and keys that are not publicly accessible.
- For production purchases on-chain:
  - Use a server to observe on-chain payments (via an indexer, Alchemy, Infura, or a lightweight node) and verify that the transaction contains expected data/value.
  - Confirm the correct network (mainnet vs testnet).
- For token payments (ERC-20), remember to require token approvals and verify token transfer receipts.

Example donation/purchase flow (ETH)
1. Buyer clicks "Connect To Web 3 Wallet" and signs wallet connection.
2. UI shows a "Donate" or "Pay" button with preset amount and recipient address.
3. On click, call `ethereum.request({ method: 'eth_sendTransaction', params: [...] })`.
4. After tx hash is returned, monitor transaction status and confirm receipt (server or client) before delivering product.

Client-side cart & checkout flow
-------------------------------
- The project includes product nodes with `data-*` attributes. To build a cart:
  - Add "Add to cart" buttons that push product data into an in-memory array and persist to `localStorage`.
  - Create a `/cart` UI pane or modal summarizing items, quantity, and total.
  - Allow checkout via PayPal Buttons that use aggregated `purchase_units` (PayPal supports multiple items in a single order).
  - For hosted forms per-product, server-side consolidation may be required (i.e., create an order server-side then redirect to PayPal).
- Security: do not trust client-side totals for fulfillment. Recalculate totals on the server or verify order details via PayPal order ID after capture.

Carousel, accessibility & responsiveness
---------------------------------------
- Carousel supports:
  - Keyboard navigation (ArrowLeft / ArrowRight)
  - Touch and pointer dragging (pointer events + touch fallback)
  - Dynamic responsiveness: visible slides adapt to viewport width
- Accessibility:
  - Skip link for screen readers
  - ARIA attributes for the carousel role/labels and products
  - Focus styles using a CSS variable `--focus`
  - Ensure color contrast: consider swapping the current bright blue/black combination to accessible colors (WCAG AA/AAA). See "Visual & accessibility notes" below.

Performance & progressive enhancement
------------------------------------
- Images use `loading="lazy"` attribute.
- If JS fails or is blocked, the page still renders content and hosted PayPal forms still work.
- For better performance:
  - Optimize images (WebP, properly scaled)
  - Minify CSS/JS for production
  - Use HTTP caching and serve assets from a CDN
  - Defer non-critical third-party scripts where possible

Security considerations
-----------------------
- Never embed PayPal secret keys or server credentials in client files.
- Validate PayPal webhooks server-side with PayPal verification.
- Sanitize any input that will be persisted or used in emails/download links.
- Run integrated third-party scripts (PayPal, Web3 providers) from reputable sources and pin to latest stable versions where possible.
- Use HTTPS in production. Mixed content (HTTP + HTTPS) will break wallet and payment flows.

Deployment
----------
Static hosting
- This site can be deployed as static files:
  - GitHub Pages — push files to `gh-pages` or `main` with appropriate config
  - Netlify / Vercel — drag-and-drop or connect repository
  - S3 + CloudFront — upload static files and configure distribution

Server component (optional)
- If you use webhooks or server-side order verification you will deploy the `server/` component to any Node-capable hosting:
  - Heroku, Railway, Render, Fly, DigitalOcean App Platform, AWS Elastic Beanstalk
- Ensure your server has a public HTTPS URL for PayPal webhooks.

Environment variables & configuration
-------------------------------------
Client-side:
- `PAYPAL_CLIENT_ID` — Replace in `index.html` PayPal SDK script tag (do not store secret).
- `PAYPAL_CURRENCY` — USD by default; change if needed.

Server-side (example)
- `PAYPAL_CLIENT_ID` — (optional) used for server REST calls
- `PAYPAL_CLIENT_SECRET` — used for server REST calls (NEVER in client)
- `PAYPAL_WEBHOOK_ID` — webhook ID for validation
- `NODE_ENV` — production/development
- `PORT` — server port

Testing & debugging
-------------------
- PayPal sandbox:
  - Create a PayPal developer account: https://developer.paypal.com
  - Create sandbox accounts (buyer & merchant)
  - Use the sandbox client id and sandbox test accounts to simulate purchases
- MetaMask:
  - Use MetaMask with a testnet (Goerli / Sepolia) while developing
  - Use faucets to fund test accounts
- Browser console:
  - Watch for PayPal SDK logs and errors
  - Check for CORS or mixed-content errors when webhook endpoints or assets are not served securely

Extending the project
---------------------
Add product management / admin
- Add a small admin JSON file or a protected server endpoint to manage inventory, price and hosted PayPal button ids.
- Secure admin UI behind HTTP basic auth or a server-side auth system.

Add ERC-20 token payments (USDC)
- Use web3/ethers to request approvals and token transfer to recipient (or to a smart contract).
- Verify transfer receipts via a server or on-chain indexer.

Add currency conversion
- Use a 3rd-party rates API (e.g., exchangerate.host, Open Exchange Rates) on the server to compute local currency prices and pass values to PayPal.

Troubleshooting
---------------
- PayPal Buttons not rendering:
  - Make sure you replaced `REPLACE_WITH_YOUR_CLIENT_ID` with a valid client id in the SDK script tag.
  - Check the browser console for SDK load errors and network requests.
  - Confirm no duplicate PayPal SDK tags are present (duplicate tags cause conflicts).
- Hosted button not working:
  - Inspect the embedded form `action` and `hosted_button_id` values.
  - If PayPal returns an error, open the form directly in a new tab to see PayPal diagnostics.
- MetaMask connect fails:
  - Ensure MetaMask extension is installed and the site is served over HTTPS (or localhost).
  - Check for rejected requests if the user denies connection.
- Webhook events not being delivered:
  - Ensure your server accepts POST requests and is reachable over HTTPS from PayPal.
  - Verify webhook signatures with PayPal's verification endpoint and check your webhook dashboard for delivery logs.

Visual & accessibility notes (recommended)
-----------------------------------------
- The provided default CSS uses a strong blue background and black/blue text which may not meet WCAG contrast ratios. Consider:
  - Using a light background with dark text, or ensuring text color contrast ratio >= 4.5:1 for body copy.
  - Making focus outlines more prominent for keyboard users.
  - Ensuring interactive elements (buttons/links) have adequate hit area and visible focus state.
- Provide alt text for all product images (the HTML has alt attributes, review content).

Server webhook stub (example)
-----------------------------
A minimal Express example (see `server/index.js`) should:
- Accept `POST /webhook/paypal`
- Parse JSON body
- Verify webhook event signature with PayPal
- Call PayPal orders API to retrieve order details if needed
- Mark order as fulfilled in your database and send a fulfillment email

Example endpoints:
- POST /webhook/paypal
- POST /create-order (server-side create order if you prefer server-created orders)
- GET /orders/:id (order status)

Troubleshooting webhooks
- Use ngrok or similar to expose local dev server to PayPal while testing
- Check the PayPal developer dashboard "Webhook events" list for delivery status
- Log raw webhook bodies and PayPal verification responses for debugging

License
-------
MIT License — see LICENSE file (or add MIT text)  
Attribution: PayPal SDK and Web3/MetaMask are third-party software with their own usage policies.

Acknowledgements
----------------
- PayPal developer docs: https://developer.paypal.com
- MetaMask / Ethereum docs: https://docs.metamask.io, https://ethereum.org
- Inspiration for styling and interactions derived from modern e-commerce micro-frontends and accessibility best-practices

Contact / Support
-----------------
If you need an enhanced implementation (server order processing, multi-currency, digital-download delivery automation, or smart-contract integration), provide:
- Preferred hosting for server webhook (Heroku, Railway, Vercel Serverless, etc.)
- PayPal client-id(s) (sandbox + live)
- Server language preference (Node/Express example included)
- Preferred crypto token / network (ETH mainnet/testnet, USDC, others)

--- 

End of README.