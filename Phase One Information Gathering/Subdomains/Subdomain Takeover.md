Once a subdomain resolves to third-party infrastructure that's since been deprovisioned, it may be
claimable, letting you serve content on a domain the target still controls in DNS.

## Check the DNS chain
```
dig +trace target.inlanefreight.com
dig target.inlanefreight.com CNAME
dig target.inlanefreight.com NS
```

## Classic CNAME takeover
A CNAME points to a provider (GitHub Pages, Heroku, S3, Azure, etc.) but the resource behind it was
deleted. The provider serves an "unclaimed" page for that hostname until someone registers it there.
```
curl -sI https://target.inlanefreight.com
```
Look for provider-specific unclaimed-resource text in the response body, not just a 404.

## Other classes worth checking
- dangling cloud storage: bucket/static-site name matches the subdomain but no longer exists; create
  the matching bucket/site and serve a marker
- SaaS custom-domain conflict: the subdomain was set as a custom domain on a SaaS platform (helpdesk,
  status page, e-commerce storefront, etc.) and the tenant was removed; claim the same custom domain in
  a new tenant

## Proof discipline
Claim only the minimum needed to prove control: a harmless marker file or a DNS TXT record, not a full
fake site. A provider error page alone that says "unclaimed" is a lead, not proof.

## Duplicate suppression
The same dangling-CNAME class often repeats across dozens of subdomains on one target. Don't file each
one as a separate finding. Group them and note whether any carry extra trust (cookies scoped to the
parent domain, an OAuth redirect allowlist entry, a trusted script source), since that's what actually
changes severity.
