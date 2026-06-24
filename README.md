# LAGO Abdijkaai Reservaties — Hermes Skill

A shareable Hermes Agent skill for checking availability and safely preparing/completing free family-pass swimming reservations at **LAGO Kortrijk Abdijkaai**.

## What it does

- Checks free online spots for LAGO Kortrijk Abdijkaai.
- Guides Hermes through the Enviso/LAGO ticketshop.
- Uses the free reservation product:
  `Reservatie | zwembeurt met familiepas, promo pas, beurtenkaart, abonnement`.
- Enforces safety rules:
  - no online payment
  - no paid tickets unless explicitly handled outside this skill
  - verify date, time, quantity, and free basket total
  - report a real confirmation marker before claiming success

## Install

Copy this folder into your Hermes skills directory:

```bash
mkdir -p ~/.hermes/skills/personal
cp -R lago-abdijkaai-reservaties-skill ~/.hermes/skills/personal/lago-abdijkaai-reservaties
```

Then reload skills or start a new Hermes session:

```text
/reload-skills
```

You can also start Hermes with the skill preloaded:

```bash
hermes -s lago-abdijkaai-reservaties
```

## Example prompts

```text
Check hoeveel vrije plaatsen er vandaag tussen nu en 19u zijn in LAGO Abdijkaai.
```

```text
Reserveer 3 plaatsen voor LAGO Abdijkaai om 18:00 met de familiepas. Geen online betaling doen.
```

## Privacy and security

This repository should not contain:

- API keys
- cookies
- passwords
- personal contact details
- real order numbers, unless deliberately documented as examples and anonymized

The skill tells Hermes not to persist or expose the Enviso API key found in page context.

## License

MIT
