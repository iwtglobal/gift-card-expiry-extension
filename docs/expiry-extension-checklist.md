# Gift Card Expiry and Extension Checklist

Programme ops checks for expiry, grace, and audited extensions. Replace durations with your SKU policy.

## SKU setup

- [ ] Expiry basis documented: sale-based or activation-based (not both silently)
- [ ] Grace duration set and known to POS / offline settle jobs
- [ ] Timezone for printed dates published to resellers
- [ ] Reminder schedule defined if buyer contact exists

## Single-card extension request

1. Verify card status (ACTIVE / GRACE / EXPIRED) and remaining balance.
2. Check `extension_count` against the SKU cap.
3. Collect reason code and supporting ticket / proof reference.
4. Apply new `expiry_at`; write audit with actor and old/new values.
5. Confirm to the requester with the new date in the programme timezone.

## Batch extension (programme or legal)

- [ ] Change ticket with SKU list, day delta, and population filter (unsold / sold-unredeemed)
- [ ] Dry-run count of affected cards
- [ ] Dual approval above your balance or card-count threshold
- [ ] Post-job sample: 20 cards spot-checked for new `expiry_at`
- [ ] Finance notified if liability reporting uses expiry
