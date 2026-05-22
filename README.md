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

Лабораторная работа №6  
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

## Цель работы: Научиться использовать RecyclerView для отображения списка данных, освоить создание адаптера и ViewHolder, применить CardView для оформления элементов списка.

---

### `item_task.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp"
    app:cardBackgroundColor="#FFFFFF">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="16dp">

        <TextView
            android:id="@+id/textTask"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:textSize="18sp"
            android:textColor="#333333"/>

        <CheckBox
            android:id="@+id/checkTask"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

    </LinearLayout>

</androidx.cardview.widget.CardView>
```
### `TaskAdapter.kt`
```kotlin
package com.example.app5

import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.CheckBox
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView

class TaskAdapter(
    private val tasks: MutableList<String>,
    private val onCheckedChange: (Boolean) -> Unit
) : RecyclerView.Adapter<TaskAdapter.TaskViewHolder>() {

    // ViewHolder хранит ссылки на элементы внутри карточки
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

        holder.checkTask.setOnCheckedChangeListener(null)
        holder.textTask.paintFlags = holder.textTask.paintFlags and android.graphics.Paint.STRIKE_THRU_TEXT_FLAG.inv()
        // Обработка чекбокса (опционально)
        holder.checkTask.setOnCheckedChangeListener { _, isChecked ->
            // Можно добавить логику отметки выполнения, например, перечеркивание текста
            if (isChecked) {
                holder.textTask.paintFlags = holder.textTask.paintFlags or android.graphics.Paint.STRIKE_THRU_TEXT_FLAG
            } else {
                holder.textTask.paintFlags = holder.textTask.paintFlags and android.graphics.Paint.STRIKE_THRU_TEXT_FLAG.inv()
            }
            onCheckedChange(isChecked)
        }

    }

    override fun getItemCount(): Int = tasks.size

    // Метод для обновления списка
    fun updateData(newTasks: List<String>) {
        tasks.clear()
        tasks.addAll(newTasks)
        notifyDataSetChanged()
    }
}
```
### `MainActivity.kt`
```kotlin
package com.example.app5
import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView

class MainActivity : AppCompatActivity() {

    private val tasks = mutableListOf<String>()
    private lateinit var adapter: TaskAdapter
    private var completedTasksCount = 0
    private lateinit var textCounter: TextView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        textCounter = findViewById<TextView>(R.id.textCounter)

        val editTextTask = findViewById<EditText>(R.id.editTextTask)
        val buttonAddTask = findViewById<Button>(R.id.buttonAddTask)
        val recyclerView = findViewById<RecyclerView>(R.id.recyclerViewTasks)

        updateCounterText()
        // Настройка RecyclerView
        recyclerView.layoutManager = LinearLayoutManager(this)
        adapter = TaskAdapter(tasks) { isChecked ->
            // Этот код сработает, когда в списке нажмут на чекбокс
            if (isChecked) {
                completedTasksCount++
            } else {
                completedTasksCount--
            }
            updateCounterText() // Обновляем цифру на экране
        }
        recyclerView.adapter = adapter

        // Добавление задачи
        buttonAddTask.setOnClickListener {
            val task = editTextTask.text.toString()
            if (task.isNotBlank()) {
                tasks.add(task)
                adapter.notifyItemInserted(tasks.size - 1) // более эффективно, чем notifyDataSetChanged
                editTextTask.text.clear()
            } else {
                Toast.makeText(this, "Введите задачу", Toast.LENGTH_SHORT).show()
            }
        }

        // Восстановление данных при повороте (опционально, см. Лаб.5)
        if (savedInstanceState != null) {
            val savedTasks = savedInstanceState.getStringArrayList("tasks")
            if (savedTasks != null) {
                tasks.clear()
                tasks.addAll(savedTasks)
                adapter.notifyDataSetChanged()
            }
            updateCounterText()
        }
    }
    private fun updateCounterText() {
        // Берем строку из strings.xml и подставляем туда наше число
        textCounter.text = getString(R.string.counter_text, completedTasksCount)
    }
    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putStringArrayList("tasks", ArrayList(tasks))
    }
}
```
## Работающее приложение на виртуальном устройстве


## Ответы
**1. Для чего нужен RecyclerView? Чем он лучше ListView?**
Для отображения больших списков с переиспользованием элементов. Лучше ListView производительностью, анимациями, гибким LayoutManager и обязательным ViewHolder.

**2. Какие компоненты необходимы для работы RecyclerView?**
- RecyclerView (в layout)

- Adapter (подвязывает данные)

- ViewHolder (кэширует View)

- LayoutManager (например, LinearLayoutManager)

**3. Что такое ViewHolder и для чего он используется?**
Внутренний класс, который хранит ссылки на View элемента списка. Нужен, чтобы избежать дорогих вызовов findViewById() при каждой перерисовке.

**4. Чем отличается notifyDataSetChanged() от notifyItemInserted()?**
- notifyDataSetChanged() — грубое обновление всего списка (мигание, потеря анимации)

- notifyItemInserted(position) — точное обновление только добавленного элемента (с анимацией)

**5. Как добавить обработку кликов на элементы RecyclerView?**
Внутри onBindViewHolder установить itemView.setOnClickListener { } (через лямбду или интерфейс из адаптера).

## Выводы
В Android studio можно по-разному обрабатывать списки, в том числе с помощью производительных методов, а также обновлять списки и обрабатывать клики на элементы RecyclerView.
