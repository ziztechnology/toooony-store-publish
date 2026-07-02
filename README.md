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

## Large Packages

By default the action uses `upload-mode: auto`: files smaller than 64 MiB keep the original single request path, while larger APK/APKS/XAPK files are uploaded through backend-issued multipart object-storage URLs and then published by `storageKey`.

```yaml
- name: Publish large package
  uses: https://git.toooony.com/actions/toooony-store-publish@main
  with:
    backend-url: ${{ secrets.BACKEND_API_URL }}
    backend-token: ${{ secrets.TOOOONY_STORE_PUBLISH_TOKEN }}
    package-file: release/app-arm64-v8a-release.apk
    package-name: cn.toooony.launcher
    version-name: ${{ github.ref_name }}
    version-code: ${{ steps.version.outputs.version_code }}
    upload-mode: auto
    multipart-threshold: "67108864"
    multipart-part-size: "16777216"
    multipart-concurrency: "4"
```

Multipart mode uses these backend endpoints with the same `store:publish` token:

```text
POST /api/v1/admin/store/packages/upload/cicd/init
POST /api/v1/admin/store/packages/upload/cicd/parts
POST /api/v1/admin/store/packages/upload/cicd/complete
POST /api/v1/admin/store/packages/upload/cicd/abort
POST /api/v1/admin/store/packages/publish/cicd  # JSON storageKey publish
```
