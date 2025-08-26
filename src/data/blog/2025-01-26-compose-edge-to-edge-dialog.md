---
author: HM Joo
pubDatetime: 2025-01-26T10:00:00Z
title: Compose Edge-to-Edge Dialog 구현
featured: true
draft: false
tags:
  - android
  - compose
  - kotlin
  - ui
description: Jetpack Compose에서 TopBar와 시스템 바를 모두 덮는 진정한 전체 화면 다이얼로그 구현하기. DecorView를 직접 조작하는 방법.
---

Compose로 개발하다가 화면 전체를 완전히 덮는 다이얼로그가 필요했는데, 기본 Dialog Composable은 TopBar를 못 덮더라. 이 문제 해결하느라 삽질 좀 했는데, 결국 해결해서 정리해본다.

## Table of contents

## 문제 상황

일반적인 Compose Dialog의 한계:
- TopBar(AppBar)를 덮지 못함
- StatusBar, NavigationBar 영역을 완전히 활용 못함
- `decorFitsSystemWindows = false` 설정이 Dialog에서는 제대로 동작 안함

특히 풀스크린 이미지 뷰어나 중요한 알림 다이얼로그 만들 때 이게 문제가 됨.

## 해결 방법: DecorView 직접 조작

### 핵심 원리

Android View 계층 구조를 보면:

```
Window
  └── DecorView (최상위 View)
      ├── StatusBar 영역
      ├── Content 영역 (앱의 모든 UI)
      │   ├── TopBar
      │   └── 기타 Compose UI
      └── NavigationBar 영역
```

DecorView는 Window의 최상위 View다. 여기에 직접 View를 추가하면 앱의 모든 UI 요소보다 위에 렌더링된다. 이걸 활용하면 진정한 전체 화면 오버레이를 만들 수 있다.

## 구현 코드

### EdgeToEdgeDialog Composable

```kotlin:EdgeToEdgeDialog.kt
@Composable
fun EdgeToEdgeDialog(
    onDismissRequest: () -> Unit,
    dismissOnBackPressed: Boolean = true,
    dismissOnOutsideClick: Boolean = false,
    backgroundDimAlpha: Float = 0.8f,
    content: @Composable () -> Unit
) {
    val context = LocalContext.current
    val view = LocalView.current
    
    // 뒤로가기 처리
    if (dismissOnBackPressed) {
        BackHandler(onBack = onDismissRequest)
    }
    
    DisposableEffect(Unit) {
        val activity = context as? android.app.Activity
        var overlayView: ComposeView? = null
        
        if (activity != null) {
            // 새로운 ComposeView 생성
            overlayView = ComposeView(activity).apply {
                setContent {
                    // 전체 화면 오버레이
                    Box(
                        modifier = Modifier
                            .fillMaxSize()
                            .background(Color.Black.copy(alpha = backgroundDimAlpha))
                            .clickable(
                                indication = null,
                                interactionSource = remember { MutableInteractionSource() }
                            ) {
                                if (dismissOnOutsideClick) {
                                    onDismissRequest()
                                }
                            },
                        contentAlignment = Alignment.Center
                    ) {
                        Box(
                            modifier = Modifier.clickable(
                                indication = null,
                                interactionSource = remember { MutableInteractionSource() }
                            ) { 
                                // 내용물 클릭 시 이벤트 전파 방지
                            }
                        ) {
                            content()
                        }
                    }
                }
            }
            
            // Activity의 DecorView에 직접 추가
            val decorView = activity.window.decorView as ViewGroup
            
            // DecorView의 최상위에 추가 (모든 UI 위에 표시)
            decorView.addView(
                overlayView,
                ViewGroup.LayoutParams(
                    ViewGroup.LayoutParams.MATCH_PARENT,
                    ViewGroup.LayoutParams.MATCH_PARENT
                )
            )
            
            // 시스템 UI 플래그 설정으로 시스템 바 영역까지 확장
            @Suppress("DEPRECATION")
            activity.window.decorView.systemUiVisibility = (
                activity.window.decorView.systemUiVisibility
                or android.view.View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                or android.view.View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                or android.view.View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
            )
        }
        
        onDispose {
            // 다이얼로그가 닫힐 때 오버레이 제거
            overlayView?.let { overlay ->
                (overlay.parent as? ViewGroup)?.removeView(overlay)
            }
        }
    }
}
```

코드 보면 알겠지만, 핵심은:

1. **ComposeView 생성**: 새로운 Compose 컨테이너를 생성
2. **DecorView 접근**: Activity의 Window에서 최상위 View 가져옴  
3. **오버레이 추가**: DecorView에 ComposeView를 직접 추가해서 모든 UI 위에 표시
4. **시스템 UI 확장**: SystemUiVisibility 플래그로 시스템 바 영역까지 확장
5. **메모리 관리**: DisposableEffect로 다이얼로그 닫힐 때 View 제거

## 사용 예제

### 기본 사용법

```kotlin
@Composable
fun MyScreen() {
    var showDialog by remember { mutableStateOf(false) }
    
    if (showDialog) {
        EdgeToEdgeDialog(
            onDismissRequest = { showDialog = false },
            dismissOnBackPressed = true,
            dismissOnOutsideClick = true,
            backgroundDimAlpha = 0.5f
        ) {
            // 다이얼로그 내용
            Card(
                modifier = Modifier
                    .width(300.dp)
                    .wrapContentHeight(),
                shape = RoundedCornerShape(16.dp)
            ) {
                Text(
                    text = "전체 화면 다이얼로그",
                    modifier = Modifier.padding(16.dp)
                )
            }
        }
    }
}
```

### 커스텀 다이얼로그

```kotlin
@Composable
fun CustomFullScreenDialog(
    title: String,
    message: String,
    onConfirm: () -> Unit,
    onDismiss: () -> Unit
) {
    EdgeToEdgeDialog(
        onDismissRequest = onDismiss,
        backgroundDimAlpha = 0.8f
    ) {
        Card(
            modifier = Modifier
                .width(400.dp)
                .wrapContentHeight(),
            shape = RoundedCornerShape(24.dp),
            colors = CardDefaults.cardColors(containerColor = Color.White)
        ) {
            Column(
                modifier = Modifier.padding(24.dp),
                horizontalAlignment = Alignment.CenterHorizontally
            ) {
                Text(
                    text = title,
                    style = MaterialTheme.typography.headlineMedium,
                    color = Color.Black
                )
                
                Spacer(modifier = Modifier.height(16.dp))
                
                Text(
                    text = message,
                    style = MaterialTheme.typography.bodyLarge,
                    color = Color.Gray
                )
                
                Spacer(modifier = Modifier.height(24.dp))
                
                Row {
                    Button(
                        onClick = onDismiss,
                        colors = ButtonDefaults.buttonColors(
                            containerColor = Color.Gray
                        )
                    ) {
                        Text("취소")
                    }
                    
                    Spacer(modifier = Modifier.width(16.dp))
                    
                    Button(onClick = onConfirm) {
                        Text("확인")
                    }
                }
            }
        }
    }
}
```

## 주요 특징

### 장점
- **완전한 전체 화면**: TopBar, BottomBar, 시스템 바 모두 덮음
- **유연한 제어**: 배경 투명도, 클릭 동작 등 세밀한 제어 가능
- **메모리 안전**: DisposableEffect로 자동 정리
- **뒤로가기 처리**: BackHandler로 네이티브 뒤로가기 버튼 지원

### 다른 방식과 비교

| 방식 | TopBar 덮기 | 시스템 바 덮기 | 구현 복잡도 |
|------|------------|---------------|------------|
| 일반 Dialog Composable | ❌ | ❌ | 낮음 |
| DialogFragment | ❌ | ⚠️ 부분적 | 중간 |
| DecorView 방식 (이 구현) | ✅ | ✅ | 중간 |
| 별도 Activity | ✅ | ✅ | 높음 |

## 주의사항

### 성능 관련
- DecorView 조작은 신중하게 사용해야 함
- 동시에 여러 개의 오버레이 추가하지 말 것
- 다이얼로그 닫힐 때 반드시 View 제거 (메모리 누수 방지)

### 호환성
- Android API 21+ (Lollipop) 이상
- Compose 1.0.0 이상 필요
- Activity 컨텍스트 필요 (Fragment에서는 추가 처리 필요)

## 트러블슈팅

### 다이얼로그가 시스템 바를 못 덮는 경우

시스템 UI 플래그 확인:
```kotlin
SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
```

### 다이얼로그 닫혀도 오버레이가 남아있는 경우

DisposableEffect의 onDispose에서 View 제거 확인:
```kotlin
onDispose {
    overlayView?.let { overlay ->
        (overlay.parent as? ViewGroup)?.removeView(overlay)
    }
}
```

### 뒤로가기 버튼이 안 먹는 경우

BackHandler 추가:
```kotlin
if (dismissOnBackPressed) {
    BackHandler(onBack = onDismissRequest)
}
```

## 마무리

이 구현은 Compose의 한계를 Android View 시스템으로 보완한 하이브리드 접근. 진정한 전체 화면 오버레이가 필요할 때 쓸만한 솔루션이다.