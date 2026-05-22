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

Лабораторная работа №8  
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

## Цель работы: Изучить архитектурный компонент ViewModel, научиться выносить логику и состояние UI из Activity, использовать StateFlow для реактивного обновления данных, обеспечить сохранение состояния при изменении конфигурации.

---

### `MainViewModel.kt`
```kotlin
package com.example.app8

import androidx.lifecycle.ViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow

class MainViewModel : ViewModel() {

    // Приватный изменяемый StateFlow с начальным значением (пустой список)
    private val _tasks = MutableStateFlow<List<String>>(emptyList())

    // Публичный неизменяемый StateFlow для подписки из UI
    val tasks: StateFlow<List<String>> = _tasks.asStateFlow()

    // Добавление новой задачи
    fun addTask(task: String) {
        val currentList = _tasks.value.toMutableList()
        currentList.add(task)
        _tasks.value = currentList
    }

    // Удаление задачи по индексу
    fun deleteTask(index: Int) {
        val currentList = _tasks.value.toMutableList()
        if (index in currentList.indices) {
            currentList.removeAt(index)
            _tasks.value = currentList
        }
    }

    // Обновление текста задачи
    fun updateTask(index: Int, newText: String) {
        val currentList = _tasks.value.toMutableList()
        if (index in currentList.indices) {
            currentList[index] = newText
            _tasks.value = currentList
        }
    }

    // Вспомогательный метод для инициализации тестовыми данными (если нужно)
    fun loadTestData() {
        _tasks.value = listOf(
            "Купить продукты",
            "Сделать ДЗ по Android",
            "Позвонить маме",
            "Записаться к врачу"
        )
    }
}
```
### `MainActivity.kt`
```kotlin
package com.example.app8

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

class MainActivity : AppCompatActivity() {

    private val viewModel: MainViewModel by viewModels()
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
            tasks = emptyList(), // адаптер будет обновляться через submitList или подобное
            onItemClick = { position ->
                val taskText = viewModel.tasks.value[position]
                val intent = Intent(this, DetailActivity::class.java)
                intent.putExtra("task_text", taskText)
                startActivity(intent)
            },
            onItemLongClick = { position ->
                viewModel.deleteTask(position)
                Toast.makeText(this, "Задача удалена", Toast.LENGTH_SHORT).show()
            }
        )
        recyclerView.adapter = adapter

        // Подписка на изменения списка задач
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.tasks.collect { tasks ->
                    adapter.updateData(tasks) // предполагаем, что у адаптера есть такой метод
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
        if (viewModel.tasks.value.isEmpty()) {
            viewModel.loadTestData()
        }
    }
}
```
### `TaskAdapter.kt`
```kotlin
package com.example.app8

import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.CheckBox
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView

class TaskAdapter(
    private var tasks: List<String>,
    private val onItemClick: (Int) -> Unit,
    private val onItemLongClick: (Int) -> Unit
) : RecyclerView.Adapter<TaskAdapter.TaskViewHolder>() {

    class TaskViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val textTask: TextView = itemView.findViewById(R.id.textTask)
        val checkTask: CheckBox = itemView.findViewById(R.id.checkTask)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): TaskViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_task, parent, false)
        return TaskViewHolder(view)
    }

    override fun onBindViewHolder(holder: TaskViewHolder, position: Int) {
        val task = tasks[position]
        holder.textTask.text = task

        holder.itemView.setOnClickListener {
            onItemClick(position)
        }

        holder.itemView.setOnLongClickListener {
            onItemLongClick(position)
            true
        }

        // (Опционально) логика чекбокса из Лаб.6
        holder.checkTask.setOnCheckedChangeListener { _, isChecked ->
            if (isChecked) {
                holder.textTask.paintFlags = holder.textTask.paintFlags or android.graphics.Paint.STRIKE_THRU_TEXT_FLAG
            } else {
                holder.textTask.paintFlags = holder.textTask.paintFlags and android.graphics.Paint.STRIKE_THRU_TEXT_FLAG.inv()
            }
        }
    }

    override fun getItemCount(): Int = tasks.size

    fun updateData(newTasks: List<String>) {
        tasks = newTasks
        notifyDataSetChanged()
    }
}
```

## Работающее приложение на виртуальном устройстве
<img width="339" height="718" alt="image" src="https://github.com/user-attachments/assets/3c740d84-ca29-4a68-8136-9c5a9d0b7881" />


## Ответы
**1. Для чего нужен ViewModel? Как он помогает при повороте экрана?**
Хранит и управляет данными, связанными с UI.
При повороте экрана ViewModel не пересоздаётся, а сохраняет все данные (счётчик, список, состояния), восстанавливая их автоматически.

**2. Чем StateFlow отличается от LiveData? В каких случаях предпочтительнее использовать StateFlow?**
LiveData — привязана к жизненному циклу.

StateFlow — корутинная (независима от платформы), имеет единый поток с начальным значением.
Предпочтительнее StateFlow в проектах с Flow, корутинами, Compose или при сложных цепочках преобразований.

**3. Что такое lifecycleScope и repeatOnLifecycle? Зачем они нужны при подписке на StateFlow?**
lifecycleScope — корутина в рамках жизненного цикла Activity/Fragment.
repeatOnLifecycle — перезапускает сбор потока при переходе в STARTED, останавливает при STOPPED, экономя ресурсы.

**4. Как обновить данные в StateFlow?**
Через MutableStateFlow и метод .value = новое_значение или .update { текущее -> новое }.

**5. Какие преимущества даёт вынос логики в ViewModel с точки зрения тестирования?**
- Чистый Kotlin без Android-зависимостей (можно тестировать на JVM)

- Лёгкое создание фальшивых данных (mock-репозитории)

- Готовое тестирование состояний через StateFlow (например, .value проверка)


## Выводы
В Android studio есть множество полезных инструментов, способов для работы с данными и их отображением на экране, при этом может экономить ресурсы системы.
