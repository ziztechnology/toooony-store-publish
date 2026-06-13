# Toooony Store Publish Action

Upload an APK/APKS/XAPK to `tbe_go`, let the backend inspect package metadata, upload it to object storage, and create or update the Toooony internal Store release.

## Minimal

```yaml
- name: Publish Toooony package
  uses: https://git.toooony.com/actions/toooony-store-publish@main
  with:
    backend-url: ${{ secrets.BACKEND_API_URL }}
    backend-token: ${{ secrets.TOOOONY_STORE_PUBLISH_TOKEN }}
    package-file: build/app/outputs/flutter-apk/app-arm64-v8a-release.apk
```

## With Overrides

```yaml
- name: Publish Launcher
  uses: https://git.toooony.com/actions/toooony-store-publish@main
  with:
    backend-url: ${{ secrets.BACKEND_API_URL }}
    backend-token: ${{ secrets.TOOOONY_STORE_PUBLISH_TOKEN }}
    package-file: app/build/outputs/apk/release/launcher-release.apk
    package-name: cn.toooony.launcher
    app-name: Toooony Launcher
    version-name: ${{ github.ref_name }}
    architecture: arm64-v8a
    notes: ${{ github.ref_name }}
    catalog-enabled: "true"
```

The backend endpoint is:

```text
POST /api/v1/admin/store/packages/publish/cicd
Authorization: Bearer <store:publish token>
Content-Type: multipart/form-data
```

Only `file` is required. Fields such as `packageName`, `versionName`, `versionCode`, `architecture`, `notes`, and catalog options are optional overrides when the package metadata is incomplete or intentionally different.
