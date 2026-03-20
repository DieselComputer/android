
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

Лабораторная работа №3  
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

## Цель работы: Изучить функциональные методы обработки коллекций в Kotlin (filter, map, sortedBy) на примере списка объектов и вывести результаты в интерфейс Android-приложения.

---


### `Product.kt`
```kotlin
package com.example.app3.models

data class Product(
    val name: String,
    val category: String,
    val price: Double,
    val inStock: Boolean
)
```
### `MainActivity.kt`
```kotlin

package com.example.app3
import android.os.Bundle
import androidx.activity.enableEdgeToEdge
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import com.example.app3.models.Product
import com.example.app3.models.Book
import android.widget.TextView
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContentView(R.layout.activity_main)

        val products = getProducts()

        val books = getBooks()

        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main)) { v, insets ->
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }
        // 1. Исходный список (для наглядности преобразуем в строку)
        val originalText = products.joinToString("\n") { "${it.name} – ${it.price} руб. (${if (it.inStock) "в наличии" else "нет"})" }
        findViewById<TextView>(R.id.textOriginal).text = originalText

        // 2. Фильтр: только товары в наличии
        val inStockProducts = products.filter { it.inStock }
        val inStockText = inStockProducts.joinToString("\n") { "${it.name} – ${it.price} руб." }
        findViewById<TextView>(R.id.textInStock).text = inStockText

        // 3. Цепочка: отфильтровать электронику, отсортировать по цене и получить список строк с названием и ценой
        val electronicsSorted = products
            .filter { it.category == "Электроника" && it.inStock }
            .sortedBy { it.price }
            .map { "${it.name} – ${it.price} руб." }
        val electronicsText = electronicsSorted.joinToString("\n")
        findViewById<TextView>(R.id.textSorted).text = electronicsText


        val booktext = books.joinToString("\n") { "${it.name}, ${it.author}, ${it.pages} стр., ${it.year} год" }
        findViewById<TextView>(R.id.Books1).text = booktext

        // книги после 2000х
        val booksyear = books.filter { it.year >= 2000 }
        val booksyeartxt = booksyear.joinToString("\n"){ "${it.name}, ${it.author}, ${it.pages}, ${it.year}" }
        findViewById<TextView>(R.id.Books2).text = booksyeartxt

        val booksSorted = books
            .sortedBy { it.pages }
            .map { "${it.name} – ${it.pages} стр." }
        val bksText = booksSorted.joinToString("\n")
        findViewById<TextView>(R.id.Books3).text = bksText

        val bksSorted = books
            .map { "${it.name} – ${it.author} " }
        val bsText = bksSorted.joinToString("\n")
        findViewById<TextView>(R.id.Books4).text = bsText
    }
    private fun getProducts(): List<Product> {
        return listOf(
            Product("Ноутбук", "Электроника", 75000.0, true),
            Product("Мышь", "Электроника", 1500.0, true),
            Product("Книга 'Котлин'", "Книги", 1200.0, false),
            Product("Флешка 64GB", "Электроника", 2000.0, true),
            Product("Блокнот", "Канцелярия", 300.0, true),
            Product("Ручка", "Канцелярия", 50.0, false),
            Product("Монитор", "Электроника", 25000.0, true)
        )
    }
    private fun getBooks(): List<Book> {
        return listOf(
            Book("Война и мир", "Лев Толстой", 10000, 1999),
            Book("Оно", "Стивен Кинг", 20000, 2000),
            Book("Зов Ктулху", "Говард Лавкрафт", 30000, 2001),
        )
    }
}
```

## Работающее приложение на виртуальном устройстве
<img width="383" height="759" alt="image" src="https://github.com/user-attachments/assets/7584e23e-0c3d-489a-9fbb-63ff8356a71f" />

## Ответы
**1. Что возвращает функция filter — новый список или изменяет существующий?**

Функция filter возвращает новый список.

**2. В чём разница между sortedBy и sortedByDescending?**

sortedBy сортирует коллекцию по возрастанию (от меньшего к большему) на основе указанного селектора.

sortedByDescending сортирует коллекцию по убыванию (от большего к меньшему).

**3. Как можно объединить несколько условий в filter?**

Несколько условий объединяются с помощью логических операторов (&& — И, || — ИЛИ) внутри лямбда-выражения.

**4. Для чего используется функция map? Приведите пример.** 

map используется для преобразования (трансформации) всех элементов коллекции по заданной формуле. Результатом всегда будет новая коллекция того же размера.
Пример:
```kotlin
val numbers = listOf(1, 2, 3)
val squares = numbers.map { it * it } // Результат: [1, 4, 9]
```

**5. Что такое joinToString и как она работает?**

joinToString — это функция для превращения коллекции в строку. Она проходит по всем элементам, склеивает их через разделитель.

## Выводы

**Android Studio** позволяет форматировать текст, фильтровать данные в списках и работать со строками.
