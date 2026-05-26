<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8b330702-0ae8-4771-aebc-6bb1afa85c62" /><div align="center">

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

Лабораторная работа №13

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

## Цель работы: Научиться выполнять сетевые запросы в Android-приложении с использованием библиотеки Retrofit и корутин, обрабатывать ответы сервера, парсить JSON-данные и отображать их в RecyclerView.

---
### `Post.kt`
```kotlin
package com.example.postsapp.models

data class Post(
    val id: Int,
    val title: String,
    val body: String
)
```
### `ApiService.kt`
```kotlin
package com.example.postsapp.api

import com.example.postsapp.models.Post
import retrofit2.http.GET

interface ApiService {
    @GET("posts")
    suspend fun getPosts(): List<Post>
}
```
### `RetrofitClient.kt`
```kotlin
package com.example.postsapp.api

import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

object RetrofitClient {
    private const val BASE_URL = "https://jsonplaceholder.typicode.com/"

    private val retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    val apiService: ApiService = retrofit.create(ApiService::class.java)
}
```
### `PostsRepository.kt`
```kotlin
package com.example.postsapp.repositories

import com.example.postsapp.api.RetrofitClient
import com.example.postsapp.models.Post
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext

class PostsRepository {
    private val apiService = RetrofitClient.apiService

    suspend fun getPosts(): List<Post> = withContext(Dispatchers.IO) {
        apiService.getPosts()
    }
}
```
### `PostsViewModel.kt`
```kotlin
package com.example.postsapp.viewmodels

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.postsapp.models.Post
import com.example.postsapp.repositories.PostsRepository
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch

sealed class PostsUiState {
    object Loading : PostsUiState()
    data class Success(val posts: List<Post>) : PostsUiState()
    data class Error(val message: String) : PostsUiState()
}

class PostsViewModel : ViewModel() {
    private val repository = PostsRepository()

    private val _uiState = MutableStateFlow<PostsUiState>(PostsUiState.Loading)
    val uiState: StateFlow<PostsUiState> = _uiState.asStateFlow()

    init {
        loadPosts()
    }

    fun loadPosts() {
        viewModelScope.launch {
            _uiState.value = PostsUiState.Loading
            try {
                val posts = repository.getPosts()
                _uiState.value = PostsUiState.Success(posts)
            } catch (e: Exception) {
                _uiState.value = PostsUiState.Error(e.message ?: "Unknown error")
            }
        }
    }
}
```
### `PostsAdapter.kt`
```kotlin
package com.example.postsapp.adapters

import android.content.Intent
import android.view.LayoutInflater
import android.view.ViewGroup
import androidx.recyclerview.widget.RecyclerView
import com.example.postsapp.R
import com.example.postsapp.SecondActivity
import com.example.postsapp.models.Post

class PostsAdapter : RecyclerView.Adapter<PostsAdapter.PostViewHolder>() {

    private var posts = emptyList<Post>()

    fun submitList(newPosts: List<Post>) {
        posts = newPosts
        notifyDataSetChanged()
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PostViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_post, parent, false)
        return PostViewHolder(view)
    }

    override fun onBindViewHolder(holder: PostViewHolder, position: Int) {
        holder.bind(posts[position])
    }

    override fun getItemCount() = posts.size

    inner class PostViewHolder(itemView: android.view.View) :
        RecyclerView.ViewHolder(itemView) {

        private val textPostId = itemView.findViewById<android.widget.TextView>(R.id.textPostId)
        private val textPostTitle = itemView.findViewById<android.widget.TextView>(R.id.textPostTitle)
        private val textPostBody = itemView.findViewById<android.widget.TextView>(R.id.textPostBody)

        fun bind(post: Post) {
            textPostId.text = "ID: ${post.id}"
            textPostTitle.text = post.title
            textPostBody.text = post.body

            // Обработчик клика по карточке
            itemView.setOnClickListener {
                val context = itemView.context
                val intent = Intent(context, SecondActivity::class.java).apply {
                    putExtra("POST_ID", post.id)
                    putExtra("POST_TITLE", post.title)
                    putExtra("POST_BODY", post.body)
                }
                context.startActivity(intent)
            }
        }
    }
}
```
### `MainActivity.kt`
```kotlin
package com.example.postsapp

import android.os.Bundle
import android.widget.Button
import android.widget.ProgressBar
import android.widget.TextView
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.example.postsapp.adapters.PostsAdapter
import com.example.postsapp.viewmodels.PostsUiState
import com.example.postsapp.viewmodels.PostsViewModel
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private val viewModel: PostsViewModel by viewModels()
    private lateinit var adapter: PostsAdapter
    private lateinit var recyclerView: RecyclerView
    private lateinit var progressBar: ProgressBar
    private lateinit var textError: TextView
    private lateinit var buttonRefresh: Button

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        recyclerView = findViewById(R.id.recyclerViewPosts)
        progressBar = findViewById(R.id.progressBar)
        textError = findViewById(R.id.textError)
        buttonRefresh = findViewById(R.id.buttonRefresh)

        setupRecyclerView()
        observeUiState()

        buttonRefresh.setOnClickListener {
            viewModel.loadPosts()
        }
    }

    private fun setupRecyclerView() {
        adapter = PostsAdapter()
        recyclerView.layoutManager = LinearLayoutManager(this)
        recyclerView.adapter = adapter
    }

    private fun observeUiState() {
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { state ->
                    when (state) {
                        is PostsUiState.Loading -> showLoading()
                        is PostsUiState.Success -> showPosts(state.posts)
                        is PostsUiState.Error -> showError(state.message)
                    }
                }
            }
        }
    }

    private fun showLoading() {
        recyclerView.visibility = android.view.View.GONE
        progressBar.visibility = android.view.View.VISIBLE
        textError.visibility = android.view.View.GONE
    }

    private fun showPosts(posts: List<com.example.postsapp.models.Post>) {
        recyclerView.visibility = android.view.View.VISIBLE
        progressBar.visibility = android.view.View.GONE
        textError.visibility = android.view.View.GONE
        adapter.submitList(posts)
    }

    private fun showError(message: String) {
        recyclerView.visibility = android.view.View.GONE
        progressBar.visibility = android.view.View.GONE
        textError.visibility = android.view.View.VISIBLE
        textError.text = "Ошибка: $message"
    }
}
```
### `activity_main.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:id="@+id/buttonRefresh"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Обновить"/>

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recyclerViewPosts"
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
### `item_post.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <TextView
            android:id="@+id/textPostId"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="ID: "
            android:textStyle="bold"
            android:textSize="14sp"/>

        <TextView
            android:id="@+id/textPostTitle"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Title"
            android:textSize="18sp"
            android:textStyle="bold"
            android:layout_marginTop="4dp"/>

        <TextView
            android:id="@+id/textPostBody"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Body"
            android:textSize="14sp"
            android:layout_marginTop="8dp"/>

    </LinearLayout>
</androidx.cardview.widget.CardView>
```

## Работающее приложение на виртуальном устройстве
<img width="361" height="775" alt="image" src="https://github.com/user-attachments/assets/21199879-c283-4354-8ecc-12ad34202bed" />


## Ответы
**1. Для чего используется библиотека Retrofit? Какие аннотации вы знаете?**

Retrofit — для HTTP-запросов к API. Аннотации: @GET, @POST, @PUT, @DELETE, @Path, @Query, @Body.

**2. Почему сетевые запросы нельзя выполнять в главном потоке?**

Сетевые запросы долгие — заблокируют UI, приложение зависнет.

**3. Что такое suspend функция и как она работает с корутинами?**

suspend — функция, которая может приостановиться (не блокируя поток) и возобновиться позже, работает только внутри корутины.

**4. Для чего нужен Dispatchers.IO?**

Dispatchers.IO — оптимизирован для операций ввода-вывода (сеть, диск, БД), запускает корутины в пуле потоков для I/O.

**5. Как обрабатывать ошибки при сетевых запросах?**

Обработка ошибок: try-catch вокруг вызова API, проверка response.isSuccessful, использование sealed class (Success/Error) в UI.

**6. Что такое JSONPlaceholder и для чего он используется?**

JSONPlaceholder — фейковое REST API для тестирования и обучения (посты, комментарии, пользователи).

## Выводы

В Android Studio можно обернуть сетевые запросы в Retrofit с аннотациями, запустить их в корутине с Dispatchers.IO, обработать ошибки через try-catch, а результат отправить в sealed class — чтобы главный поток рисовал экран, а не ждал ответа от JSONPlaceholder.
