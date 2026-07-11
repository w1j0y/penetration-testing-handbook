## Reverse WHOIS by registrant email
```
# viewdns.info/reversewhois/ (free, web UI)
curl "https://api.whoisfreaks.com/v1.0/whois/reverse?apiKey=<key>&email=target@example.com"
```

## Reverse WHOIS by org name
Pivot on the registrar-assigned organization string too — it often surfaces domains that share
infrastructure but use different contact emails.

## Passive DNS — domain to IP history
```
# securitytrails.com/domain/<domain>/history/a
curl "https://www.virustotal.com/api/v3/domains/<domain>/resolutions?limit=40" -H "x-apikey: <VT_API_KEY>"
```

## Passive DNS — reverse IP lookup
```
curl "https://www.virustotal.com/api/v3/ip_addresses/<ip>/resolutions?limit=40" -H "x-apikey: <VT_API_KEY>"
```

## Notes
- filter co-located domains by ASN before treating shared-IP as meaningful — CDN/shared hosting
  (Cloudflare, AWS, Fastly) clusters thousands of unrelated domains on one IP
- free-tier passive DNS APIs usually cap at 40-100 records and ~90 days lookback
- domains registered to the same email/org in a short window is the strongest signal — a single shared
  IP alone is weak
