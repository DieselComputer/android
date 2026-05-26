<div align="center">

**МИНИСТЕРСТВО НАУКИ И ВЫСШЕГО ОБРАЗОВАНИЯ РОССИЙСКОЙ ФЕДЕРАЦИИ**  
**ФЕДЕРАЛЬНОЕ ГОСУДАРСТВЕННОЕ БЮДЖЕТНОЕ ОБРАЗОВАТЕЛЬНОЕ УЧРЕЖДЕНИЕ ВЫСШЕГО ОБРАЗОВАНИЯ**  
**«САХАЛИНСКИЙ ГОСУДАРСТВЕННЫЙ УНИВЕРСИТЕТ»**

<br>
<br>
<br>
<br>
<br>

Институт естественных наук и техносферной безопасности  
Кафедра информатики  
Вдовина Милена Романовна

<br>
<br>
<br>
<br>
<br>

Лабораторная работа №12

01.03.02 Прикладная математика и информатика

3 Курс

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

<div align="right">
Научный руководитель<br>
Соболев Евгений Игоревич
</div>

<br>
<br>
<br>

г. Южно-Сахалинск  
2026 г.

</div>

---  

## Цель работы: Научиться выполнять длительные операции в фоновом потоке с использованием корутин и viewModelScope, управлять состоянием загрузки в UI, реализовать имитацию загрузки данных и обработку ошибок.

---
### `UiState.kt`
```kotlin
package com.example.app12.ui

import com.example.app12.database.TaskEntity

sealed class TasksUiState {
    object Loading : TasksUiState()
    data class Success(val tasks: List<TaskEntity>) : TasksUiState()
    data class Error(val message: String) : TasksUiState()
}
```
### `MainViewModel.kt`
```kotlin
package com.example.app12

import androidx.lifecycle.ViewModel
import kotlinx.coroutines.flow.StateFlow
import androidx.lifecycle.viewModelScope
import com.example.app12.database.TaskEntity
import kotlinx.coroutines.launch
import com.example.app12.data.repository.TaskRepository
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.stateIn
import com.example.app12.ui.TasksUiState
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.asStateFlow

class MainViewModel(
    private val repository: TaskRepository
) : ViewModel() {

    private val _uiState = MutableStateFlow<TasksUiState>(TasksUiState.Loading)
    val uiState: StateFlow<TasksUiState> = _uiState.asStateFlow()

    init {
        loadTasks()
    }

    fun loadTasks() {
        viewModelScope.launch {
            _uiState.value = TasksUiState.Loading
            try {
                // Имитация длительной загрузки (например, сетевой запрос)
                delay(2000) // 2 секунды

                // Получаем данные из репозитория
                val tasks = repository.getTasksOnce()
                _uiState.value = TasksUiState.Success(tasks)
            } catch (e: Exception) {
                _uiState.value = TasksUiState.Error(e.message ?: "Ошибка загрузки")
            }
        }
    }

    fun addTask(title: String) {
        viewModelScope.launch {
            repository.addTask(title)
            // После добавления можно перезагрузить список или оптимистично обновить
            loadTasks()
        }
    }

    fun deleteTask(task: TaskEntity) {
        viewModelScope.launch {
            repository.deleteTask(task)
            loadTasks()
        }
    }

    fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean) {
        viewModelScope.launch {
            repository.toggleTaskCompletion(task, isCompleted)
            loadTasks()
        }
    }

    fun refresh() {
        loadTasks()
    }
}
```
### `TaskRepository.kt`
```kotlin
package com.example.app12.data.repository

import com.example.app12.database.TaskEntity
import kotlinx.coroutines.flow.Flow

interface TaskRepository {
    fun getAllTasks(): Flow<List<TaskEntity>>
    suspend fun addTask(title: String)
    suspend fun deleteTask(task: TaskEntity)
    suspend fun updateTask(task: TaskEntity)
    suspend fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean)
    suspend fun deleteAllTasks()

    suspend fun getTasksOnce(): List<TaskEntity>
}
```
### `activity_main.xm`
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <EditText
        android:id="@+id/editTextTask"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="@string/hint_input"
        android:layout_marginBottom="8dp"/>

    <Button
        android:id="@+id/buttonAddTask"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/button_add_task"
        android:layout_marginBottom="16dp"/>

    <Button
        android:id="@+id/buttonRefresh"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Обновить"
        android:layout_marginBottom="16dp"
        />

    <!-- RecyclerView для списка задач -->

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recyclerViewTasks"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:visibility="gone"/>

        <ProgressBar
            android:id="@+id/progressBar"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:visibility="gone"/>

        <TextView
            android:id="@+id/textError"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:text="Ошибка загрузки"
            android:visibility="gone"/>

    </FrameLayout>
</LinearLayout>

```
### `MainActivity.kt`
```kotlin
package com.example.app12

import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import android.content.Intent
import androidx.activity.viewModels
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import androidx.lifecycle.Lifecycle
import kotlinx.coroutines.launch
import com.example.app12.database.AppDatabase
import com.example.app12.data.repository.TaskRepositoryImpl
import com.example.app12.ui.TasksUiState
import android.widget.ProgressBar
import android.widget.TextView
import android.view.View


class MainActivity : AppCompatActivity() {
    private val database by lazy { AppDatabase.getInstance(this) }
    private val repository by lazy { TaskRepositoryImpl(database.taskDao()) }
    private val viewModel: MainViewModel by viewModels {
        MainViewModelFactory(repository)
    }
    private lateinit var adapter: TaskAdapter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val editTextTask = findViewById<EditText>(R.id.editTextTask)
        val buttonAddTask = findViewById<Button>(R.id.buttonAddTask)
        val recyclerView = findViewById<RecyclerView>(R.id.recyclerViewTasks)

        // Настройка RecyclerView
        recyclerView.layoutManager = LinearLayoutManager(this)
        adapter = TaskAdapter(
            tasks = emptyList(),
            onItemClick = { task ->
                val intent = Intent(this, DetailActivity::class.java)
                intent.putExtra("task_text", task.title)
                startActivity(intent)
            },
            onItemLongClick = { task ->
                viewModel.deleteTask(task)
                Toast.makeText(this, "Задача удалена", Toast.LENGTH_SHORT).show()
            },
            onCheckChange = { task, isChecked ->
                viewModel.toggleTaskCompletion(task, isChecked)
            }
        )
        recyclerView.adapter = adapter

        // Подписка на изменения списка задач
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { state ->
                    when (state) {
                        is TasksUiState.Loading -> {
                            findViewById<RecyclerView>(R.id.recyclerViewTasks).visibility = View.GONE
                            findViewById<ProgressBar>(R.id.progressBar).visibility = View.VISIBLE
                            findViewById<TextView>(R.id.textError).visibility = View.GONE
                        }
                        is TasksUiState.Success -> {
                            findViewById<RecyclerView>(R.id.recyclerViewTasks).visibility = View.VISIBLE
                            findViewById<ProgressBar>(R.id.progressBar).visibility = View.GONE
                            findViewById<TextView>(R.id.textError).visibility = View.GONE
                            adapter.updateData(state.tasks)
                        }
                        is TasksUiState.Error -> {
                            findViewById<RecyclerView>(R.id.recyclerViewTasks).visibility = View.GONE
                            findViewById<ProgressBar>(R.id.progressBar).visibility = View.GONE
                            findViewById<TextView>(R.id.textError).visibility = View.VISIBLE
                            findViewById<TextView>(R.id.textError).text = state.message
                        }
                    }
                }
            }
        }

        // Добавление задачи
        buttonAddTask.setOnClickListener {
            val task = editTextTask.text.toString()
            if (task.isNotBlank()) {
                viewModel.addTask(task)
                editTextTask.text.clear()
            } else {
                Toast.makeText(this, "Введите задачу", Toast.LENGTH_SHORT).show()
            }
        }
        findViewById<Button>(R.id.buttonRefresh).setOnClickListener {
            viewModel.refresh()
        }
        // Загрузим тестовые данные при первом запуске (если список пуст)
        //if (viewModel.tasks.value.isEmpty()) {
         //   viewModel.loadTestData()
        //}
    }
}
```

## Работающее приложение на виртуальном устройстве
<img width="339" height="713" alt="image" src="https://github.com/user-attachments/assets/137c77bd-51b8-4154-a8c9-72811287b50b" />

## Ответы
**1. Почему длительные операции нельзя выполнять в главном потоке?**

Главный поток отвечает за отрисовку UI. Если его заблокировать — приложение зависнет.

**2. Что такое viewModelScope и как он связан с жизненным циклом ViewModel?**

Встроенный CoroutineScope, привязанный к ViewModel. Корутины запущенные в нём автоматически отменяются при уничтожении ViewModel (при закрытии экрана).

**3. Какие преимущества даёт использование sealed class для представления состояний UI?**

- Все возможные состояния явно перечислены (Success, Loading, Error)

- Безопасность: when покрывает все варианты без else

- Можно хранить разные данные в разных состояниях

**4. Как имитировать задержку в корутине?**

delay(время в миллисекундах)

**5. Как обрабатывать ошибки при выполнении корутин?**

- try-catch внутри корутины

- Использовать .catch { } оператор для Flow

- Перехватывать в CoroutineExceptionHandler (реже на практике)

## Выводы

В Android Studio можно вынести репозиторий, корутины и sealed class'ы, чтобы при повороте экрана данные не терялись, а при замене SQLite на Room ничего не сломалось.
