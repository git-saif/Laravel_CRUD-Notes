
Laravel -এ API দিয়ে CRUD Operation এর জন্য মূলত ৪টি Step Follow করতে হয়। সেগুলো হলোঃ

1. API Routes        =>  `routes\api.php`
2. Model               => `app\Models\Crud.php`
3. Migration          => `database\migrations\2025_04_14_164123_create_cruds_table.php`
4. Controller         => `app\Http\Controllers\CrudController.php`
5. View                  => এটি সর্বশেষে আলোচনা করা হবে।
   
নিচে এগুলোর বিস্তারিত বর্ণনা দেয়া হলোঃ
#### **Workflow**:
```scss
		(Route → Controller → Model → View)
```
____
## Step-1: (Web Route)

##### `routes/web.php`:
```php
Route::get('/dashboard', function () {
    return view('components.dashboard');
})->name('dashboard');

/*
|--------------------------------------------------------------------------
|                         API Resource Routes.
|--------------------------------------------------------------------------
*/

Route::group(['prefix' => 'dashboard', 'as' => 'dashboard.'], function () {

    Route::resources([
        'crud-1' => Crud1Controller::class,
    ]);
});
```
_____
## Step-2: (Model)

- Make model with migration file:
```bash
	php artisan make:model Crud-1 -m
```

##### `app\Models\Crud1.php`:
```php
class Crud1 extends Model
{
    use HasFactory;

    protected $guarded = [];
}
```
____
## Step-3: (Migration)

##### `database\migrations\2025_05_05_100447_create_crud1s_table.php`:
```php
	Schema::create('crud1s', function (Blueprint $table) {
		$table->id();
		$table->string('topic_name');
		$table->string('title');
		$table->string('description');
		$table->enum('status', ['active', 'inactive'])->default('active');
		$table->timestamps();
	});
```
_____
## Step-4: (Controller)

- Make API Resource Controller :
```bash
	php artisan make:controller Api/Crud1Controller --api
```

> এটি app/Http/Controllers/Api/Crud1Controller.php ফাইল তৈরি করবে।
> api flag দিলে controller এ শুধু API method (যেগুলো view return করে না) থাকবে।

or
```bash
	php artisan make:controller Api/Crud1Controller --api --model=Crud1
```

> এইটা দিলে controller এ CRUD1 model এর use statement চলে আসবে। যেমনঃ `use App\Models\Crud1;`
> এবং সরাসরি model access করতে পারবে, আলাদা করে import না করেও।

###### 🧾 সংক্ষেপে পার্থক্য:

| Command                                                               | কাজ                                | Extra                                            |
| --------------------------------------------------------------------- | ---------------------------------- | ------------------------------------------------ |
| `php artisan make:controller Api/Crud1Controller --api`               | শুধুমাত্র API method সহ controller | ৫টি method (index, store, show, update, destroy) |
| `php artisan make:controller Api/Crud1Controller --api --model=Crud1` | API controller + model import সহ   | model auto-import হবে                            |
______

আরেকটি পার্থক্যঃ

|বিষয়|`--api`|`--api --model=Crud1`|
|---|---|---|
|Model Binding|❌ না|✅ হ্যাঁ|
|Model instance auto injection|❌ না|✅ হ্যাঁ|
|findOrFail() দরকার|✅ হ্যাঁ|❌ না|
|404 auto রিটার্ন|❌ না|✅ হ্যাঁ|
|কোডের পরিমাণ|বেশি|কম ও পরিষ্কার|
##### `app\Http\Controllers\Api\Crud1Controller.php`:
```php
class Crud1Controller extends Controller
{
    public function index()
    {
        $crud1 = Crud1::orderBy('id', 'asc')->paginate(5);
        return response()->json($crud1);
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'topic_name' => 'required',
            'title' => 'required',
            'description' => 'required',
            'status' => 'required',
        ]);

        $crud = Crud1::create($validated);

        return response()->json([
            'message' => 'Data successfully stored.',
            'data' => $crud
        ], 201);
    }

    public function show($id)
    {
        $crud = Crud1::findOrFail($id);
        return response()->json($crud);
    }

    public function update(Request $request, $id)
    {
        $crud = Crud1::findOrFail($id);

        $validated = $request->validate([
            'topic_name' => 'required|string|max:255',
            'title' => 'required|string|max:255',
            'description' => 'required|string',
            'status' => 'required|in:active,inactive',
        ]);

        $crud->update($validated);

        return response()->json([
            'message' => 'Data successfully updated.',
            'data' => $crud
        ]);
    }

    public function destroy($id)
    {
        $crud = Crud1::findOrFail($id);
        $crud->delete();

        return response()->json(['message' => 'Data successfully deleted.']);
    }
}
```
______


এখন এখানে api যেহেতু only json data response করে, তাই এর frontend view প্রদর্শনের জন্য আলাদাভাবে web route, Controller & View create করতে হবে।
1. Views                =>  `index.blade.php` , `create.blade.php` , `edit.blade.php`


```scss
		Upcomming 
```

