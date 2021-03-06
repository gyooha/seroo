---
title: "Android Backstack"
date: 2020-01-29
last_modified_at: 2020-04-12
desc: "헷갈리는 문제들을 하나씩 정리"
keywords: "Android,Activity,BackStack"
permalink: "/android/backstack"
categories: 
    - Android
tags: 
    - Android
    - Activity
    - BackStack
icon: icon-html
---

# Android 백스택(Bacstack)

안드로이드에는 여러가지 launchMode가 존재한다. 이 launchMode에 따라서 안드로이드의 Activity의 백스택의 쌓이는 방식이 다르다.

> 이번 포스팅의 예제에서는 MainActivity, B_Activity, C_Activity, D_Activity로 예를 들고 있으니 참고해두자.

## Standard

```xml
<application>
     <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity android:name=".B_Activity"/>
        <activity android:name=".C_Activity"/>
        <activity android:name=".D_Activity"/>
</application>
```

각 Activity에 특정 launchMode를 설정하지 않았을 때 기본적으로 standard로 설정된다. 이러한 경우 MainActivity -> B_Activity로 이동하는 경우 MainActivity를 기준으로 위로 쌓이게 된다. Main -> B -> C -> D -> D -> D 로 이동해 보고 dump를 떠보자!

>아래의 명령어로 덤프를 뜰 수 있다.
> * adb shell dumpsys activity activities

```
Hist #5: ActivityRecord{3f339ec u0 io.seroo.androidproblem/.D_Activity t33}
Hist #4: ActivityRecord{50a0c9a u0 io.seroo.androidproblem/.D_Activity t33}
Hist #3: ActivityRecord{4195d6d u0 io.seroo.androidproblem/.D_Activity t33}
Hist #2: ActivityRecord{2acfa43 u0 io.seroo.androidproblem/.C_Activity t33}
Hist #1: ActivityRecord{f0fb4b6 u0 io.seroo.androidproblem/.B_Activity t33}
Hist #0: ActivityRecord{99d26ac u0 io.seroo.androidproblem/.MainActivity t33}
```

위의 결과와 같이 _launchMode_ 를 standard로 설정한 경우 위로 계속 쌓이기만 한다.

## SingleTop

```xml
<application>
     <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity android:name=".B_Activity"/>
        <activity android:name=".C_Activity"/>
        <activity android:name=".D_Activity" android:launchMode="singleTop"/>
</application>
```

_launchMode_ 를 singleTop으로 설정할 경우 standard와 약간 다르다. 

백스택의 최상단 Activity와 같은 종류의 Activity가 백스택에 들어올려고 하는 경우 새로운 Activity를 백스택에 넣는것이 아닌 백스택의 최상단 Activity에서 onNewIntent 메소드가 호출된다.

> 위의 예제에서는 MainActivity -> B_Activity -> C_Activity -> D_Activity -> D_Activity -> D_Activity를 순서대로 이동하면 동일한 Activity가 입력으로 들어와도 계속 쌓였지만, 백스택의 Top에서 동일한 종류의 Activity가 입력으로 들어오는 경우 스택에 쌓이지 않고, onNewIntent 콜백이 호출된다. 

```
Hist #3: ActivityRecord{4195d6d u0 io.seroo.androidproblem/.D_Activity t33}
Hist #2: ActivityRecord{2acfa43 u0 io.seroo.androidproblem/.C_Activity t33}
Hist #1: ActivityRecord{f0fb4b6 u0 io.seroo.androidproblem/.B_Activity t33}
Hist #0: ActivityRecord{99d26ac u0 io.seroo.androidproblem/.MainActivity t33}
```

이후 D_Activity를 호출하면 아래와 같이 로그가 찍힌다.

```
2020-01-30 00:52:28.406 18429-18429/io.seroo.androidproblem D/GYH: onNewIntent
```

하지만 MainActivity -> B_Activity -> C_Activity -> D_Activity 이후 밑에 존재하는 B_Activity를 다시 호출하게 되면 백스택에 깔린 B_Activity가 백스택의 Top으로 올라오는 것이 아닌 새로운 B_Activity 인스턴스가 생성되서 아래와 같이 백스택에 쌓이게 된다.

> 이러한 동작은 SingleTask에서 작동한다.

```
Hist #2: ActivityRecord{2acfa43 u0 io.seroo.androidproblem/.C_Activity t33}
Hist #1: ActivityRecord{f0fb4b6 u0 io.seroo.androidproblem/.B_Activity t33}
Hist #3: ActivityRecord{4195d6d u0 io.seroo.androidproblem/.D_Activity t33}
Hist #2: ActivityRecord{2acfa43 u0 io.seroo.androidproblem/.C_Activity t33}
Hist #1: ActivityRecord{f0fb4b6 u0 io.seroo.androidproblem/.B_Activity t33}
Hist #0: ActivityRecord{99d26ac u0 io.seroo.androidproblem/.MainActivity t33}
```

## SingleTask

```xml
<application>
     <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity android:name=".B_Activity"/>
        <activity
            android:name=".C_Activity"
            android:launchMode="singleTask">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.BROWSABLE" />
                <category android:name="android.intent.category.DEFAULT" />

                <data
                    android:host="test"
                    android:scheme="seroo" />
            </intent-filter>
        </activity>
        <activity android:name=".D_Activity"/>
</application>
```

SingleTask도 기본적으로 같은 앱에서는 같은 Task에 백스택이 쌓이게 된다.

> 필자는 다른 백스택에 쌓이는 줄 알았는데 덤프 떠보니 같은 백스택에 쌓인다. 추가로 SingleTask를 제대로 사용하려고 하는 경우 Intent flag에 NEW_TASK를 같이 넣어줘야 제대로 동작한다.

> 백스택에서 아래에 SingleTask로 선언되어 있는 Activity를 호출시 위에 쌓인 스택들이 clear되고 호출된 Activity에서 _onNewIntent()_ 콜백이 호출된다.

테스트를 위해 **새로운 프로젝트 _AndroidProblem2_**를 만들어 MainActivity에서 _AndroidProblem_ 의 C_Activity를 호출하면, 아래와 같이 둘은 서로다른 Task에 존재하게 된다.

```
TaskRecord{ba7f224 #52 A=io.seroo.androidproblem U=0 StackId=46 sz=2}
        Run #1: ActivityRecord{eb3dd8e u0 io.seroo.androidproblem/.D_Activity t52}
        Run #0: ActivityRecord{4725c9c u0 io.seroo.androidproblem/.C_Activity t52}
```

```
TaskRecord{852807e #51 A=io.seroo.androidproblem2 U=0 StackId=45 sz=1}
        Run #0: ActivityRecord{15b3370 u0 io.seroo.androidproblem2/.MainActivity t51}
```

이미 C_Activity가 백그라운드에서 살아 있는 경우 호출될 때 _onNewIntent()_가 불리게 된다. 백그라운드에 존재하지 않는 경우 새로운 C_Activity 인스턴스를 생성한다.

## SingleInstance

```xml
  <activity android:name=".MainActivity" android:lanchMode="singleInstance">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity android:name=".B_Activity"/>
        <activity android:name=".C_Activity"/>
        <activity android:name=".D_Activity"/>
```

singleInstance는 기본적으로 하나의 Task에 하나의 인스턴스를 갖는것을 의미한다. SingleTask와 유사하게 백그라운드에 singleInstance로 선언된 인스턴스가 살아있는 경우 불리면 _onNewIntent()_ 콜백이 호출된다. 

위와 같이 MainActivity를 singleInstance로 지정하게 되면 MainActivity는 Task의 유일한 인스턴스가 되고 아래와 같이 B_Activity 부터 백스택이 쌓이기 시작한다.

```
TaskRecord{9cd37ae #66 A=io.seroo.androidproblem U=0 StackId=60 sz=3}
        Run #2: ActivityRecord{3005a4a u0 io.seroo.androidproblem/.D_Activity t66}
        Run #1: ActivityRecord{a6d93af u0 io.seroo.androidproblem/.C_Activity t66}
        Run #0: ActivityRecord{9fd03c4 u0 io.seroo.androidproblem/.B_Activity t66}
```

```
  TaskRecord{1f611b0 #65 A=io.seroo.androidproblem U=0 StackId=59 sz=1}
        Run #0: ActivityRecord{d4d7365 u0 io.seroo.androidproblem/.MainActivity t65}
```

> 여기서 재미있는 점은 B_Activity -> C_Activity -> D_Activity로 쌓여있는 66번 Task에서 D_Activity -> MainActivity로 이동해서 다시 MainActivity -> B_Activity로 이동하면 B_Activity로 이동하는 것이 아닌 D_Activity로 이동한다. 결국 A_Activity -> D_Activity -> A_Activity -> D_Activity 반복하게 된다.

# 결론

* standard - 호출된 Activity 순서대로 백스택에 쌓이게 된다.
* singleTop - 백스택의 Top Activity를 중복해서 스택에 쌓으려고 하면 백스택에 쌓이지 않는 대신 _onNewIntent()_ 콜백이 호출된다.
* singleTask - 같은 앱에선 같은 Task에 백스택이 쌓인다. singleTask로 선언된 Activity가 백스택의 아래에 존재할 때 상단 Activity에서 하단의 singleTask로 선언된 Activity 호출시 호출된 Activity위에 쌓인 스택을 clear하고 기존에 존재하는 Activity를 가져온다. 이 때 _onNewIntent()_ 콜백이 호출된다. 
* singleInstance - 하나의 Task에 하나의 Activity만 존재한다. singleTask와 유사하게 백그라운드에 호출된 Activity가 존재하면 새로운 인스턴스를 생성하는 대신 기존에 존재하는 인스턴스를 가져오고 _onNewIntent()_ 콜백이 호출된다.

## 참조
* [작업과 백스택](https://developer.android.com/guide/components/activities/tasks-and-back-stack?hl=en)