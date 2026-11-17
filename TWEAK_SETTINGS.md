# Adding settings to your tweak (Estr0g3n)

Estr0g3n shows a settings screen for your tweak in its **Tweaks** tab. You don't
write any UI code: ship one plist describing the settings, and read the values
in your tweak.

## 1. The plist

Install it at `/Library/Estr0g3n/Preferences/<your.package.id>.plist`
(in Theos: `layout/Library/Estr0g3n/Preferences/<your.package.id>.plist`).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>title</key>          <string>My Tweak</string>
    <key>package</key>        <string>com.example.mytweak</string>   <!-- your package id -->
    <key>defaults</key>       <string>com.example.mytweak</string>   <!-- preferences domain -->
    <key>notification</key>   <string>com.example.mytweak/changed</string>
    <key>description</key>    <string>Shown next to the tweak in the Tweaks tab.</string>
    <key>items</key>
    <array>
        <dict>
            <key>cell</key>  <string>group</string>
            <key>label</key> <string>General</string>
        </dict>
        <dict>
            <key>cell</key>        <string>switch</string>
            <key>key</key>         <string>enabled</string>
            <key>label</key>       <string>Enabled</string>
            <key>default</key>     <true/>
            <key>description</key> <string>Turn the tweak on or off.</string>
        </dict>
        <dict>
            <key>cell</key>    <string>list</string>
            <key>key</key>     <string>speed</string>
            <key>label</key>   <string>Animation Speed</string>
            <key>titles</key>  <array><string>Slow</string><string>Normal</string><string>Fast</string></array>
            <key>values</key>  <array><real>0.5</real><real>1</real><real>2</real></array>
            <key>default</key> <real>1</real>
        </dict>
        <dict>
            <key>cell</key>   <string>text</string>
            <key>key</key>    <string>greeting</string>
            <key>label</key>  <string>Greeting</string>
            <key>default</key><string>Hello</string>
        </dict>
        <dict>
            <key>cell</key>   <string>button</string>
            <key>label</key>  <string>Restart Home Screen</string>
            <key>action</key> <string>respring</string>
        </dict>
    </array>
</dict>
</plist>
```

| Key | Meaning |
| --- | --- |
| `title` | Name shown in the Tweaks tab and at the top of the screen. Defaults to the file name. |
| `package` | Your package id. If it matches the package that installed your dylib, your tweak's row in the Tweaks tab opens these settings. Defaults to the file name. |
| `defaults` | Preferences domain the values are saved in. Defaults to `package`. |
| `notification` | Darwin notification posted after every change, so your tweak can update live. Optional. |
| `description` | Text shown in the side panel. Optional. |

### Item types (`cell`)

| `cell` | What it does | Keys |
| --- | --- | --- |
| `group` | A heading row | `label`, `description` |
| `switch` | On/Off; saves a boolean | `key`, `label`, `default`, `description` |
| `list` | Pick one; saves the matching entry from `values` | `key`, `label`, `titles`, `values` (defaults to `titles`), `default`, `description` |
| `text` | Type a value with the on-screen keyboard; saves a string | `key`, `label`, `default`, `description`, `secure` (hide the value in the list) |
| `button` | `action` = `respring` restarts the home screen; anything else just posts your `notification` | `label`, `action`, `description` |

## 2. Reading the values in your tweak

Values are saved with `CFPreferences` for the `mobile` user, so they end up in
`/var/mobile/Library/Preferences/<defaults>.plist`.

```objc
static BOOL enabled = YES;

static void loadPrefs(void) {
    CFPreferencesAppSynchronize(CFSTR("com.example.mytweak"));
    CFPropertyListRef v = CFPreferencesCopyAppValue(CFSTR("enabled"), CFSTR("com.example.mytweak"));
    enabled = v ? CFBooleanGetValue(v) : YES;   // fall back to the same default as the plist
    if (v) CFRelease(v);
}

%ctor {
    loadPrefs();
    CFNotificationCenterAddObserver(CFNotificationCenterGetDarwinNotifyCenter(), NULL,
        (CFNotificationCallback)loadPrefs, CFSTR("com.example.mytweak/changed"), NULL,
        CFNotificationSuspensionBehaviorDeliverImmediately);
}
```

The Apple TV home screen (AppleTV.app) runs as `mobile`, so this works as-is for
home-screen tweaks. For a tweak in a process running as root, read the plist file
at `/var/mobile/Library/Preferences/<defaults>.plist` directly instead.
