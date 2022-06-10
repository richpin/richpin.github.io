---
layout: post
title: "[Android] Recyclerview에 Header를 달기"
description: "가장 간단명료했던 방법"
img: /title/drawing_banner.png
tags: [Android]
---

`Recyclerview`를 다루다 보면 가장 위에 다른 형태의 컨텐츠를 달아야 할 때가 있을 것입니. 제가 말하고자 하는 것은 항상 고정되어 있는 형태가 아닌 `Recyclerview`와 같이 `scroll`이 가능한 즉, `Header`를 다는 것입니다. 저의 경우에는 SNS처럼 커뮤니티 기능을 만드는 도중 가장 위에 마치 게시판같이 환영 멘트, 새로운 정보, 광고 등의 정보를 달 공간이 필요했습니다. 그리고 제가 원하는 것은 고정되어 있는 것이 아닌 `Recyclerview`와 같이 스크롤이 되는 관계였습니다.

# 무슨 방법들이 있을까?

이와 같은 문제를 구글링 해보면 가장 흔하게 접하는 것이 바로 `Nested Scroll View`를 이용하라는 것입니. 여러 `Layout`들을 묶어 마치 하나처럼 `scroll` 되는 이 방법을 저도 제일 처음으로 써보았습니다. 동작 자체는 원하는 대로 되었지만, 성능의 문제가 발생했습니다. 아무래도 커뮤니티인만큼 굉장히 동적으로 동작하는 `Recyclerview`라서 그런지 전에 없었던 버벅임이 도저히 감당할 수 없을 정도로 심했습니다. 이해가 가지 않어 원인을 찾아보려 애썼지만 결국 포기가 더 빨랐네요... :sweat_smile:

두 번째 시도했던 방법은 `Header`를 `Recyclerview`의 첫번째 아이템으로 다루는 방법입니다. 아이템이 첫번째라면 다른 아이템 타입을 적용시켜 관리하는 방법은 언뜻 보기에도 그렇게 끌리지는 않았습니다. 뭔가 깔끔하기 보다 억지로 우겨넣는(?) 방법인 것처럼 느껴졌기 때문입니다. 하지만, 해결도 제대로 안되면서 깔끔한 방식, 코드만 찾는 저의 습성때문에 피 본 날이 하루가 아니니 그래도 시도는 해보았네요. 제가 제대로 신경을 못쓴 것이겠지만, 역시나 어플이 뻑이 가버렸습니다. 원인을 파악할 의지도 없었습니다. 도대체 왜 제가 생각하는 깔끔한 방법이 없는 것인지...항상 이렇습니다. :joy: 그러다 정말 우여곡절 끝에 우연히 딱 맘에 드는 방법을 찾아 이 글을 쓰게 되었습니다.

# 제가 소개하는 방법

이 방법은 `RecyclerView`가 그대로 `RecyclerView`인 채로 존재할 수 있어서(xml 처리가 다여서) 가장 깔끔했습니다.

```xml
<androidx.coordinatorlayout.widget.CoordinatorLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.google.android.material.appbar.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:elevation="0dp">

        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="@color/white"
            app:layout_scrollFlags="scroll|enterAlwaysCollapsed">

            <ImageView
                android:id="@+id/banner"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:adjustViewBounds="true"/>
        </RelativeLayout>

    </com.google.android.material.appbar.AppBarLayout>

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerview"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        android:overScrollMode="never">

    </androidx.recyclerview.widget.RecyclerView>
</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

`CoordinatorLayout`의 `AppBarLayout`을 이용하여 본래 `AppBar`(상단 툴바라고 생각하면 편함.)로 쓰여질 공간에 Header 컨텐츠를 넣는 것입니다. 따라서 `AppBar`로서의 이질감을 없애기 위해 `elevation`을 `0dp`로 주었습니다. 위와 같이 `AppBar`안의 `Layout`에 `app:layout_scrollFlags="scroll"`과 `Recyclerview`에 `app:layout_behavior="@string/appbar_scrolling_view_behavior"`을 해주면 이들은 마치 하나같이 `scroll`을 가능케 합니다. 추가적으로 `app:layout_scrollFlags`에는 `"enterAlways", "enterAlwaysCollapsed", "exitUntilCollapsed"` 기능이 있는데 이는 `scroll`시 `AppBar`가 노출되는 양상을 각기 다르게 나타나기에 취향껏 고르시면 됩니다. 더불어, `RecyclerView`특유의 `scroll`시 그림자가 생겨서 `AppBar`와의 이질감을 형성하기 때문에 `android:overScrollMode="never"`까지 설정해주면 좋습니다. :smile:
