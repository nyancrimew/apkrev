# apkrev - android app reverse engineering pipeline

this is tailored pretty much just to my exact needs, down to (for now) hardcoded directories in the script.

```
$ ./apkrev <packagename>
```

## dependencies

* [curl-impersonate](https://github.com/lwthiker/curl-impersonate)
* [htmlq](https://github.com/mgdm/htmlq)
* [jadx](https://github.com/skylot/jadx)
* [hbctool](https://github.com/niosega/hbctool)
* [react-native-decompiler](https://www.npmjs.com/package/react-native-decompiler) (run via npx, so only npx is required)
* vscodium (editor will be configurable in the future)