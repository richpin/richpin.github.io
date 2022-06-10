---
layout: post
title: "[Android] 원하는 이미지를 다양한 픽셀에 맞도록 android resource로 추가하는 법"
description: "9-patch 이미지를 사용하여 이미지 리소스 생성하기"
img: /title/android_resize.png
tags: [Android]
---

안드로이드 어플리케이션을 만드는 도중 초반 작업인 **Splash** 화면을 만들고 있었습니다.

# 안드로이드 Splash 화면 만들기

어플이 실행되기 전에 로딩 화면과도 같은 `Splash`는 대부분의 상용 어플들에 적용이 되어 있습니다. 이 `Splash` 화면을 만들기 위해 많은 방법들이 있습니다. 대표적으로 사용되는 것이 `Splash Activity` 방법인데 이는 간단히 `Main Activity`가 실행되기 전, `Splash Activity`를 띄우는 방법입니다. 허나 단순히 팝업 이미지를 위해 `Activity`를 소모하는 것은 자원 낭비가 심하다 판단해 요즘은 `Theme` 방법을 쓰는 것 같습니다. 물론 저는 이 방법으로 했습니다. (만약 애니메이션 형식의 Splash를 원한다면 [여기](https://developer.android.com/about/versions/12/features/splash-screen)를 참조하도록)

방법은 간단합니다. 먼저 `drawable - New - Drawable Resource File`을 들어가 새로운 splash용 `xml 파일`을 생성하여 줍니다.

![splash_create](/assets/img/etc/android_density/splash_create.png){: width="70%" height="70%"}

만들어 졌다면 해당 xml 파일로 들어가 아래와 같은 코드를 넣어줍니다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@android:color/white" />
    <item>
        <bitmap
            android:src="@drawable/app_logo"
            android:gravity="center"/>
    </item>
</layer-list>
```

`android:drawable`에는 넣고 싶은 배경색을 넣어주면 되고, `android:src`에는 넣고 싶은 `Splash` 이미지를 넣어주면 됩니다.
그 후에 `AndroidManifest.xml`에서 `MainActivity`의 `Theme`을 새로 만든 `SplashTheme`으로 지정하여 줍니다.

```xml
<activity
    android:name=".MainActivity"
    android:exported="true"
    android:theme="@style/SplashTheme">
```

그리고 `MainActivity.kt`(`Java`의 경우엔 `MainActivity.java`)의 `onCreate`에서 가장 먼저 `Theme`을 기존 `MainActivity`의 `Theme`으로 교체 해줍니다. 다소 야매같은 방법이지만 비효율적인 낭비 없이 깔끔하게 할 수 있는 방법인 것 같습니다.:relieved:

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        setTheme(R.style.Theme_Nutrihanjum)
        super.onCreate(savedInstanceState)
    }
```

# 이미지를 다양한 픽셀에 맞도록 android resource로 추가하기

여기서 문제는 원하는 이미지를 `drawable`에 raw하게 넣어버리고 실행을 하면 얘 크기가 아주 들쭉날쭉이라는 것입다. 이는 픽셀 밀도 차이를 고려하지 않아서 그렇습니다. Android 기기는 다양한 화면 크기(핸드셋, 태블릿, TV 등)로 제공될 뿐만 아니라 화면의 픽셀 크기도 다양합니다. 즉, 한 기기에서는 제곱인치당 160픽셀을 사용하지만 다른 기기에서는 같은 공간에 480픽셀을 사용합니다. 이러한 픽셀 밀도의 편차를 고려하지 않으면 시스템이 이미지를 확장하거나(결과적으로 이미지가 흐려짐) 이미지가 완전히 잘못된 크기로 표시될 수 있는 것이죠.

![relative_size](/assets/img/etc/android_density/relative_size.png){: width="60%" height="60%"}

이에 대해 더 명확한 내용은 잘 설명된 글들이 많으니 생략하겠습니다. **(이미 다른 좋은 글들이 많은 주제는 굳이 건들지 않는 것이 내 블로그의 의의랍니다.)**

따라서 이미지를 그냥 박는 것이 아니라 몇 가지의 과정이 필요하게 됩니다.

먼저, [Android Asset Studio - Simple nine-patch generator](https://romannurik.github.io/AndroidAssetStudio/nine-patches.html#&sourceDensity=320&name=example)에 들어가서 넣고 싶은 이미지를 올립니다.

![asset_studio](/assets/img/etc/android_density/asset_studio.png){: width="80%" height="80%"}

그런 다음 별도 과정 없이 바로 위에 표시해둔 저장 표시를 클릭하면 `zip`파일이 다운되고 압축을 해제하면 `res`라는 폴더 안에 픽셀 밀도에 맞는 각 이미지들이 정렬된 폴더들이 생성되어 있게 됩니다.

![studio_result](/assets/img/etc/android_density/studio_result.png){: width="70%" height="70%"}

파일 형식이 `9.png`로 되어있는 것은 이 이미지들이 `9-patch image` 형식이라 그렇다고만 알면 될 것 같습니다.

이제 이 파일들을 또 그대로 `drawable`에 갖다 박는다면! 위와 같이 `9-patch` 이미지 특유의 테두리 선들이 남아있을 것이다...:flushed:

따라서 우리는 `Resource Manager`를 사용해야 합니다. 위치는 아래와 같습나다.

![resource_manager](/assets/img/etc/android_density/resource_manager.png){: width="50%" height="50%"}

그 후에 저 영역에 `res` 폴더 안 5개의 폴더를 드래그 하여 넣거나 **+** 에서 'Import Drawables`를 클릭하여 res폴더를 지정해주면

![import_drawables](/assets/img/etc/android_density/import_drawables.png){: width="50%" height="50%"}

파일 이름 지정해주고 설정들은 기호에 따라 맞춰준뒤 Next

![import_finish](/assets/img/etc/android_density/import_finish.png){: width="50%" height="50%"}

마지막으로 `finish`를 클릭해주면

![drawable_result](/assets/img/etc/android_density/drawable_result.png){: width="50%" height="50%"}

이와 같이 기똥차게 픽셀 밀도 별로 정리되어 있는 것을 볼 수 있습니다. 이제 맘 편히 리소스로 쓸 수 있는 상태가 되었네요. :grin:

구글링을 해서 뒤져봐도 비슷한 부류의 얘기들 뿐, 내가 원했던 이 얘기를 하는 명확한 글을 찾지 못했습니다. 그래서 이렇게나 간단한 일인데도 애 좀 먹었답니다...:sweat_smile:
