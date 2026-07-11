## Gitleaks
```
gitleaks detect --source . -v
```

## TruffleHog
```
trufflehog git https://github.com/inlanefreight/example-repo.git
```

## GitHub Dorks
```
site:github.com "inlanefreight.com" "api_key" OR "secret_key" OR "password"
```
```
site:github.com "inlanefreight.com" filename:.env OR filename:secrets.json
```
