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

Лабораторная работа №4  
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

## Цель работы: Освоить создание пользовательского интерфейса в Android с использованием ConstraintLayout, изучить основные компоненты: ImageView, TextView, Button. Научиться работать с ресурсами (строки, цвета, размеры) и обрабатывать нажатия кнопок.

---

### `activity_main.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/gray_light"
    android:padding="100dp"
    tools:context=".MainActivity">


    <ImageView
        android:id="@+id/imageAvatar"
        android:layout_width="@dimen/avatar_size"
        android:layout_height="@dimen/avatar_size"
        android:layout_marginTop="@dimen/margin_normal"

        android:foreground="@mipmap/ic_launcher_avatar_foreground"
        android:importantForAccessibility="no"
        android:src="@drawable/ic_profile"
        app:layout_constraintBottom_toTopOf="@+id/textName"
        app:layout_constraintHorizontal_bias="0.496"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/textStatus"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="32dp"
        android:text="@string/profile_status"
        android:textColor="@color/purple_500"
        android:textSize="@dimen/text_size_status"
        app:layout_constraintHorizontal_bias="0.494"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/textName" />

    <Button
        android:id="@+id/buttonEdit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="192dp"
        android:backgroundTint="@color/purple_200"
        android:text="@string/button_edit"
        app:cornerRadius="@dimen/button_corner_radius"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/imageAvatar" />

    <Button
        android:id="@+id/buttonExit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:backgroundTint="@color/teal_200"
        android:text="@string/button_exit"
        app:cornerRadius="@dimen/button_corner_radius"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/buttonEdit" />

    <TextView
        android:id="@+id/textName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="72dp"
        android:text="@string/profile_name"
        android:textColor="@color/black"
        android:textSize="@dimen/text_size_name"
        android:textStyle="bold"
        app:layout_constraintHorizontal_bias="0.491"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/imageAvatar" />

    <EditText
        android:id="@+id/editName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="60dp"
        android:autofillHints="Имя"
        android:inputType="text"
        android:text="@string/profile_name"
        android:textColor="@color/black"
        android:textSize="@dimen/text_size_name"
        android:textStyle="bold"
        app:layout_constraintHorizontal_bias="0.491"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/imageAvatar"
        tools:ignore="LabelFor"
        android:visibility="gone"/>
</androidx.constraintlayout.widget.ConstraintLayout>
```
### `MainActivity.kt`
```kotlin
package com.example.app4
import android.content.SharedPreferences
import android.os.Bundle
import android.view.View
import androidx.activity.enableEdgeToEdge
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import kotlin.system.exitProcess

class MainActivity : AppCompatActivity() {
    private lateinit var sharedPref: SharedPreferences

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContentView(R.layout.activity_main)
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main)) { v, insets ->
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }

        sharedPref = getSharedPreferences("my_prefs", MODE_PRIVATE)

        fun saveUsername(username: String) {
            with(sharedPref.edit()) {
                putString("username", username)
                apply()
            }
        }

        var edit = false
        val buttonEdit = findViewById<Button>(R.id.buttonEdit)
        val textEdit = findViewById<EditText>(R.id.editName)
        // val textName = findViewById<TextView>(R.id.textName)
        val textName = findViewById<TextView>(R.id.textName)
        textName.text = sharedPref.getString("username", "")
        val textStatus = findViewById<TextView>(R.id.textStatus)



        buttonEdit.setOnClickListener {
            if (!edit){
                Toast.makeText(this, R.string.toast_message, Toast.LENGTH_SHORT).show()
                textEdit.visibility = View.VISIBLE
                textName.visibility = View.GONE
                textStatus.visibility = View.GONE
                textEdit.setText(textName.text)
                buttonEdit.text = "Сохранить"
                edit = true
            } else {
                textEdit.visibility = View.GONE
                textName.visibility = View.VISIBLE
                textStatus.visibility = View.VISIBLE
                buttonEdit.text = "Редактировать"
                textName.text = textEdit.text
                saveUsername(textName.text.toString())
                edit = false
            }
        }
        
        val buttonExit = findViewById<Button>(R.id.buttonExit)
        buttonExit.setOnClickListener {
            finishAffinity()
            exitProcess(0)
        }
    }
}
```

## Работающее приложение на виртуальном устройстве
<img width="333" height="663" alt="app" src="https://github.com/user-attachments/assets/135f0c86-01c3-40ae-8aba-b123242de0b0" />

## Ответы
**1. Для чего используется ConstraintLayout? Какие у него преимущества перед LinearLayout?**

ConstraintLayout используется для создания гибких плоских иерархий без вложенности. Преимущество: сложный интерфейс с одним корневым макетом (быстрее, чем вложенные LinearLayout).

**2. Что такое app:layout_constraint... атрибуты?**

Это атрибуты, задающие ограничения (constraints) для привязки стороны View к другой View или родителю (например, layout_constraintTop_toBottomOf).

**3. Как вынести размеры и цвета в ресурсы? Зачем это нужно?**

Цвета — в res/values/colors.xml
```xml
<resources>
    <color name="purple_200">#FFBB86FC</color>
    ...
</resources>
```
Размеры — в res/values/dimens.xml
```xml
<resources>
    <dimen name="avatar_size">120dp</dimen>
    ...
</resources>
```
Это нужно для переиспользования, удобная поддержка тем и адаптация под разные экраны.

**4. Каким образом можно обработать клик на кнопке в Kotlin-коде?**

button.setOnClickListener { // код }

**5. Как добавить обработчик нажатия на ImageView?**
imageView.setOnClickListener { // код }

## Выводы
Android Studio позволяет применять рабочие паттерны, использующиеся практически повсеместно в разработке приложений на Android. Среда предоставляет инструменты для создания производительного приложения с минимизированной глубиной вложенности View для ускорения отрисовки интерфейса, а единая модель событий позволяет обрабатывать клик для большиства объектов View одинаково.
