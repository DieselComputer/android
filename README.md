
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

Лабораторная работа №2  
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

## Цель работы: Написание консольных утилит на Kotlin внутри Android проекта. Расчеты, работа со строками. Подготовка классов данных для будущего приложения.

---

## Созданные классы данных и функций-утилит

### 1. `book.kt`
```kotlin
package com.example.app2.utils

data class book(
    val title: String,
    val author: String,
    val year: Int,
    val price: Double
)
```

### 2. `StringUtils.kt`
```kotlin
package com.example.app2.utils

// Проверка, что строка похожа на email (содержит @ и .)
fun String.isValidEmail(): Boolean {
    return this.contains("@") && this.contains(".")
}

// Форматирование автора: "Толстой Л.Н."
fun formatAuthorName(fullName: String): String {
    val parts = fullName.split(" ").filter { it.isNotBlank() }
    return when (parts.size) {
        1 -> parts[0]  // только фамилия
        2 -> "${parts[0]} ${parts[1].first()}."  // фамилия и инициал
        3 -> "${parts[0]} ${parts[1].first()}.${parts[2].first()}."  // фамилия и два инициала
        else -> fullName
    }
}

// Применение скидки к цене книги
fun applyDiscount(price: Double, discountPercent: Double): Double {
    require(discountPercent in 0.0..100.0) { "Скидка должна быть от 0 до 100" }
    return price * (1 - discountPercent / 100)
}
```

### 3. `PasswordValidator.kt`
```kotlin
package com.example.app2.utils

object PasswordValidator {
    fun validate(password: String): String {
        val errors = mutableListOf<String>()

        // проверка длины
        if (password.length < 8) {
            errors.add("Пароль должен содержать минимум 8 символов")
        }

        // проверка наличия цифр
        if (!password.any { it.isDigit() }) {
            errors.add("Пароль должен содержать хотя бы одну цифру")
        }

        // проверка наличия заглавных букв
        if (!password.any { it.isUpperCase() }) {
            errors.add("Пароль должен содержать хотя бы одну заглавную букву")
        }

        // проверка наличия спецсимволов
        val specialCharacters = "!@#\$%^&*()_+-=[]{}|;:,.<>?/~"
        if (!password.any { it in specialCharacters }) {
            errors.add("Пароль должен содержать хотя бы один спецсимвол")
        }

        return if (errors.isEmpty()) {
            "Пароль надёжный"
        } else {
            errors.joinToString("\n")
        }
    }
}
```
### 4. `MainActivity.kt`
```kotlin
package com.example.app2

import android.os.Bundle
import androidx.activity.enableEdgeToEdge
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import com.example.app2.utils.*
import android.widget.TextView
import android.graphics.Color
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()

        setContentView(R.layout.activity_main)
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main)) { v, insets ->
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }
        val book = book("Война и мир", "Толстой Лев Николаевич", 1869, 500.0)
        val formattedAuthor = formatAuthorName(book.author)
        val discountedPrice = applyDiscount(book.price, 15.0)

        findViewById<TextView>(R.id.textView1).text = "Книга: ${book.title}"
        findViewById<TextView>(R.id.textView2).text = "Автор: $formattedAuthor"
        findViewById<TextView>(R.id.textView3).text = "Цена со скидкой: $discountedPrice руб."

        val resultText = findViewById<TextView>(R.id.resultText)

        // проверка пароля
        val password = "E54r9@iiii"
        val validationResult = PasswordValidator.validate(password)
        resultText.text = validationResult

        if (validationResult == "Пароль надёжный") {
            resultText.setTextColor(Color.GREEN)
        } else {
            resultText.setTextColor(Color.RED)
        }

    }
}
```

### 5. `activity_main.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"

    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/textView1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="#9C27B0"
        android:textSize="24sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.497"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.366" />

    <TextView
        android:id="@+id/textView2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="70dp"
        android:textColor="#9C27B0"
        android:textSize="24sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.498"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.351" />


    <TextView
        android:id="@+id/textView3"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="70dp"
        android:textColor="#9C27B0"
        android:textSize="24sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.498"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.402" />

    <TextView
        android:id="@+id/resultText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="70dp"
        android:textColor="#9C27B0"
        android:textSize="24sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.498"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.484" />

    <TextView

        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="@string/greeting"
        android:textColor="#0000FF"
        android:textSize="24sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.498"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.251" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

## Листинг юнит-тестов
### `StringUtilsTest.kt`
```kotlin
package com.example.app2.utils

import org.junit.Assert.*
import org.junit.Test

class StringUtilsTest {
    @Test
    fun emailValidation_correct() {
        assertTrue("test@example.com".isValidEmail())
        assertTrue("user.name@domain.co".isValidEmail())
    }

    @Test
    fun emailValidation_incorrect() {
        assertFalse("testexample.com".isValidEmail())
        assertFalse("test@example".isValidEmail())
        assertFalse("".isValidEmail())
    }

    @Test
    fun formatAuthorName_fullName() {
        assertEquals("Толстой Л.Н.", formatAuthorName("Толстой Лев Николаевич"))
        assertEquals("Пушкин А.С.", formatAuthorName("Пушкин Александр Сергеевич"))
    }

    @Test
    fun formatAuthorName_twoParts() {
        assertEquals("Толстой Л.", formatAuthorName("Толстой Лев"))
        assertEquals("Пушкин А.", formatAuthorName("Пушкин Александр"))
    }

    @Test
    fun formatAuthorName_onePart() {
        assertEquals("Толстой", formatAuthorName("Толстой"))
    }

    @Test
    fun applyDiscount_normal() {
        assertEquals(90.0, applyDiscount(100.0, 10.0), 0.001)
        assertEquals(75.0, applyDiscount(150.0, 50.0), 0.001)
    }

    @Test
    fun applyDiscount_zero() {
        assertEquals(100.0, applyDiscount(100.0, 0.0), 0.001)
    }

    @Test(expected = IllegalArgumentException::class)
    fun applyDiscount_invalidLow() {
        applyDiscount(100.0, -5.0)
    }

    @Test(expected = IllegalArgumentException::class)
    fun applyDiscount_invalidHigh() {
        applyDiscount(100.0, 110.0)
    }
    @Test
    fun testValidPassword() {
        assertEquals("Пароль надёжный", PasswordValidator.validate("StrongP@ss1"))
        assertEquals("Пароль надёжный", PasswordValidator.validate("Pass123!"))
        assertEquals("Пароль надёжный", PasswordValidator.validate("Complex#Password99"))
    }


    @Test
    fun testPasswordWithoutDigits() {
        val result = PasswordValidator.validate("NoDigits!@#")
        assertEquals(
            "Пароль должен содержать хотя бы одну цифру",
            result
        )
    }

    @Test
    fun testPasswordWithoutUppercase() {
        val result = PasswordValidator.validate("nouppercase123!")
        assertEquals(
            "Пароль должен содержать хотя бы одну заглавную букву",
            result
        )
    }

    @Test
    fun testPasswordWithoutSpecialChars() {
        val result = PasswordValidator.validate("NoSpecial123")
        assertEquals(
            "Пароль должен содержать хотя бы один спецсимвол",
            result
        )
    }

    @Test
    fun testEmptyPassword() {
        val result = PasswordValidator.validate("")
        assertEquals(
            "Пароль должен содержать минимум 8 символов\n" +
                    "Пароль должен содержать хотя бы одну цифру\n" +
                    "Пароль должен содержать хотя бы одну заглавную букву\n" +
                    "Пароль должен содержать хотя бы один спецсимвол",
            result
        )
    }
}
```

## Выполненные тесты
<img width="540" height="149" alt="image" src="https://github.com/user-attachments/assets/839c5104-aa36-4697-afd1-5d92ae280043" />

## Работающее приложение на виртуальном устройстве
<img width="365" height="779" alt="image" src="https://github.com/user-attachments/assets/3a4c1e1d-fba9-4289-887c-c6c0f6214c70" />

## Ответы
**1. Для чего в Kotlin используются data class?**

Data class в Kotlin используются для классов, основная цель которых — хранение данных. Они автоматически генерируют стандартные методы.

**2. Чем отличается функция расширения от обычной функции?**

Функция расширения позволяет добавить новую функциональность к существующему классу без наследования.

**3. Как запустить юнит-тесты в Android Studio?**
1. Открыть тестовый класс, нажать правой кнопкой мыши на классе или методе, выбрать "Run 'TestClassName'"

2. Нажать на зеленый треугольник слева от класса/метода непосредственно в файле

3. Запустить через верхнюю кнопку, поменяв 'app' на 'TestClassName'

**4. Что такое assertEquals и для чего нужен третий параметр (дельта) при сравнении вещественных чисел?** 

assertEquals — метод JUnit для проверки равенства ожидаемого и фактического значений.

Третий параметр (дельта) нужен при сравнении вещественных чисел из-за погрешностей вычислений.

**5. В какой директории проекта хранятся тесты, выполняющиеся на JVM?**

В стандартном проекте Android тесты хранятся в двух основных директориях:

test/ — директория для unit-тестов, которые выполняются на локальной JVM (быстрые, не требуют эмулятора)

androidTest/ — для инструментальных тестов, требующих Android-устройство или эмулятор

## Выводы

**Android Studio** позволяет проводить тесты без необходимости запускать приложение на виртуальной/реальной машине с помощью созданного класса StringUtilsTest. В процессе были изучены Функции расширения и обработка исключений, а также создан класс данных.
