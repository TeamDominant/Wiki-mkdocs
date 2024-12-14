# Betterfox - Firefox user.js for speed, privacy, and security. Your favorite browser, but better.
31% [faster](https://medium.com/@mihirgrand/comparing-popular-firefox-forks-6fa83fdfdaad#:~:text=31%25%20more%20than%20vanilla%20Firefox) than regular Firefox 

## Required reading
!!! note
    Create a [backup profile](https://github.com/yokoffing/Betterfox/wiki/Backup), if you feel unsure about the changes.

If you don't have it already: [Get Firefox](https://www.mozilla.org/en-US/firefox/all/#product-desktop-release)

## Installation
1. Download the user.js file [here](https://raw.githubusercontent.com/TeamDominant/Betterfox/main/user.js) (Right click > `Save Link As…`).
2. Review [Common Overrides](https://github.com/yokoffing/Betterfox/wiki/Common-Overrides) and make any necessary changes.
   * See also [Optional Hardening](https://github.com/yokoffing/Betterfox/wiki/Optional-Hardening) for other recommendations.
   * Do not change [Recommended Overrides](https://github.com/TeamDominant/wiki/Recommended-Overrides) as there is no need to do so, create an [Issue](https://github.com/TeamDominant/Betterfox/issues/) if I'm wrong.
3. Open Firefox. In the URL bar, type `about:profiles` and press `Enter`.
4. For the profile you want to use (or use default), click `Open Folder` in the **Root Directory** section.
5. Close Firefox.
6. Move the `user.js` file into the folder.
7. Open Firefox and make changes in settings if you need.  

*After restarting Firefox:* 

1. Get an **ad blocker** like [uBlock Origin](https://addons.mozilla.org/blog/ublock-origin-everything-you-need-to-know-about-the-ad-blocker/) with our [recommended filters](https://github.com/yokoffing/filterlists#guidelines).
2. Go to `C:\Program Files\Mozilla Firefox\distribution` and put [policies.json](https://raw.githubusercontent.com/TeamDominant/Betterfox/main/policies.json) there.
3. Restart again and then check `about:policies`, if you see tweaks there, then you're good to go.
   * You probably don't have distribution, so create it.
   * If your Firefox folder isn't in that path, so find it yourself and do the same steps.

## Overrides

**Follow the original [Betterfox wiki](https://github.com/yokoffing/Betterfox/wiki)**

### Passwords override
!!! danger 
    For those who store passwords in the browser (in a Mozilla account) - remove the `//` at the beginning of line 238; otherwise, the browser won't suggest entering the saved password.

1. Open `user.js` with any IDE
2. Move to the end of file and edit line 238 as said above.

In result, code should look like this:
```js title="user.js" linenums="238" hl_lines="1"
user_pref("signon.rememberSignons", true);
user_pref("ui.key.menuAccessKeyFocuses", false);
user_pref("browser.tabs.warnOnClose", true);
user_pref("browser.tabs.hoverPreview.enabled", false);

/****************************************************************************
 * END: MY OVERRIDES                                                        *
****************************************************************************/
```

## Modifications for other platforms
- [Fennec](https://f-droid.org/packages/org.mozilla.fennec_fdroid/) (F-droid)