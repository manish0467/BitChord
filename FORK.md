# manish0467/BitChord — fork notes

This fork adds BitChord's `bitchord://queue` link with title/artist matching (used by MovieBox's
"Play all"). It ships the **dev flavor** (`com.dev.bitchord`, "BitChord Dev"), which installs next to
official BitChord, from this repository's GitHub releases. Its in-app updater reads this repository's
releases (`UPDATE_REPO` in `app/build.gradle.kts`); the prod flavor still reads upstream's.

## Signing

- `bitchord-release.jks` and `keystore.properties` live in the project root and are gitignored.
- Both are backed up in Bitwarden. **Losing them means no update can ever install over an existing
  copy**: users would have to uninstall and lose their library.
- Never commit either file. For CI, store the keystore as `KEYSTORE_BASE64` and the passwords as
  `KEYSTORE_PASSWORD`, `KEY_ALIAS`, `KEY_PASSWORD` repository secrets (what `android.yml` expects).

## Versions

Upstream's version plus a fourth number: `1.9.2.1`, `1.9.2.2`, then `1.9.3.1` after merging upstream's
1.9.3. Only numbers: the updater treats any `-suffix` as a pre-release. Raise `bitchordVersionCode`
by one every release and keep it above upstream's. Both live in the root `build.gradle.kts`, and
`bitchordBetaSuffix` stays empty for a release.

## Releasing

```bash
# 1. bump bitchordVersionCode / bitchordVersionName in build.gradle.kts, commit on main
./gradlew :app:assembleDevRelease
# 2. rename for the updater: it only takes assets named "BitChord-*"; the universal APK first
cp app/build/outputs/apk/dev/release/app-dev-universal-release.apk   BitChord-v1.9.2.1.apk
cp app/build/outputs/apk/dev/release/app-dev-arm64-v8a-release.apk   BitChord-v1.9.2.1_arm64-v8a.apk
# 3. tag and release (the tag is the version with a "v")
git tag v1.9.2.1 && git push origin main v1.9.2.1
gh release create v1.9.2.1 BitChord-v1.9.2.1.apk BitChord-v1.9.2.1_arm64-v8a.apk \
  -R manish0467/BitChord --title "BitChord 1.9.2.1 (fork)" --notes "..."
```

## Keeping up with upstream

```bash
git fetch upstream
git merge upstream/main          # resolve conflicts, keep UPDATE_REPO and the fork's version
# set the version to upstream's + ".1", code above upstream's, then test the queue link from MovieBox
```
