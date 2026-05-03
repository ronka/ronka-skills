---
name: expo-publish
description: Guides publishing an Expo/EAS app — OTA updates, production builds (iOS/Android), store submission, and simulator builds. Use when the user says "publish", "release", "submit to store", "OTA update", "build for production", "build simulator", or runs any EAS build/submit/update workflow. Knows the project's npm scripts, version-bump logic, and correct command order.
---

# Expo Publish

## Workflows

### 1. OTA Update (no store review needed)

Bumps `UPDATE_VERSION` in `constants/version.ts`, then pushes the update to all users on the current channel.

```bash
npm run update
```

Then commit the version bump:
```bash
git add constants/version.ts && git commit -m "chore: bump UPDATE_VERSION"
```

---

### 2. Production Build

The build scripts auto-bump the patch version in `app.json` before building.

**iOS only:**
```bash
npm run build:production:ios
```

**Android only:**
```bash
npm run build:production:android
```

**Both platforms:**
```bash
npm run build:production:ios && npm run build:production:android
```

After building, commit the version bump:
```bash
git add app.json && git commit -m "chore: bump app version"
```

---

### 3. Store Submission

Submit the latest build to the App Store / Play Store. Run **after** a successful production build.

**iOS:**
```bash
npm run submit:production:ios
```

**Android:**
```bash
npm run submit:production:android
```

---

### 4. Full Release (Build + Submit)

**iOS:**
```bash
npm run build:production:ios && npm run submit:production:ios
```

**Android:**
```bash
npm run build:production:android && npm run submit:production:android
```

Commit version bump after builds complete.

---

### 5. Simulator Build

For local development/testing — not for store submission.

**iOS simulator:**
```bash
npm run build:simulator:ios
```

**Android simulator:**
```bash
npm run build:simulator:android
```

---

## Setup in a New Project

Copy the bundled scripts into the project:
```bash
cp <skill-path>/scripts/bump-app-version.js scripts/
cp <skill-path>/scripts/increment-update-version.js scripts/
```

Add these npm scripts to `package.json`:
```json
"update": "node scripts/increment-update-version.js && npx eas update",
"build:production:ios": "node scripts/bump-app-version.js && npx eas build --platform ios --profile production",
"build:production:android": "node scripts/bump-app-version.js && npx eas build --platform android --profile production",
"submit:production:ios": "npx eas submit --platform ios --profile production",
"submit:production:android": "npx eas submit --platform android --profile production",
"build:simulator:ios": "npx eas build --platform ios --profile development-simulator",
"build:simulator:android": "npx eas build -p android --profile development-simulator"
```

Ensure `constants/version.ts` exports `UPDATE_VERSION`:
```ts
export const UPDATE_VERSION = 1;
```

---

## Version Bump Logic

| Script | What gets bumped | File |
|--------|-----------------|------|
| `npm run update` | `UPDATE_VERSION` (integer, +1) | `constants/version.ts` |
| `npm run build:production:*` | `version` patch (semver x.y.z+1) | `app.json` |

Always commit version bumps so git history tracks releases accurately.

## Checklist Before Releasing

- [ ] All changes committed and pushed
- [ ] Tested on a physical device or simulator
- [ ] `eas.json` profiles are correct for the target environment
