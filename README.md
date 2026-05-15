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

Лабораторная работа №5  
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

## Цель работы: Научиться обрабатывать пользовательский ввод, работать с состоянием (счетчик, список задач), динамически обновлять интерфейс приложения на Kotlin.

---

### `activity_main.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <!-- Блок 1: Счётчик -->
    <TextView
        android:id="@+id/textCounter"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/counter_text"
        android:textSize="24sp"
        android:layout_marginBottom="16dp"/>

    <Button
        android:id="@+id/buttonIncrement"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/button_increment"
        android:layout_marginBottom="24dp"/>

    <Button
        android:id="@+id/buttonReset"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/button_clear"
        android:layout_marginBottom="24dp"/>

    <!-- Блок 2: Поле ввода и отображение текста -->
    <EditText
        android:id="@+id/editTextInput"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="@string/hint_input"
        android:inputType="text"
        android:layout_marginBottom="8dp"/>

    <Button
        android:id="@+id/buttonShow"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/button_show"
        android:layout_marginBottom="8dp"/>

    <TextView
        android:id="@+id/textEntered"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/label_entered"
        android:textSize="18sp"
        android:layout_marginBottom="24dp"/>

    <!-- Блок 3: ToDo список -->
    <EditText
        android:id="@+id/editTextTask"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="@string/hint_input"
        android:inputType="text"
        android:layout_marginBottom="8dp"/>

    <Button
        android:id="@+id/buttonAddTask"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/button_add_task"
        android:layout_marginBottom="8dp"/>

    <Button
        android:id="@+id/buttonClearAll"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/button_clear_all"
        android:layout_marginBottom="8dp"/>


    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:layout_marginBottom="8dp">

        <EditText
            android:id="@+id/editTextIndex"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:hint="@string/edit_index"
            android:inputType="number"
            android:layout_marginEnd="8dp"/>

        <Button
            android:id="@+id/buttonDeleteTask"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/button_remove"/>
    </LinearLayout>

    <!-- ScrollView для прокручиваемого списка задач -->
    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:background="#F0F0F0">

        <TextView
            android:id="@+id/textTasks"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/label_tasks"
            android:textSize="18sp"
            android:padding="8dp"/>

    </ScrollView>

</LinearLayout>
```
### `MainActivity.kt`
```kotlin
package com.example.app5

import android.content.Context
import android.content.SharedPreferences
import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {

    private var counter = 0
    private val tasks = mutableListOf<String>()
    private lateinit var sharedPreferences: SharedPreferences

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        sharedPreferences = getSharedPreferences("TodoAppPrefs", Context.MODE_PRIVATE)

        loadSavedData()

        val textCounter = findViewById<TextView>(R.id.textCounter)
        val buttonIncrement = findViewById<Button>(R.id.buttonIncrement)

        val editTextInput = findViewById<EditText>(R.id.editTextInput)
        val buttonShow = findViewById<Button>(R.id.buttonShow)
        val textEntered = findViewById<TextView>(R.id.textEntered)

        val editTextTask = findViewById<EditText>(R.id.editTextTask)
        val buttonAddTask = findViewById<Button>(R.id.buttonAddTask)
        val textTasks = findViewById<TextView>(R.id.textTasks)

        val editTextIndex = findViewById<EditText>(R.id.editTextIndex)
        val buttonDeleteTask = findViewById<Button>(R.id.buttonDeleteTask)

        updateCounterDisplay(textCounter)

        buttonIncrement.setOnClickListener {
            counter++
            updateCounterDisplay(textCounter)
            saveCounter()
        }

        val buttonReset = findViewById<Button>(R.id.buttonReset)
        buttonReset.setOnClickListener {
            counter = 0
            updateCounterDisplay(textCounter)
            saveCounter()
        }

        val buttonClearAll = findViewById<Button>(R.id.buttonClearAll)
        buttonClearAll.setOnClickListener {
            tasks.clear()
            updateTasksDisplay(textTasks)
            saveTasks()
            Toast.makeText(this, "Все задачи удалены", Toast.LENGTH_SHORT).show()
        }

        buttonShow.setOnClickListener {
            val inputText = editTextInput.text.toString()
            if (inputText.isNotBlank()) {
                textEntered.text = getString(R.string.label_entered) + " $inputText"
            } else {
                Toast.makeText(this, "Введите текст", Toast.LENGTH_SHORT).show()
            }
        }

        buttonAddTask.setOnClickListener {
            val task = editTextTask.text.toString()
            if (task.isNotBlank()) {
                tasks.add(task)
                updateTasksDisplay(textTasks)
                saveTasks() // Сохраняем список задач
                editTextTask.text.clear()
                Toast.makeText(this, "Задача добавлена", Toast.LENGTH_SHORT).show()
            } else {
                Toast.makeText(this, "Введите задачу", Toast.LENGTH_SHORT).show()
            }
        }

        buttonDeleteTask.setOnClickListener {
            val indexText = editTextIndex.text.toString()
            if (indexText.isNotBlank()) {
                val index = indexText.toIntOrNull()
                if (index != null && index >= 0 && index < tasks.size) {
                    val removedTask = tasks.removeAt(index)
                    updateTasksDisplay(textTasks)
                    saveTasks() // Сохраняем список задач
                    Toast.makeText(this, "Удалена задача: $removedTask", Toast.LENGTH_SHORT).show()
                    editTextIndex.text.clear()
                } else {
                    Toast.makeText(this, "Неверный индекс. Доступно: 0..${tasks.size - 1}", Toast.LENGTH_SHORT).show()
                }
            } else {
                Toast.makeText(this, "Введите индекс задачи", Toast.LENGTH_SHORT).show()
            }
        }
    }

    private fun updateCounterDisplay(textView: TextView) {
        textView.text = getString(R.string.counter_text, counter)
    }

    private fun updateTasksDisplay(textView: TextView) {
        if (tasks.isEmpty()) {
            textView.text = getString(R.string.label_tasks)
        } else {
            textView.text = tasks.mapIndexed { index, task -> "$index. $task" }.joinToString("\n")
        }
    }

    private fun saveCounter() {
        sharedPreferences.edit().putInt("counter", counter).apply()
    }

    private fun saveTasks() {
        val tasksString = tasks.joinToString("|||") // Используем уникальный разделитель
        sharedPreferences.edit().putString("tasks", tasksString).apply()
    }

    private fun loadSavedData() {
        counter = sharedPreferences.getInt("counter", 0)

        val tasksString = sharedPreferences.getString("tasks", "")
        if (!tasksString.isNullOrEmpty()) {
            tasks.clear()
            tasks.addAll(tasksString.split("|||"))
        }

        val textCounter = findViewById<TextView>(R.id.textCounter)
        val textTasks = findViewById<TextView>(R.id.textTasks)

        updateCounterDisplay(textCounter)
        updateTasksDisplay(textTasks)
    }

    override fun onDestroy() {
        super.onDestroy()
        saveCounter()
        saveTasks()
    }

    override fun onPause() {
        super.onPause()
        saveCounter()
        saveTasks()
    }
}
}
```

## Работающее приложение на виртуальном устройстве
<img width="418" height="798" alt="image" src="https://github.com/user-attachments/assets/78c09341-fd17-4a04-9a2d-576ae0024f65" />

## Ответы
**1. Как получить текст из EditText?**
```kotlin
val text = editText.text.toString()
```

**2. Почему при повороте экрана данные (счётчик, список задач) сбрасываются? Как это можно исправить?**
Активность пересоздаётся. Исправление:

1. ViewModel

2. onSaveInstanceState + onRestoreInstanceState

**3. Для чего используется joinToString? Как изменить разделитель?**
Превращает коллекцию в строку.

Разделитель:
```kotlin
list.joinToString(separator = ", ")
```

**4. В чём разница между List и MutableList?**
List — неизменяемый (только чтение)

MutableList — можно добавлять/удалять элементы

**5. Как очистить поле ввода после добавления задачи?**

```kotlin
editText.text.clear()
```
или
```kotlin
editText.setText("")
```

## Выводы
Kotlin в Android studio позволяет обрабатывать пользовательский ввод в зависимости от задач, работать с состоянием (счетчик, список задач), динамически обновлять интерфейс приложения.
