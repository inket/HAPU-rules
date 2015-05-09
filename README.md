### HAPU Rules

---
This repo contains the rules used by the app [HAPU](http://mahdi.jp/apps/hapu). These rules serve to tell how the app should detect when the user is watching a video (whether locally or streaming via browser), and extract media information.

This part was separated from the app to allow it to be more flexible, and mostly to allow users to contribute to these rules.

These files use the YAML format (read on that if you've never used it). Read below for the exact syntax for the rules.

- **app-rules.yaml**: This file contains the rules for detecting local apps.
- **rules.yaml**: This one is for online streaming services.

---

### Contributing

Make a pull request, I'll check (y)

#### app-rules.yaml
##### Syntax
These are the elements for describing the app you want to add:

- **identifier**: The bundle identifier of the app. This value will be used when querying system APIs for running apps. ([string] **required**)

	Note: If your app is a CLI without a bundle identifier, please write a unique string that will be used to identify the app in HAPU. 

- **process**: The name of the process(es) which will be handling the reading of the video file. ([string or list of strings] optional)

	Note: If the process name is unspecified, it will be assumed that the process reading the video file will be the app's main process. (same name, which is mostly the case)

You probably won't need them, but they're implemented anyway:

- **daemon**: Set to true/yes if the described app is a CLI or a hostile app which won't declare itself to system APIs, such as mpv. ([boolean] optional)

- **ignore_if**: The regular expression pattern(s) used to determine whether the ignore a process or not. The string matched against will be the process' command (i.e. what you see when you write `ps x`) ([string or list of strings] optional)

- **osascript**: If the described app allows for scripting with AppleScript, include the AS code which can be used to determine the *path* of the open video file. ([string] optional)

*That's cool but how do I find these values?*

*identifier*: Open the `Info.plist` in your app's bundle, find the value for the key `CFBundleIdentifier`.

*process*: Open a video file with your player, `lsof | grep <videofilename>`, get the PID and use it to find the command `ps -p <pid>`. The process name will be the last part of the command.

##### Example

```
MPlayer OSX Extended:
 identifier: hu.mplayerhq.mplayerosx.extended
 process:
  - mplayer
  - mplayer2
VLC:
 identifier: org.videolan.vlc
 osascript: >
  tell application "VLC" to path of current item
QuickTime Player:
 identifier: com.apple.QuickTimePlayerX
mpv:
 identifier: io.mpv
 daemon: Yes
```

##### Testing

Create `~/Library/Application Support/HAPU/app-rules.yaml`, write your new rules inside it. Restart HAPU, open a video with your player and check whether it works or not.

#### rules.yaml
##### Syntax
These are the elements for describing the streaming service you want to add:

- **domain**: The domain of the service. Subdomains are automatically included ([string] **required**) 

- **url**: A regular expression pattern which will be matched against the page URL to help determine whether the current page has a video page or not. ([string] **required**)

- **anime**: Javascript code which will be ran on the page to determine the name of the anime currently streaming. ([string] **required**)

	Note: Whether you can use jQuery or not depends on the website you're writing rules for. Open your Web Inspector, and write `jQuery` in the Console to know.

- **episode**: Javascript code which will be ran on the page to determine the episode number of the anime currently streaming. ([string] **required**)

- **season**: Javascript code which will be ran on the page to determine the season number of the anime currently streaming. ([string] optional)

*That's cool but how do I find these values?*

Use your Javascript skills to extract the information from the page. Use Web Inspector's Console.  

##### Example

```
AdultSwim:
 domain: "adultswim.com"
 url: adultswim.com/videos/.+?/.+
 anime: strSubSectionName
 episode: >
  $("a[href='"+document.location.pathname+"'] .as-episode-identifier").html().replace(/[^0-9]/g, "");
```

##### Testing

If your code returns correct results in the Web Inspector Console, you should be okay.

Create `~/Library/Application Support/HAPU/rules.yaml`, write your new rules inside it. Restart HAPU, start streaming something on the described website and check whether it works or not.

---

### Considerations

If you want to disable a service/app locally, just change the value to `skip` in your `~/Library/Application Support/HAPU/(app-)rules.yaml`. e.g.

```
mpv:
 identifier: io.mpv
 daemon: Yes
```
becomes

```
mpv: skip
```

**rules.yaml**

I haven't rewritten the current streaming rules in Javascript yet, so even though the following services don't appear in rules.yaml, they're still supported (more or less) by the app:

- Netflix
- Crunchyroll
- Hulu
- Funimation
- Vizanime
- Daisuki
- Wakanim.co.uk
- Wakanim.tv
- Animaxtv.co.uk
- AnimeDigitalNetwork
- AnimeCenter
- AnimeUltima
- AniLinkz
- AnimeSeason
- AnimeHere
- DubbedAnimeStream
- WatchAnimeOn
- Chia-Anime
- AnimeFushigi
- GoGoAnime
- AnimeDreaming
- AnimeSeed
- AnimePalm
- AnimeStatic
- LoveMyAnime
- AnimeTwist
- KissAnime
- rawrANIME.tv
- JKanime

### Contact

Reach me on twitter: [@inket](https://twitter.com/inket) / Contact info on [my website](http://mahdi.jp)