Source: Bugcrowd blog

[Bugcrowd - Access Control vs. Account Takeover](https://www.bugcrowd.com/blog/access-control-vs-account-takeover-what-bug-bounty-hunters-need-to-know/)
date_read: 2026-07-28

---

# Access Control vs. Account Takeover

## Summary

Account Takeover (ATO) and access-control vulnerabilities are often
conflated but are mechanically different. ATO is about gaining control
of *another user's account* through authentication or session
weaknesses. Access control is about performing actions or accessing
data the *current, authenticated* user shouldn't be allowed to — a
separate question from who you're logged in as.

## Key lessons

- ATO is mainly related to authentication and account/session control.
- Access control is mainly about authorization *after* authentication.
- IDOR/BOLA are the most common forms of broken access control.
- Impact — not just the mechanism — is what determines severity and how
a finding should be reported.

## New things learned

- A single access-control flaw can escalate into ATO if it allows
sensitive account info or account-management functions to be changed
(e.g. an IDOR on a "change email" endpoint isn't just an IDOR anymore).
- A vulnerability doesn't need to reach full account takeover to be a
valid, reportable issue.
- Testing needs to explicitly separate the authentication check from
the authorization check — they're different failure modes, and
conflating them muddies both the finding and the report.

## How this applies to hunting

Two separate questions to ask when testing an authenticated application:

```
Can I become/access another account?
        ↓
Authentication / ATO

Can I access something my current account shouldn't?
        ↓
Authorization / Access Control
```

Naming which question a finding actually answers is what makes impact
accurate in a report — conflating the two either overstates or
understates severity.

## Testing methodology

- Test whether User A can access User B's resources.
- Test whether a lower-privileged user can perform higher-privileged
actions.
- Check whether account-management functions properly verify
authorization, not just authentication (i.e. does the app confirm
you're logged in *and* that you're allowed to change *this* account).

## Notes

The core distinction: IDOR does not automatically mean ATO. An IDOR may
only expose or modify a single resource; ATO requires gaining control of
the victim's account itself. Real impact depends entirely on what the
broken authorization actually allows — a photo-view IDOR and an
email-change IDOR are the same *mechanism* with wildly different
*severity*.

## Related

- `authentication/` — once built out, cross-link the ATO side of this
distinction there instead of duplicating it.
