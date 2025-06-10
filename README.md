Понял. Делаем руководство пользователя для `KoGen Di` по аналогии с предыдущим. Беру нашу финальную статью и превращаю ее в сжатую инструкцию для `README.md`.

Вот готовый текст в формате Markdown.

---
```markdown
# KoGen Di: Руководство пользователя

**KoGen Di** — это библиотека для Android, которая использует кодогенерацию (KSP) для создания простого, независимого и типобезопасного Dependency Injection контейнера, который идеально подходит для многомодульных проектов.

**Основные принципы:**
* **Ноль инициализации:** Не требует привязки к классу `Application`.
* **Независимость:** Каждый модуль может иметь свой собственный, изолированный DI-контейнер.
* **Минимум кода:** Большинство зависимостей регистрируются автоматически с помощью аннотаций.

---

## 🚀 Установка и Настройка

Библиотека опубликована в **Maven Central**.

### Шаг 1: Подключаем плагин KSP

**В файле `build.gradle.kts` корневого проекта:**
```kotlin
plugins {
    // ...
    id("com.google.devtools.ksp") version "2.0.0-1.0.22" apply false // Используйте актуальную версию KSP для вашего Kotlin
}
```

**В файле `build.gradle.kts` вашего модуля:**
```kotlin
plugins {
    // ...
    id("com.google.devtools.ksp")
}
```

### Шаг 2: Добавляем зависимости

```kotlin
dependencies {
    // Ваша версия может отличаться
    implementation("io.github.eugenprog:android-di:1.0.1")
    ksp("io.github.eugenprog:android-di:1.0.1")

    // Зависимости для поддержки ViewModel (нужны, только если вы используете эту функцию)
    implementation(platform("androidx.compose:compose-bom:2024.06.00"))
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.8.2")
}
```
**Важно:** *Версии для `implementation` и `ksp` вашей библиотеки должны совпадать.*

### Шаг 3: Настраиваем кодогенерацию

В `build.gradle.kts` вашего модуля добавьте блок `ksp`.
```kotlin
ksp {
    // Обязательный параметр: пакет вашего проекта
    arg("packageName", "com.myawesome.project")
    // Опциональный параметр для включения поддержки ViewModel
    arg("includeViewModelInjector", "true") 
}
```
`packageName` — **обязательный** параметр. `includeViewModelInjector` — **опциональный**, по умолчанию `false`.

---

## ⚙️ Как пользоваться

### 1. Объявление зависимостей

#### А) Для обычных классов (`@KoGenComponent`)
Пометьте ваш класс (репозиторий, сервис) аннотацией `@KoGenComponent`. Укажите `true`, если он должен быть синглтоном.
```kotlin
@KoGenComponent(singleton = true)
class UserProfileServiceImpl(
    private val source: UserProfileSource,
) : UserProfileService {
    // ...
}
```

#### Б) Для зависимостей со сложным созданием (`@KoGenBean`)
Для объектов, которые требуют сложной логики создания (напр., из `Retrofit`), создайте функцию, которая возвращает этот объект, и пометьте ее аннотацией `@KoGenBean`.
```kotlin
@KoGenBean(singleton = true)
fun provideUserProfileSource(
    context: Context,
): UserProfileSource {
    // ... логика создания объекта
}
```

#### В) Для ViewModel (`@KoGenViewModel`)
Пометьте ваш `ViewModel` соответствующей аннотацией.
```kotlin
@KoGenViewModel
class MyScreenViewModel(
    private val userProfileService: UserProfileService
) : ViewModel() {
    // ...
}
```

### 2. Получение зависимостей

#### Основная точка входа: `inject()`
Используйте эту глобальную функцию в `Activity`, `Fragment` и других классах.
```kotlin
class MyActivity : AppCompatActivity() {
    // Ленивое получение
    private val userProfileService: UserProfileService by inject()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Прямое получение
        val anotherService: AnotherService = inject()
    }
}
```

#### Вторая точка входа: `koGenViewModel()`
Используйте эту функцию в ваших Composable-экранах.
```kotlin
@Composable
fun MyScreen(
    viewModel: MyScreenViewModel = koGenViewModel()
) {
    // ...
}
```
---

## ⚠️ Важные замечания

1.  **Код появляется после первой сборки.** Функции `inject()` и `koGenViewModel()` физически отсутствуют в коде до тех пор, пока вы не соберете проект хотя бы один раз. Не пугайтесь, если IDE будет "ругаться" на их отсутствие.

2.  **Поддержка `ViewModel` — опциональна.** Чтобы ее активировать, нужно передать `arg("includeViewModelInjector", "true")` в настройках KSP.

3.  **KSP иногда "сходит с ума".** В редких случаях стандартное лечение — полная очистка проекта (`./gradlew clean`) и пересборка.

```
