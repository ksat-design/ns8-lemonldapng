# <img src="https://github.com/ksat-design/ns8-ksatdesign/blob/repomd/ns8/updates/lemonldapng/logo.png" width="5%" alt="NS8 Logo"> NS8 LemonLDAP::NG Module

## 📦 Install

Install the module with:

```bash
add-module ghcr.io/ksat-design/lemonldapng:latest 1
```

Sample output:

```json
{
  "module_id": "lemonldapng1",
  "image_name": "lemonldapng",
  "image_url": "ghcr.io/ksat-design/lemonldapng:latest"
}
```

---

## 📚 Documentation

**Official docs**: [lemonldap-ng.org/documentation/latest](https://lemonldap-ng.org/documentation/latest)  
**Application Integration**: [lemonldap-ng.org/documentation/latest/applications.html](https://lemonldap-ng.org/documentation/latest/applications.html)

---

## 🌐 Portals and DNS Setup

Create DNS records pointing to your server IP, replacing `domain.com` with your configured host:

- `auth.domain.com`
- `manager.domain.com`
- `test1.domain.com`
- `test2.domain.com`
- `lemonldapng.domain.com`

- `auth`: app portal for users
- `manager`: admin portal (requires Domain Admins)

---

## 🔒 LDAP Auto Discovery

LDAP settings are auto-discovered at module start.

- `administrator` and `Domain Admins` can access the manager
- All other LDAP users can access the auth portal

---

## 🎨 Customisation

The `llng` directory stores custom themes and assets.

```bash
runagent -m lemonldapng1
cd llng
```

Available subfolders:

```
./llng/
├── apps
├── auth
├── captcha
├── menutab
├── plugins
├── register
├── template
├── theme
├── backgrounds
└── userdb
```

---

## 💾 Persistent Volumes

| Volume        | Mount Path                    |
|---------------|-------------------------------|
| `etc`         | `/etc/lemonldap-ng`           |
| `var-conf`    | `/var/lib/lemonldap-ng/conf`  |
| `var-psessions` | `/var/lib/lemonldap-ng/sessions` |
| `var-sessions` | `/var/lib/lemonldap-ng/psessions` |
| `nginx`       | `/etc/nginx/sites-enabled`    |

To access:

```bash
runagent -m lemonldapng1
podman exec -ti lemonldapng-app bash
cd /etc/lemonldap-ng
```

---

## ⚙️ Configure

Run:

```bash
api-cli run configure-module --agent module/lemonldapng1 --data - <<EOF
{
  "host": "lemonldapng.domain.com",
  "http2https": true,
  "lets_encrypt": false,
  "ldap_domain": "domain.com"
}
EOF
```

This starts and configures the module and sets up a Traefik virtual host.

---

## 🛠️ Sync Configuration Schema

> ⚠️ **Issue Notice**  
> After reboot or restore, `configure-module` may fail due to outdated schema examples.  
> **We will include this fix in the next update.**

### Temporary Fix Script

Create `sync-lemonldapng-schema-example.sh`:

<details>
<summary>Click to expand script</summary>

```bash
#!/bin/bash

echo "🔍 Scanning for available LemonLDAP-NG modules in /home..."
MODULES=($(ls -d /home/lemonldapng* 2>/dev/null | xargs -n1 basename))

if [ ${#MODULES[@]} -eq 0 ]; then
  echo "❌ No lemonldapng modules found under /home."
  exit 1
fi

echo "📦 Available modules:"
select MODULE in "${MODULES[@]}"; do
  if [[ -n "$MODULE" ]]; then
    echo "✅ Selected module: $MODULE"
    break
  else
    echo "❌ Invalid selection. Please try again."
  fi
done

CONFIG_DIR="/home/$MODULE/.config/actions/configure-module"
GET_CONF_SCRIPT="/home/$MODULE/.config/actions/get-configuration/20read"
SCHEMA="$CONFIG_DIR/validate-input.json"
BACKUP="$SCHEMA.bak"

TMP_JSON=$(mktemp)
CLEAN_JSON=$(mktemp)

echo "🔍 Getting current config from module..."
runagent -m "$MODULE" bash -c "/usr/local/agent/pyenv/bin/python3 $GET_CONF_SCRIPT" > "$TMP_JSON"
jq '{host, http2https, lets_encrypt, ldap_domain, cda_status, saml_status}' "$TMP_JSON" > "$CLEAN_JSON"

echo "📦 Backing up schema..."
runagent -m "$MODULE" bash -c "cp $SCHEMA $BACKUP"

JSON_INLINE=$(cat "$CLEAN_JSON")

echo "🔧 Updating validate-input.json..."
runagent -m "$MODULE" bash <<EOF
tmpfile="$CONFIG_DIR/tmp-validate-input.json"
jq --argjson example '$JSON_INLINE' '.examples = [\$example]' "$SCHEMA" > "\$tmpfile"
mv "\$tmpfile" "$SCHEMA"
EOF

runagent -m "$MODULE" bash -c "jq . $SCHEMA" >/dev/null && echo "✅ Schema updated successfully." || echo "❌ Schema update failed."
rm -f "$TMP_JSON" "$CLEAN_JSON"
```

</details>

Run it:

```bash
chmod +x sync-lemonldapng-schema-example.sh
./sync-lemonldapng-schema-example.sh
```

---

## 📥 Get Current Configuration

```bash
api-cli run get-configuration --agent module/lemonldapng1
```

---

## 🧹 Uninstall

```bash
remove-module --no-preserve lemonldapng1
```

---

## ✉️ Smarthost Discovery

The module uses:

- `bin/discover-smarthost` to populate `smarthost.env`
- `events/smarthost-changed/10reload_services` to reload the service on updates

---

## 🐞 Debugging

- View env vars:

```bash
runagent -m lemonldapng1 env
```

- Run agent shell:

```bash
runagent -m lemonldapng1
```

- View running containers:

```bash
podman ps
```

- Shell into container:

```bash
podman exec -ti lemonldapng-app sh
```

- Container environment:

```bash
podman exec lemonldapng-app env
```

---

## 🧪 Testing

```bash
./test-module.sh <NODE_ADDR> ghcr.io/nethserver/lemonldapng:latest
```
