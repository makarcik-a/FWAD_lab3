# Лабораторная работа №3. Основы работы с базами данных в Laravel

## Цель работы

Познакомиться с основными принципами работы с базами данных в Laravel. Научиться создавать миграции, модели и сиды на основе веб-приложения `To-Do App`.

## Условие

В данной лабораторной работе вы продолжите разработку приложения `To-Do App` для команд, начатого в предыдущих лабораторных работах.

Вы добавите функциональность работы с базой данных, создадите модели и миграции, настроите связи между моделями и научитесь использовать фабрики и сиды для генерации тестовых данных.

### №1. Подготовка к работе

> [!TIP]
> Темы: S3

1. Установите СУБД MySQL, PostgreSQL или SQLite на вашем компьютере.
2. Создание базы данных: Создайте новую базу данных для вашего приложения **todo_app**.
3. Настройте переменные окружения в файле `.env` для подключения к базе данных:
   ```
      DB_CONNECTION=sqlite
    # DB_HOST=127.0.0.1
    # DB_PORT=3306
    # DB_DATABASE=laravel
    # DB_USERNAME=root
    # DB_PASSWORD=
   ```

### №2. Создание моделей и миграций

> [!TIP]
> Темы: S6, S7

1. Создайте модель `Category` — категория задачи.
   - `php artisan make:model Category -m`
2. Определение структуры таблицы **category** в миграции:
   - Добавьте поля:
     - `id` — первичный ключ;
     - `name` — название категории;
     - `description` — описание категории;
     - `created_at` — дата создания категории;
     - `updated_at` — дата обновления категории.
```php
public function up(): void
    {
        Schema::create('categories', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->text('description')->nullable();
            $table->timestamps();
        });
    }
```    
3. Создайте модель `Task` — задача.
4. Определение структуры таблицы **task** в миграции:
   - Добавьте поля:
     - `id` — первичный ключ;
     - `title` — название задачи;
     - `description` — описание задачи;
     - `created_at` — дата создания задачи;
     - `updated_at` — дата обновления задачи.
```php
    public function up(): void
    {
        Schema::create('tasks', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->text('description')->nullable();
            $table->timestamps();
        });
    }
```
5. Запустите миграцию для создания таблицы в базе данных:
   ```bash
   php artisan migrate
   ```
6. Создайте модель `Tag` — тег задачи.
7. Определение структуры таблицы **tag** в миграции:
   - Добавьте поля:
     - `id` — первичный ключ;
     - `name` — название тега;
     - `created_at` — дата создания тега;Й
     - `updated_at` — дата обновления тега.
```php
public function up(): void
{
    Schema::create('tags', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->timestamps();
    });
}
```
8. Добавьте поле `$fillable` в модели `Task`, `Category` и `Tag` для массового заполнения данных.
```php
# Category
protected $fillable = [
    'name',
    'description',
];

# Tag
protected $fillable = [
    'name',
];

# Task
protected $fillable = [
    'title',
    'description',
];
```

### №3. Связь между таблицами

> [!TIP]
> Темы: S8

1. Создайте миграцию для добавления поля `category_id` в таблицу **task**.
   - `php artisan make:migration add_category_id_to_tasks_table --table=tasks`
   - Определите структуру поля `category_id` и добавьте внешний ключ для связи с таблицей **category**.
```php
Schema::table('tasks', function (Blueprint $table) {
            $table->foreign('category_id')
                  ->references('id')->on('categories')
                  ->onDelete('set null');
 });
```

2. Создайте промежуточную таблицу для связи многие ко многим между задачами и тегами:
   - `php artisan make:migration create_task_tag_table`
3. Определение соответствующей структуры таблицы в миграции.
   - Данная таблица должна связывать задачи и теги по их идентификаторам.
   - **Например**: `task_id` и `tag_id`: `10` задача связана с `5` тегом.
```php
Schema::create('task_tag', function (Blueprint $table) {
    $table->foreign('task_id')->references('id')->on('tasks')->onDelete('cascade');
    $table->foreign('tag_id')->references('id')->on('tags')->onDelete('cascade');
});
```

4. Запустите миграцию для создания таблицы в базе данных.

### №4. Связи между моделями

> [!TIP]
> Темы: S8

1. Добавьте отношения в модель `Category` (Категория может иметь много задач)
   - Откройте модель `Category` и добавьте метод:
     ```php
     public function tasks()
     {
         return $this->hasMany(Task::class);
     }
     ```
2. Добавьте отношения в модель `Task`
   - Задача прикреплена к одной категории.
```php
public function category()
{
    return $this->belongsTo(Category::class);
}
```

   - Задача может иметь много тегов.
```php
public function tags()
{
    return $this->belongsToMany(Tag::class, 'task_tag');
}
```

3. Добавьте отношения в модель `Tag` (Тег может быть прикреплен к многим задачам)
```php
public function tasks()
{
    return $this->belongsToMany(Task::class, 'task_tag'); // Отношение "Многие ко многим"
}
```

4. Добавьте соотвтествующие поля в `$fillable` моделей.
```php
protected $fillable = [
        'title',
        'description',
        'category_id',
];
```

### №5. Создание фабрик и сидов

> [!TIP]
> Темы: S7, S8

1. Создайте фабрику для модели `Category`:
   - `php artisan make:factory CategoryFactory --model=Category`
   - Определите структуру данных для генерации категорий.
```php
public function definition()
{
    return [
        'name' => $this->faker->word, // Сгенерированное название категории
        'description' => $this->faker->sentence, // Описание категории
    ];
}
```
2. Создайте фабрику для модели `Task`.
```php
public function definition()
{
    return [
        'title' => $this->faker->sentence,
        'description' => $this->faker->paragraph,
        'category_id' => Category::factory(),
    ];
}
```
4. Создайте фабрику для модели `Tag`.
```php
public function definition()
{
    return [
        'name' => $this->faker->word,
    ];
}
```
6. Создайте сиды (`seeders`) для заполнения таблиц начальными данными для моделей: `Category`, `Task`, `Tag`.

      ```bash
      php artisan make:seeder CategorySeeder
      php artisan make:seeder TaskSeeder
      php artisan make:seeder TagSeeder
      ```


7. Обновите файл `DatabaseSeeder` для запуска сидов и запустите их:
   ```bash
   php artisan db:seed
   ```

### №6. Работа с контроллерами и представлениями

> [!TIP]
> Темы: S4, S5, S7, S8

1. Откройте контроллер `TaskController` (`app/Http/Controllers/TaskController.php`).
2. Обновите метод `index` для получения списка задач из базы данных. 
```php
public function index()
{
    $tasks = Task::with(['category', 'tags'])->get();

    return view('tasks.index', compact('tasks'));
}
```
3. Обновите метод `show` для отображения отдельной задачи. 
```php
public function show($id)
{
    $task = Task::with(['category', 'tags'])->findOrFail($id);

    return view('tasks.show', compact('task'));
}
```
4. В методах `index` и `show` используйте метод `with` (**Eager Loading**) для загрузки связанных моделей.
4. Обновите соответствующие представления для отображения списка задач и отдельной задачи.
5. Обновите метод `create` для отображения формы создания задачи и метод `store` для сохранения новой задачи в базе данных. 
   - **Примечание**: Поскольку вы ещё не изучали работу с формами, используйте объект `Request` для получения данных. **Например**:
     ```php
     $request->input('title');
     // или
     $request->all();
     ```
6. Обновите метод `edit` для отображения формы редактирования задачи и метод `update` для сохранения изменений в базе данных.
```php
    public function edit($id)
    {
        $task = Task::with(['category', 'tags'])->findOrFail($id);
        $categories = Category::all();
        $tags = Tag::all();

        return view('tasks.edit', compact('task', 'categories', 'tags'));
    }

    public function update(Request $request, $id)
    {
        // Валидация данных
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'description' => 'nullable|string',
            'category_id' => 'required|exists:categories,id',
            'tags' => 'nullable|array',
            'tags.*' => 'exists:tags,id',
        ]);

        $task = Task::findOrFail($id);
        $task->update([
            'title' => $validated['title'],
            'description' => $validated['description'],
            'category_id' => $validated['category_id'],
        ]);

        // Обновляем теги
        $task->tags()->sync($validated['tags'] ?? []);

        return redirect()->route('tasks.show', $task->id);
    }
```
8. Обновите метод `destroy` для удаления задачи из базы данных.4
```php
    public function destroy($id)
    {
        $task = Task::findOrFail($id);
        $task->delete();

        return redirect()->route('tasks.index');
    }
```
  
## Контрольные вопросы

1. Что такое миграции  и для чего они используются?

   Миграции — это способ управления схемой базы данных в системе контроля версий. Они позволяют изменять структуру базы данных   (создавать, изменять, удалять таблицы и столбцы) и отслеживать эти изменения. Миграции упрощают процесс развертывания и обновления базы данных, обеспечивая согласованность структуры базы данных на всех этапах разработки и в разных средах.

2. Что такое фабрики и сиды, и как они упрощают процесс разработки и тестирования?

   Фабрики — это инструменты для создания фиктивных данных для моделей. Они позволяют легко генерировать тестовые данные для разработки и тестирования. Сиды (seeders) — это скрипты, которые заполняют базу данных начальными данными. Они используются для создания начальных данных, необходимых для работы приложения. Фабрики и сиды упрощают процесс разработки и тестирования, позволяя быстро создавать и заполнять базу данных тестовыми данными.

3. Что такое ORM? В чем различия между паттернами `DataMapper` и `ActiveRecord`?

   DataMapper и ActiveRecord? ORM (Object-Relational Mapping) — это технология, которая позволяет взаимодействовать с базой данных, используя объектно-ориентированный подход. ORM автоматически сопоставляет объекты в коде с записями в базе данных.

   1. `DataMapper` — это паттерн, при котором объекты и их данные хранятся отдельно, а специальный объект (DataMapper) отвечает за их синхронизацию с базой данных.
   2. `ActiveRecord` — это паттерн, при котором объекты содержат как данные, так и методы для работы с базой данных. Каждый объект напрямую связан с записью в базе данных и может выполнять операции CRUD (создание, чтение, обновление, удаление).

4. В чем преимущества использования ORM по сравнению с прямыми SQL-запросами?

   1. Упрощение кода: ORM позволяет писать меньше кода для выполнения операций с базой данных.
   2. Безопасность: ORM автоматически экранирует данные, что помогает предотвратить SQL-инъекции.
   3. Поддержка разных СУБД: ORM абстрагирует работу с базой данных, что позволяет легко менять СУБД без изменения кода.
   4. Объектно-ориентированный подход: ORM позволяет работать с базой данных, используя объекты и их методы, что делает код более читаемым и поддерживаемым.

5. Что такое транзакции и зачем они нужны при работе с базами данных?

   Транзакции — это последовательность операций с базой данных, которые выполняются как единое целое. Транзакции обеспечивают атомарность, согласованность, изоляцию и долговечность (ACID). Они гарантируют, что все операции внутри транзакции будут выполнены успешно или ни одна из них не будет выполнена. Транзакции необходимы для обеспечения целостности данных и предотвращения неконсистентных состояний базы данных в случае сбоев или ошибок.
