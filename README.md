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

Лабораторная работа №11

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

## Цель работы: Изучить архитектурный паттерн Repository, научиться выделять слой доступа к данным, отделяя его от бизнес-логики, выполнить рефакторинг существующего приложения для использования репозитория.

---
### `TaskRepository.kt`
```kotlin
package com.example.app11.data.repository

import com.example.app11.database.TaskEntity
import kotlinx.coroutines.flow.Flow

interface TaskRepository {
    fun getAllTasks(): Flow<List<TaskEntity>>
    suspend fun addTask(title: String)
    suspend fun deleteTask(task: TaskEntity)
    suspend fun updateTask(task: TaskEntity)
    suspend fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean)
    suspend fun deleteAllTasks()
}
```
### `TaskRepositoryImpl.kt`
```kotlin
package com.example.app11.data.repository

import com.example.app11.database.TaskDao
import com.example.app11.database.TaskEntity
import kotlinx.coroutines.flow.Flow
//import javax.inject.Inject

class TaskRepositoryImpl(
    private val taskDao: TaskDao
) : TaskRepository {

    override fun getAllTasks(): Flow<List<TaskEntity>> = taskDao.getAllTasks()

    override suspend fun addTask(title: String) {
        val task = TaskEntity(title = title)
        taskDao.insertTask(task)
    }

    override suspend fun deleteTask(task: TaskEntity) {
        taskDao.deleteTask(task)
    }

    override suspend fun updateTask(task: TaskEntity) {
        taskDao.updateTask(task)
    }

    override suspend fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean) {
        val updatedTask = task.copy(isCompleted = isCompleted)
        taskDao.updateTask(updatedTask)
    }

    override suspend fun deleteAllTasks() {
        taskDao.deleteAll()
    }
}
```
### `MainViewModel.kt`
```kotlin
package com.example.app11

import androidx.lifecycle.ViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import androidx.lifecycle.viewModelScope
import com.example.app11.database.AppDatabase
import com.example.app11.database.TaskEntity
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.launch
import com.example.app11.data.repository.TaskRepository
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.stateIn

class MainViewModel(
    private val repository: TaskRepository
) : ViewModel() {

    val tasks: StateFlow<List<TaskEntity>> = repository.getAllTasks()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = emptyList()
        )

    fun addTask(title: String) {
        viewModelScope.launch {
            repository.addTask(title)
        }
    }

    fun deleteTask(task: TaskEntity) {
        viewModelScope.launch {
            repository.deleteTask(task)
        }
    }

    fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean) {
        viewModelScope.launch {
            repository.toggleTaskCompletion(task, isCompleted)
        }
    }

    fun deleteAllTasks() {
        viewModelScope.launch {
            repository.deleteAllTasks()
        }
    }
}
```
### `MainViewModelFactory.kt`
```kotlin
package com.example.app11

import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import com.example.app11.data.repository.TaskRepository
import com.example.app11.MainViewModel

class MainViewModelFactory(
    private val repository: TaskRepository
) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(MainViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return MainViewModel(repository) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```
### `MainActivity.kt`
```kotlin
package com.example.app11

import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
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
import com.example.app11.database.AppDatabase
import com.example.app11.data.repository.TaskRepositoryImpl

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
                viewModel.tasks.collect { tasks ->
                    adapter.updateData(tasks)
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

        // Загрузим тестовые данные при первом запуске (если список пуст)
        //if (viewModel.tasks.value.isEmpty()) {
         //   viewModel.loadTestData()
        //}
    }
}
```

## Работающее приложение на виртуальном устройстве
<img width="381" height="821" alt="image" src="https://github.com/user-attachments/assets/6867d46d-9560-499d-a0da-8e277453ef3d" />

## Ответы
**1. Какую роль выполняет слой Repository в архитектуре приложения?**

Repository управляет данными из разных источников (БД, сеть, кэш) и скрывает детали работы с ними от ViewModel.

**2. Какие преимущества даёт использование Repository по сравнению с прямым обращением к DAO из ViewModel?**

- ViewModel не знает, откуда пришли данные (БД, сеть, кэш)

- Легко подменить источник (например, мок для тестов)

- Упрощается добавление кэширования, синхронизации, fallback-логики

**3. Как изменится ViewModel, если мы захотим добавить ещё один источник данных (например, сетевое API)?**

ViewModel не изменится — она по-прежнему вызывает репозиторий. Вся логика выбора или объединения источников останется внутри Repository.

**4. Почему методы репозитория объявлены как suspend?**

Repository выполняет длительные операции (запросы к БД, сети, диску). suspend позволяет вызывать их из корутин, не блокируя главный поток.

**5. Что такое инверсия зависимостей и как она применяется в данном рефакторинге?**

Принцип: зависеть от абстракции, а не от конкретной реализации. В рефакторинге ViewModel получает интерфейс Repository (абстракцию), а не конкретный DAO или API — легко подменить реализацию.

## Выводы

В Android Studio можно отделить слой доступа к данным для более удобного обновления или замены частей или модулей приложения.
