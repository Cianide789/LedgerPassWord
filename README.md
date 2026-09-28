# Ledger Passwords for Nano Gen5

This repository builds the unmodified official [LedgerHQ/app-passwords](https://github.com/LedgerHQ/app-passwords) source for Nano Gen5 (`apex_p`). It does not modify or vendor the upstream application source.

## Build

Run **Actions → Build Ledger Passwords for Nano Gen5 → Run workflow**, or push to `main`. The workflow resolves the current default-branch HEAD of the official upstream repository, records its immutable commit SHA, and calls LedgerHQ's pinned `reusable_build.yml` workflow for `apex_p` only. It passes no flags and selects no use case. In the current upstream Makefile this leaves `TESTING=0`; the test APDU code guarded by `#ifdef TESTING` is excluded.

The run publishes the `passwords-gen5-validated` artifact for 7 days. It contains `app.elf`, `app.apdu`, a copy of both at `build/apex_p/bin/` for Ledger's device tooling, `BUILD_INFO.txt`, and `SHA256SUMS.txt`. The packaging job fails if either loader file is missing, if multiple device outputs are present, or if the ELF symbol/string scan finds `test_generate_password`.

Verify downloaded files before use:

```sh
sha256sum -c SHA256SUMS.txt
```

Compare `BUILD_INFO.txt`'s upstream SHA with the desired source revision. An Actions artifact is a developer build; it is not a Ledger-reviewed release and does not carry Ledger's production signing or endorsement.

## Load on a Nano Gen5

Ledger's current developer tooling supports **Apex p** and provides a **Load on device** task through the Ledger Developer Tools VS Code extension. The extension's current loader uses `ledgerblue.runScript` with the target's `app.apdu` and `app.elf` files. Use a dedicated development device when possible.

1. Download and verify the `passwords-gen5-validated` artifact. Keep the source checkout of `LedgerHQ/app-passwords` at the exact commit in `BUILD_INFO.txt`.
2. Place the artifact's two files from `build/apex_p/bin/` into that checkout's `build/apex_p/bin/` directory (replacing files there), or use the extension's existing build output directory for the selected **Apex p** target.
3. Connect the Nano Gen5, unlock it with its PIN, and leave it on the dashboard. Do not start another app.
4. In VS Code, open the upstream app checkout with Ledger Developer Tools installed, select **Apex p**, and run **Load on device**. This invokes Ledger's loader with the `.apdu` and `.elf` files.
5. Read the device's installation warning and confirm only if you intend to install this developer/sideload application. Verify the app identity and any prompt on the device. The sideload is not a Ledger Wallet store installation and is not endorsed or reviewed by Ledger.

If the device refuses installation, stop and follow the current Ledger Developer Tools / device OS requirements; do not install a custom certificate or bypass a device warning to force it. Never expose your recovery phrase during app installation. Because Passwords deterministically derives account passwords from the device seed, install only a build whose source SHA, checksums, and test-mode validation you have independently checked.
