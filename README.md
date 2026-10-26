# Workflows-Blocklist

Runtime control lists for Mann Consulting's Jamf Pro workflows. Mann's scripts read these files from this repo on every policy run, so a change here takes effect fleet-wide on the next run with no changes needed in Jamf Pro.

| File | What it does |
|------|--------------|
| `blocklist` | Application labels that Mann has temporarily paused from installing or updating, usually because the current vendor release is broken. |
| `unsubscribelist` | Jamf Pro servers whose Mann Jamf Maintenance subscription has ended. Policies on those servers stop running and remote logging is disabled. |

## blocklist

One application label per line. Add a label to pause it everywhere; remove it once a good release ships.

## unsubscribelist

One SHA-256 hash per line. Each hash is the full Jamf Pro server URL of an unsubscribed client. Only the hash is stored, so client names are never exposed in this public repo.

### Adding a server

Hash the full HTTPS URL **with the trailing slash** and append the result:

```zsh
echo -n "https://example.jamfcloud.com/" | shasum -a 256 | awk '{print $1}' >> unsubscribelist
```

The URL must match exactly what the client's Macs have stored for their Jamf Pro server, so keep the `https://` and the trailing `/`. The `-n` on `echo` is required. Commit and push with a generic message such as `Add new entry to unsubscribelist`.

### Removing a server

Delete the hash line, then commit and push with a generic message such as `Remove entry from unsubscribelist`.

## My policies stopped with a subscription warning

If your Jamf Pro policy logs show a warning that your server is no longer subscribed to Mann's Jamf Maintenance, your server is on the unsubscribe list. You have three options.

**Resubscribe (recommended).** Email **support@mann.com**. Once your subscription is active again your server is removed from the list and policies resume on their next run.

**Override the check.** You can keep running the workflows without a subscription. Mann strongly advises against this: the scripts will no longer receive updates, so a change from Apple or an application vendor can break them without warning, and remote logging to Mann is disabled so Mann cannot see or help with failures. If you accept that risk, use either method below.

*Per policy, with a script parameter:*

1. In Jamf Pro, open each policy that runs a Mann workflow script.
2. Under **Scripts**, find **Parameter 11**.
3. Add `skipsubscriptioncheck=yes`. If the parameter already has a value, append it with a semicolon, for example `NOTIFY=silent;skipsubscriptioncheck=yes`.
4. Save the policy.

*Fleet-wide, with a configuration profile:*

Deploy a configuration profile with an **Application & Custom Settings** payload for the preference domain `com.mann.Workflows` that sets `SKIPSUBSCRIPTIONCHECK` to `true`. Scope it to all computers that run Mann workflows. This covers every policy at once and survives policy edits.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>PayloadContent</key>
    <array>
        <dict>
            <key>PayloadType</key>
            <string>com.mann.Workflows</string>
            <key>PayloadIdentifier</key>
            <string>com.mann.Workflows.skipsubscriptioncheck</string>
            <key>PayloadUUID</key>
            <string>1D4A9C2E-6B7F-4E3A-9C1D-2F8B5A7E6C40</string>
            <key>PayloadVersion</key>
            <integer>1</integer>
            <key>PayloadDisplayName</key>
            <string>Mann Workflows</string>
            <key>SKIPSUBSCRIPTIONCHECK</key>
            <true/>
        </dict>
    </array>
    <key>PayloadDisplayName</key>
    <string>Mann Workflows - Skip Subscription Check</string>
    <key>PayloadIdentifier</key>
    <string>com.mann.Workflows.skipsubscriptioncheck.profile</string>
    <key>PayloadOrganization</key>
    <string>Your Organization</string>
    <key>PayloadScope</key>
    <string>System</string>
    <key>PayloadType</key>
    <string>Configuration</string>
    <key>PayloadUUID</key>
    <string>7E2F5B1A-3C9D-4A8E-B6F0-5D1C8E9A2B73</string>
    <key>PayloadVersion</key>
    <integer>1</integer>
</dict>
</plist>
```

Upload the file under **Computers > Configuration Profiles > Upload**, or build the same payload in Jamf Pro under **Application & Custom Settings**. If you already deploy the Mann Workflows Configuration profile from its JSON schema, the key appears there as **Skip subscription check**; set it to Yes.

**Stop using the workflows.** Disable or delete the policies. This is the safest choice if you will not maintain the scripts yourself.

## An app says it is blocklisted

The application's current release has a problem that makes it unsafe to install, and Mann has paused it for every client until the vendor ships a fix. No action is needed. If you need it installed regardless, email **support@mann.com**.
