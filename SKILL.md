---
name: lago-abdijkaai-reservaties
description: "Check availability and prepare or complete free LAGO Kortrijk Abdijkaai family-pass swimming reservations safely."
version: 1.0.0
author: Jeroen Thibaut / Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [lago, swimming, reservations, tickets, family-pass, belgium, kortrijk]
    related_skills: [web-reservation-workflows]
---

# LAGO Abdijkaai reservaties

Use this skill when the user asks to check free spots or make a swimming reservation for **LAGO Kortrijk Abdijkaai**.

Target ticketshop:

```text
https://www.lago.be/nl/kortrijk-abdijkaai/ticketshop?offers=18435
```

Primary ticket type:

```text
Reservatie | zwembeurt met familiepas, promo pas, beurtenkaart, abonnement
```

This ticket is intended for users who already have a family pass / promo pass / punch card / subscription, or who will pay/validate the pass on-site. The online reservation should remain free.

## Safety boundary

Allowed without extra confirmation only if the user has clearly asked to book and all of these are true:

1. The selected location is **LAGO Kortrijk Abdijkaai**.
2. The selected product is exactly the free reservation product:
   `Reservatie | zwembeurt met familiepas, promo pas, beurtenkaart, abonnement`.
3. The user has provided or confirmed:
   - date
   - time slot
   - number of swimmers / reservations
   - contact details required by the form
4. The basket/order total remains **Gratis** / **€0**.
5. No online payment method, payment card, Bancontact, Payconiq, or paid checkout is requested.

If any of these checks fail, stop and ask the user before continuing.

Never enter payment details or approve a paid order.

## Standard availability-check flow

1. Open the ticketshop URL.
2. Select or inspect the ticket item named:
   `Reservatie | zwembeurt met familiepas, promo pas, beurtenkaart, abonnement`.
3. Set quantity to the requested amount; use `1` if the user only asks for availability and does not specify quantity.
4. Continue to `Datum & tijd`.
5. Select or inspect the requested date.
6. Read the available time slots and the remaining-ticket counts shown by the page.
7. For “between now and HH:MM”, filter out slots earlier than the current local time and slots starting at or after the requested cutoff unless the user says otherwise.
8. Report results as a small table:

```text
| Time slot | Free spots |
|---|---:|
| 17:30–18:00 | 51 |
```

If the requested date is disabled or the API/page returns no slots, say that it appears not bookable online and do not invent availability.

## Standard reservation flow

1. Open the ticketshop URL.
2. Choose the free reservation ticket.
3. Set the requested quantity.
4. Continue to `Datum & tijd`.
5. Select the requested date and time slot.
6. Verify:
   - product name
   - quantity
   - date
   - time slot
   - availability count is sufficient
7. Add to basket only if still reversible.
8. In checkout, verify the basket/order total is **Gratis** / **€0**.
9. Fill only user-provided contact details.
10. Submit the final reservation only if the safety boundary above is satisfied.
11. Report confirmation evidence. A visible `Bestelnummer:` is sufficient confirmation evidence.

Without a visible order number, e-ticket link, confirmation text, or confirmation email, do not claim the booking succeeded.

## Browser automation notes

The LAGO ticketshop uses Enviso custom elements and sometimes shadow DOM. Accessibility snapshots may be incomplete. When normal clicks do not work, inspect the DOM from the page context.

Useful recursive DOM helper:

```js
function all(root = document) {
  let arr = [];
  for (const el of root.querySelectorAll('*')) {
    arr.push(el);
    if (el.shadowRoot) arr = arr.concat(all(el.shadowRoot));
  }
  return arr;
}
```

Find the family-pass reservation ticket:

```js
const item = all().find(e =>
  e.tagName === 'ENVISO-TICKET-ITEM' &&
  e.getAttribute('name')?.startsWith('Reservatie | zwembeurt met familiepas')
);

item && {
  name: item.getAttribute('name'),
  ticketId: item.getAttribute('ticket-id'),
  value: item.getAttribute('value'),
  maxTickets: item.getAttribute('max-tickets')
};
```

The ticket has been observed with `ticket-id="41942"`, but do not rely only on the ID; prefer the product name.

Click the quantity increment button when the custom element host click fails:

```js
const wrapper = all().find(e =>
  (e.className || '').toString().includes('enviso-nud-wrapper') &&
  (e.textContent || '').includes('Aantal Reservatie | zwembeurt met familiepas')
);

const incHost = Array.from(wrapper.children).find(e =>
  e.tagName === 'ENVISO-BUTTON' &&
  e.getAttribute('data-testid')?.includes('increment')
);

incHost?.shadowRoot?.querySelector('button')?.click();
```

Then verify the selected quantity from both the input and ticket item if possible.

## Capacity API note

The page stores the Enviso API key in page context as:

```js
drupalSettings.envisoTicketingWidget.apiKey
```

Do **not** copy, print, save, or commit this API key. If checking capacities via API, fetch from inside the browser page context and avoid persisting the key.

Observed endpoint pattern:

```js
fetch('https://api.enviso.io/ticketwidgetapi/v3/salespoints/2717/offers/18435/capacities?includealltimeslots=true&fromdate=YYYY-MM-DD&todate=YYYY-MM-DD&quantity=1', {
  headers: { 'x-api-key': drupalSettings.envisoTicketingWidget.apiKey }
}).then(r => r.json())
```

The response may contain:

```json
{
  "frequency": "PerSlot",
  "capacities": [
    {
      "quantity": 56,
      "totalQuantity": 60,
      "timeslot": {
        "start": "2026-06-25T16:00:00Z",
        "end": "2026-06-25T16:30:00Z"
      }
    }
  ]
}
```

Treat all availability numbers as time-sensitive.

## Date and time selection notes

Date cells have been observed as table cells like:

```html
<td class="enviso-day ..." data-day="6-25" aria-selected="true" aria-disabled="false">25</td>
```

Verify selected date:

```js
all().filter(e =>
  e.tagName === 'TD' && e.getAttribute('aria-selected') === 'true'
).map(e => ({ text: e.textContent.trim(), day: e.getAttribute('data-day') }));
```

Timeslot buttons expose text similar to:

```text
18:00 - 18:30 56 resterende tickets.
```

Read all visible timeslots:

```js
all().filter(e =>
  e.tagName === 'BUTTON' &&
  (e.textContent || '').match(/\d{1,2}:\d{2}\s*-\s*\d{1,2}:\d{2}/)
).map(e => ({
  text: e.textContent.trim().replace(/\s+/g, ' '),
  disabled: e.disabled || e.hasAttribute('disabled'),
  className: e.className?.toString()
}));
```

## Personal-info fields

Only fill personal details explicitly provided by the user. Common fields include:

- `Voornaam*`
- `Achternaam*`
- `E-mail*`
- `E-mail bevestigen*`
- `Land*`
- postal code

Postal code can be inside an `ENVISO-TEXTBOX` shadow-backed field with:

```text
data-testid="enviso-form-item-address.postalCode"
```

Set shadow-backed fields by updating the host and inner input, then dispatch composed `input` and `change` events.

## Final verification checklist

Before reporting success:

- [ ] Correct location: LAGO Kortrijk Abdijkaai.
- [ ] Correct free reservation product selected.
- [ ] Correct quantity selected.
- [ ] Correct date selected.
- [ ] Correct time slot selected.
- [ ] Basket/order total is `Gratis` / `€0`.
- [ ] No online payment requested.
- [ ] Confirmation evidence is visible: `Bestelnummer: ...`, e-ticket link, or equivalent confirmation.

If the browser fails mid-flow, report only the last verified state.
