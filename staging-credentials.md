# Staging Credentials

⚠️ **CHANGE FOR PRODUCTION** - All passwords are `NikonD90` for staging simplicity.

## OPNsense API
```
Host: 192.168.254.254
Key: Xb1BnYBOO+HgrkCsxed0wtsxcwujlNuO5Jpz7ULw7xHsnuxZTEm2JwPRQKTVnqETEmHJdsfDsUBY
Secret: napENL6EtELLFethzlKJpoNb/w+YICoqjqqfOzbgJvk6rNlmsn85XddjgHMpuhgz/lkQGf3j6j4Fthzn
```

## SSH Access
| Host | IP | User | Password |
|------|-----|------|----------|
| OPNsense | 192.168.254.254 | root | NikonD90 |
| Zyxel | 192.168.254.2 | root | NikonD90 |
| ONTI | 192.168.254.3 | root | NikonD90 |
| deep-thought | 192.168.254.101 | root | NikonD90 |
| napster | 192.168.254.102 | root | NikonD90 |

## Web Interfaces
| Service | URL | User | Password |
|---------|-----|------|----------|
| OPNsense | https://192.168.254.254 | root | NikonD90 |
| Proxmox (hal) | https://192.168.254.100:8006 | root | NikonD90 |

## Notes
- All staging passwords: `NikonD90`
- API keys are unique per service
- **Before production:** rotate ALL credentials, store in Vaultwarden
