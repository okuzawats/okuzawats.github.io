---
title: "[Android] ライブラリを使ってRuntime Permissionをリクエストする"
description: "PermissionsDispatcher、RxPermissionsを使って、AndroidのRuntime Permissionをリクエストする方法についてまとめます。"
date: 2019-09-28T21:41:10+09:00
categories: ["Android"]
aliases:
    - /blog/runtime-permission-with-rxpermissions/
    - /posts/runtime-permission-with-rxpermissions/
    - /blog/runtime-permission-with-permissionsdispatcher/
draft: false
---

ライブラリを使ってAndroidのRuntime Permissionをリクエストする方法をまとめます。Runtime Permissionをリクエストするために使える有名なライブラリには、PermissionsDispatcherとRxPermissionsがあります。

## PermissionsDispatcherを用いたRuntime Permissionのリクエスト

PermissionsDispatcherを使ってRuntime Permissionをリクエストする方法を示します。

- [permissions\-dispatcher/PermissionsDispatcher: A declarative API to handle Android runtime permissions\.](https://github.com/permissions-dispatcher/PermissionsDispatcher)

とりあえずAndroidManifest.xmlにパーミッションを追加します。

```xml
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```

app/build.gradleにPermissionDispatcherとkaptを追加します。

```groovy
apply plugin: 'kotlin-kapt'

dependencies {
    // PermissionDispatcher
    implementation "org.permissionsdispatcher:permissionsdispatcher:4.5.0"
    kapt "org.permissionsdispatcher:permissionsdispatcher-processor:4.5.0"
}
```

Fragmentに `@RuntimePermissions` というアノテーションを付けます。PermissionsDispatcherによって自動生成されるメソッドがあるので、以下、適宜ビルドしながら進めます。

```kotlin
@RuntimePermissions
class StartFragment : Fragment() {
   // do something here
}
```

`READ_EXTERNAL_STORAGE` のパーミッションを得て、ギャラリーを表示するみたいな実装を想定して、パーミッション周りの処理を実装していきます。

```kotlin
/** Permission Dispatcher */
@NeedsPermission(Manifest.permission.READ_EXTERNAL_STORAGE)
fun showGallery() {
    findNavController().navigate(R.id.action_start_to_gallery)
}

@OnShowRationale(Manifest.permission.READ_EXTERNAL_STORAGE)
fun onShowRationaleForReadExternalStorage(request: PermissionRequest) {
    showRationaleDialog(
        requireContext(),
        request,
        R.string.start_request_permission_for_read_external_storage
    )
}

@OnPermissionDenied(Manifest.permission.READ_EXTERNAL_STORAGE)
fun onReadExternalStorageDenied() {
    Toast.makeText(requireContext(), "Permission denied!", Toast.LENGTH_LONG).show()
}

@OnNeverAskAgain(Manifest.permission.READ_EXTERNAL_STORAGE)
fun onReadExternalStorageNeverAskAgain() {
    Toast.makeText(requireContext(), "Never ask again!", Toast.LENGTH_LONG).show()
}
```

`onRequestPermissionsResult()` をoverrideして、この中でPermissionsDispatcherによって自動生成される `onRequestPermissionsResult()` を呼び出せば、パーミッションの許可周りを適切にハンドリングしてくれます。

```kotlin
override fun onRequestPermissionsResult(
    requestCode: Int,
    permissions: Array<out String>,
    grantResults: IntArray
) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults)
    onRequestPermissionsResult(requestCode, grantResults)
}
```

`showRationaleDialog()` は拡張関数で定義しました。

```kotlin
fun Fragment.showRationaleDialog(
    context: Context,
    request: PermissionRequest,
    @StringRes messageResId: Int
) {
    AlertDialog.Builder(context)
        .setPositiveButton(R.string.ok) { _, _ -> request.proceed() }
        .setNegativeButton(R.string.cancel) { _, _ -> request.cancel() }
        .setMessage(messageResId)
        .show()
}
```

最後に、PermissionsDispatcherによって自動生成された `showGalleryWithPermissionCheck()` を呼び出せば、パーミッションのチェックが走ります。

```kotlin
showGalleryWithPermissionCheck()
```

## RxPermissionsを用いたRuntime Permissionのリクエスト

RxPermissionsを使ってRuntime Permissionをリクエストする方法を示します。

- [tbruyelle/RxPermissions: Android runtime permissions powered by RxJava2](https://github.com/tbruyelle/RxPermissions)

プロジェクトレベルの `build.gradle` にJitPackのリポジトリを追加します。

```groovy
allprojects {
    repositories {
        maven { url 'https://jitpack.io' }
    }
}
```

アプリレベルの `build.gradle` にRxJavaとRxPermissionsを追加します。

```groovy
dependencies {
    // RxJava
    implementation 'io.reactivex.rxjava2:rxjava:2.2.13'

    // RxPermissions
    implementation 'com.github.tbruyelle:rxpermissions:0.10.2'
}
```

AndroidManifest.xmlにパーミッションを追加します。

```xml
<uses-permission android:name="android.permission.CAMERA" />
```

RxPermissionsによるRuntime Permissionの実装は簡単です。以下はFragmentでの実装になります。

```kotlin
val rxPermissions = RxPermissions(this)

rxPermissions
    .request(Manifest.permission.CAMERA)
    .subscribe { granted ->
        if (granted) {
            // permission granted
        } else {
            // permission denied
        }
    }
```
