# Lullex WiFi Provisioning — Organization Setup

This directory contains the Lullex organization configuration for
`openwisp-wifi-login-pages`. The configuration uses placeholder tokens that
**must be replaced** before deployment.

---

## Placeholder Tokens

`lullex.yml` ships with the following placeholders:

| Placeholder | Where to get it | Required |
|-------------|-----------------|----------|
| `__LULLEX_ORG_UUID__` | OpenWISP Admin → Organizations → *Lullex* → UUID field | Yes |
| `__LULLEX_RADIUS_TOKEN__` | OpenWISP Admin → RADIUS → Organization RADIUS Settings → Token | Yes |

**Never commit real UUIDs or tokens to git.** The placeholders are designed to
be replaced at provisioning/deployment time, not edited in source control.

---

## Provisioning Steps

### 1. Replace UUID

```bash
sed -i 's|__LULLEX_ORG_UUID__|<your-org-uuid-here>|g' \
    organizations/lullex/lullex.yml
```

### 2. Replace RADIUS Token

```bash
sed -i 's|__LULLEX_RADIUS_TOKEN__|<your-radius-token-here>|g' \
    organizations/lullex/lullex.yml
```

### 3. Change `server.host`

Edit `organizations/lullex/lullex.yml` and set `server.host` to your OpenWISP
RADIUS backend URL:

```yaml
server:
  host: "https://radius.your-domain.com"   # production
  # or
  host: "https://api.lab.example.com"      # lab (default shipped value)
```

Alternatively, inject the value at deploy time:

```bash
sed -i 's|https://api.lab.example.com|https://radius.your-domain.com|g' \
    organizations/lullex/lullex.yml
```

### 4. Change Captive Portal Action URLs (Production)

The shipped values in `captive_portal_login_form.action` and
`captive_portal_logout_form.action` point at a **local mock** suitable for
lab/dev only. For production, replace them with your real NAS endpoints:

```yaml
captive_portal_login_form:
  action: "https://nas.your-domain.com/captive-portal/login"   # production

captive_portal_logout_form:
  action: "https://nas.your-domain.com/captive-portal/logout"  # production
```

Also update the `redirurl` value under `additional_fields` to your real
post-login landing page:

```yaml
- name: redirurl
  value: "https://wifi.your-domain.com/lullex/status"
```

### 5. Run `yarn setup`

After all placeholders are replaced:

```bash
yarn install   # first time only
yarn setup
```

This produces:

- `client/configs/lullex.json` — Lullex client configuration
- `client/organizations.json` — list of orgs (now includes `{"slug": "lullex"}`)
- `server/config.json` — server-side config (now includes Lullex)
- `client/assets/lullex/` — copied from `client_assets/`
- `server/assets/lullex/` — copied from `server_assets/`

### 6. Run / Build

Dev server:

```bash
yarn start
```

Production build:

```bash
yarn build
```

---

## Files in this Directory

```
organizations/lullex/
├── lullex.yml                     # main organization config
├── README.md                      # this file
├── client_assets/
│   ├── lullex-logo.svg            # header logo
│   ├── favicon.svg                # browser tab icon
│   ├── index.css                  # Lullex soft-SaaS theme
│   ├── index.js                   # custom client JS (currently empty)
│   ├── facebook.svg               # social icon (unused; reserved)
│   ├── google.svg                 # social icon (unused; reserved)
│   ├── twitter.svg                # social icon (unused; reserved)
│   └── linkedin.svg               # social icon (unused; reserved)
├── server_assets/
│   ├── privacy-en.md              # privacy policy (English)
│   └── terms-en.md                # terms of service (English)
└── translations/
    ├── en.custom.json             # custom English string reference
    └── ar.custom.json             # Arabic placeholder (pending)
```

---

## Arabic / RTL Support

RTL styling is already wired in `client_assets/index.css` under
`[dir="rtl"]` selectors. To enable full Arabic support:

1. Fill in `translations/ar.custom.json` with translations.
2. Create `i18n/ar.po` based on `i18n/en.po` structure.
3. Uncomment the Arabic language entry in `lullex.yml`:

   ```yaml
   languages:
     - text: "English"
       slug: "en"
     - text: "العربية"
       slug: "ar"
   ```

4. Run `yarn setup`.

---

## Security Notes

- **Never commit a real `__LULLEX_RADIUS_TOKEN__` value.** Use a secret
  manager or CI/CD injection.
- **`__LULLEX_ORG_UUID__` is less sensitive but still environment-specific.**
  Treat it as a deployment configuration value.
- The captive portal `action` URLs are public knowledge, but using `https://`
  is required in production.

---

## Troubleshooting

**Q: `yarn setup` reports no errors but `client/configs/lullex.json` is missing.**
A: Verify `organizations/lullex/lullex.yml` exists and is valid YAML. The
filename must match the directory name.

**Q: I see the placeholder tokens in the running app.**
A: You forgot to replace them before `yarn setup`. Re-run the sed commands
above and rebuild.

**Q: How do I revert to the default org?**
A: Lullex is additive — it does not modify or remove the `default` org.
Both coexist after `yarn setup`.
