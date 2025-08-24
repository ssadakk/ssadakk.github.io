---
author: ssadakk
pubDatetime: 2024-12-05T03:00:00.000Z
title: Android MVI pattern
slug: mvi_pattern
featured: false
draft: false
tags:
  - Android
  - MVI
  - Design pattern
  - MVI pattern
description: Android MVI pattern 요약
---

# Android MVI pattern

## 1. MVI?
MVI는 **Model-View-Intent**의 약자, 단방향 데이터 흐름과 불변 상태를 기반으로 동작하는 아키텍처 패턴.

- **Model**: 앱의 상태를 정의하며, 불변 객체로 설계.
- **View**: 사용자 인터페이스(UI)를 렌더링하고 사용자 이벤트를 감지.
- **Intent**: 사용자 이벤트 또는 동작을 모델로 전달하는 매개체.

### MVI의 주요 특징
1. **단방향 데이터 흐름**: 데이터가 항상 한 방향으로 흐름.
2. **불변 상태 관리**: 상태가 한번 생성되고난 후 수정되지 않는다. 업데이트시 새로운 상태 생성
3. **높은 디버깅 가능성**: 상태 변화가 명확하게 정의되어 있어 디버깅 및 테스트가 쉬움.

### MVI vs. MVVM

| 특징        | MVVM                         | MVI                             |
| ----------- | ---------------------------- | ------------------------------- |
| 데이터 흐름 | 양방향 (데이터 바인딩 사용)  | 단방향                          |
| 상태 관리   | 뷰모델에서 관찰 가능         | 단일 상태 객체 관리             |
| 복잡도      | 중간                         | 초기 설정 복잡                  |
| 사용 사례   | 간단한 앱, CRUD 애플리케이션 | 복잡한 비즈니스 로직, 실시간 UI |

## 2. MVI 사용시 이점
### MVI의 장점
1. **단일 진실 원칙**: 상태가 하나의 소스에서 관리되어 UI와의 동기화 문제가 줄어듬.
2. **예측 가능한 동작**: 모든 상태 변화가 명시적으로 정의되므로 디버깅이 용이.
3. **유지보수 용이성**: 상태 관리와 UI 업데이트가 분리되어 유지보수가 쉽다.
4. **리플레이 가능 이벤트**: 상태와 이벤트를 저장해 재실행할 수 있다.

### 적용 사례
- 복잡한 상태 관리가 필요한 애플리케이션 (예: 채팅 앱, 주식 앱).
- 실시간 데이터 처리 및 UI 업데이트가 중요한 경우.

## 3. MVI 요소 분석
### Model
Application의 상태를 정의. Data class로 설계되고 필요한 정보를 포함
```kotlin
data class ViewState(
    val isLoading: Boolean,
    val items: List<String>,
    val errorMessage: String? = null
)
```

### View 
View는 상태를 기반으로 UI를 그리고 사용자 이벤트를 인텐트로 변환하여 전달

### Intent
사용자 액션이나 이벤트를 정의. 이를 통해 상태를 업데이트 하는 동작
```kotlin
sealed class UserIntent {
    object LoadItems : UserIntent()
    data class AddItem(val item: String) : UserIntent()
    data class RemoveItem(val index: Int) : UserIntent()
}
```

## 4. 예제 (TODO list 만들기)
### 1. view state 정의
```kotlin
data class TodoState(
    val todos: List<Todo> = emptyList(),
    val newTodoText: String = "",
    val isLoading: Boolean = false,
    val error: String? = null
)

data class Todo(
    val id: Int,
    val text: String,
    val isCompleted: Boolean = false
) 
```

### 2. Intent 정의
```kotlin
sealed class TodoIntent {
    data class AddTodo(val text: String) : TodoIntent()
    data class ToggleTodo(val id: Int) : TodoIntent()
    data class DeleteTodo(val id: Int) : TodoIntent()
    data class UpdateNewTodoText(val text: String) : TodoIntent()
    object ClearCompleted : TodoIntent()
} 
```

### 3. ViewModel 구현
```kotlin
class TodoViewModel : ViewModel() {
    private val _state = MutableStateFlow(TodoState())
    val state: StateFlow<TodoState> = _state.asStateFlow()
    
    private var nextId = 1

    fun processIntent(intent: TodoIntent) {
        when (intent) {
            is TodoIntent.AddTodo -> {
                if (intent.text.isNotBlank()) {
                    _state.update { currentState ->
                        val newTodo = Todo(id = nextId++, text = intent.text)
                        currentState.copy(
                            todos = currentState.todos + newTodo,
                            newTodoText = ""
                        )
                    }
                }
            }
            
            is TodoIntent.ToggleTodo -> {
                _state.update { currentState ->
                    val updatedTodos = currentState.todos.map { todo ->
                        if (todo.id == intent.id) {
                            todo.copy(isCompleted = !todo.isCompleted)
                        } else {
                            todo
                        }
                    }
                    currentState.copy(todos = updatedTodos)
                }
            }
            
            is TodoIntent.DeleteTodo -> {
                _state.update { currentState ->
                    currentState.copy(
                        todos = currentState.todos.filterNot { it.id == intent.id }
                    )
                }
            }
            
            is TodoIntent.UpdateNewTodoText -> {
                _state.update {
                    Log.e("hmjoo", "update text : ${intent.text}")
                    it.copy(newTodoText = intent.text) }
            }
            
            is TodoIntent.ClearCompleted -> {
                _state.update { currentState ->
                    currentState.copy(
                        todos = currentState.todos.filterNot { it.isCompleted }
                    )
                }
            }

            else -> {}
        }
    }
} 

```

### 4. View 연결
```kotlin
class ToDoActivity : AppCompatActivity() {
    private val viewModel: ToDoViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_todo)

        viewModel.state.observe(this) { state ->
            render(state)
        }

        // 사용자 동작 처리
        findViewById<Button>(R.id.addButton).setOnClickListener {
            val newItem = "New Task"
            viewModel.processIntent(ToDoIntent.AddItem(newItem))
        }
    }

    private fun render(state: ToDoViewState) {
        if (state.isLoading) {
            // 로딩 화면 표시
        } else {
            // UI 업데이트
        }
    }
}
```
