# Gift Card Expiry and Extension

How a **gift card system** sets expiry rules, what a grace period is for, and how controlled extensions keep unredeemed value from becoming a support fight or an accounting surprise. Written for operators evaluating [EVD System](https://evdsystem.com/) by [MoboGage](https://evdsystem.com/about-mobogage/).

---

## What expiry and extension mean

A **gift card expiry** is the date (or datetime) after which the card should no longer redeem at face value under normal sale rules. An **extension** is a controlled move of that date forward—usually after a customer request, a programme decision, or a legal requirement in a given market.

Expiry is not the same as breakage accounting. The card can be expired for redemption while finance still holds a liability until policy says otherwise. Keep those two clocks labelled clearly in the ledger.

## Why programmes need both

Cards that never expire create open-ended liability. Cards that expire with no notice create angry buyers and reseller chargebacks. A practical middle path is:

- Publish an expiry at sale (or "expires N months after activation").
- Send reminders before the date where you have a channel to the buyer.
- Allow a short grace period for in-flight redemptions.
- Permit extension only through audited roles, with a reason code.

## A state model that has worked

```
ISSUED -> ACTIVE (first redeem or activation)
              |
              +-- past expiry, inside grace --> GRACE (redeem allowed with flag)
              |
              +-- past grace -----------------> EXPIRED (redeem blocked)
              |
              +-- extension approved ---------> ACTIVE (new expiry_at)
              |
              +-- void / refund ---------------> CLOSED
```

**Activation-based expiry** (N months from first use or first balance load) fits reloadable programmes. **Sale-based expiry** (fixed date printed on the pack) fits single-use retail cards. Do not mix both on one SKU without a clear rule in the product sheet.

**Grace period** is usually hours to a few days (**illustrative** range), not months. It exists so a POS that was offline at 23:55 can still settle a redeem that started before midnight.

## Worked example (figures are illustrative)

SKU `GC-50` is a single-use card sold at **50.00** units, expires **12 months after sale**, grace **48 hours**. Card `GC-50-9182` was sold on 2025-03-01, so `expiry_at` is 2026-03-01 00:00 in the programme timezone.

On 2026-03-01 at 18:00 a merchant tries to redeem **50.00**. Status is GRACE (inside 48 hours). Redeem succeeds; the record stores `redeemed_in_grace=true`.

On 2026-03-04 the same card would be EXPIRED. A call-centre agent with role `EXPIRE_EXTEND` may extend by **30 days** (**illustrative**) with reason `customer_request_proof_of_purchase`. New `expiry_at` becomes 2026-04-03. The audit row stores actor, old date, new date, and ticket id.

A bulk marketing extension of **90 days** for an entire unsold warehouse batch is a different operation: it should run as a batch job with a change ticket, not as one-off agent clicks.

## Records to keep

| Field | Why |
|-------|-----|
| `card_id`, `sku` | Identity |
| `sold_at`, `activated_at`, `expiry_at` | Which clock applied |
| `grace_ends_at` | Separate from expiry for POS settle |
| `status` | ACTIVE / GRACE / EXPIRED / CLOSED |
| `extension_count`, `last_extended_by` | Abuse and audit |
| `reason_code` | customer_request, legal, programme, batch |

## Things that go wrong

**Silent expiry.** Buyers who never got a reminder treat expiry as a defect. If you have email or SMS on the order, schedule at least one notice before `expiry_at`.

**Unlimited agent extensions.** Without a cap on `extension_count` or a dual-control rule above a balance threshold, expiry becomes optional and liability never closes.

**Timezone drift.** A card "expiring 1 March" in UTC vs Asia/Calcutta changes who wins a dispute. Store `expiry_at` in UTC and render in the programme's published zone.

A checklist for programme ops is in [docs/expiry-extension-checklist.md](./docs/expiry-extension-checklist.md).

---

## FAQ

**What is gift card expiry?**
It is the point after which normal redemption is blocked, based on sale date, activation date, or a fixed printed date—depending on the SKU rules.

**What is a grace period?**
A short window after expiry where an in-flight or slightly late redeem can still complete, flagged for audit.

**Who should be allowed to extend expiry?**
Only roles with an audit trail, reason codes, and preferably a cap on how many times one card may be extended.

**Does expiry equal accounting breakage?**
Not automatically. Redemption block and liability release can be different dates; label them separately.

**How does this relate to EVD System?**
Gift cards and electronic vouchers share lifecycle ideas—issuance, redeem, expiry—inside MoboGage's EVD System and voucher management work.

---

## Glossary

| Term | Meaning |
|------|---------|
| Expiry | Datetime after which normal redeem is blocked |
| Grace period | Short post-expiry window for late/in-flight redeem |
| Extension | Audited move of `expiry_at` forward |
| Activation-based expiry | Clock starts at first use or load |
| Sale-based expiry | Clock starts when the card is sold |
| Breakage | Accounting treatment of unredeemed value (separate from redeem block) |

## Further reading

- [What is an electronic voucher?](https://evdsystem.com/evd-software/what-is-an-electronic-voucher/) on evdsystem.com
- [Voucher management system (MoboGage)](https://evdsystem.com/voucher-management-system-mobogage/) on evdsystem.com
- Related repo: [gift-card-reloadable-vs-single-use](https://github.com/iwtglobal/gift-card-reloadable-vs-single-use)

## License

MIT. See [LICENSE](./LICENSE). Educational material only, not legal or regulatory advice.
