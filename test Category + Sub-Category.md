দারুন! 🎯 নিচে তোমার চাওয়া মতো **Category (Crud7)** ও **Subcategory (Crud8)** এর জন্য একটি সম্পূর্ণ Laravel CRUD setup দেওয়া হলো —  
যেখানে **auto slug generation**, **shared request validation**, এবং **Resource Route structure** পুরোপুরি অনুসরণ করা হয়েছে।

---

## 🧩 Step 1: Routes

`routes/web.php`

```php
Route::group(['prefix' => 'dashboard', 'as' => 'dashboard.'], function () {
    Route::resources([
        'crud-7' => App\Http\Controllers\Crud7Controller::class, // Category
        'crud-8' => App\Http\Controllers\Crud8Controller::class, // Subcategory
    ]);
});
```

---

## 🧱 Step 2: Migrations

### Category Table Migration

`database/migrations/2025_10_11_000001_create_crud7s_table.php`

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void {
        Schema::create('crud7s', function (Blueprint $table) {
            $table->id();
            $table->string('name')->unique();
            $table->string('slug')->unique();
            $table->integer('serial_no')->unique();
            $table->enum('status', ['active', 'inactive'])->default('active');
            $table->timestamps();
        });
    }

    public function down(): void {
        Schema::dropIfExists('crud7s');
    }
};
```

---

### Subcategory Table Migration

`database/migrations/2025_10_11_000002_create_crud8s_table.php`

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void {
        Schema::create('crud8s', function (Blueprint $table) {
            $table->id();
            $table->foreignId('crud7_id')->constrained('crud7s')->onUpdate('cascade')->onDelete('cascade');
            $table->string('name')->unique();
            $table->string('slug')->unique();
            $table->integer('serial_no')->unique();
            $table->enum('status', ['active', 'inactive'])->default('active');
            $table->timestamps();
        });
    }

    public function down(): void {
        Schema::dropIfExists('crud8s');
    }
};
```

---

## 🧠 Step 3: Models

### Category Model

`app/Models/Crud7.php`

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Str;

class Crud7 extends Model
{
    protected $fillable = ['name', 'slug', 'serial_no', 'status'];

    // Auto slug generation
    public static function boot()
    {
        parent::boot();

        static::creating(function ($category) {
            $category->slug = Str::slug($category->name);
        });

        static::updating(function ($category) {
            $category->slug = Str::slug($category->name);
        });
    }

    public function subcategories()
    {
        return $this->hasMany(Crud8::class, 'crud7_id');
    }
}
```

---

### Subcategory Model

`app/Models/Crud8.php`

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Str;

class Crud8 extends Model
{
    protected $fillable = ['crud7_id', 'name', 'slug', 'serial_no', 'status'];

    public static function boot()
    {
        parent::boot();

        static::creating(function ($subcategory) {
            $subcategory->slug = Str::slug($subcategory->name);
        });

        static::updating(function ($subcategory) {
            $subcategory->slug = Str::slug($subcategory->name);
        });
    }

    public function category()
    {
        return $this->belongsTo(Crud7::class, 'crud7_id');
    }
}
```

---

## 🧾 Step 4: Request Class (Shared Logic)

`app/Http/Requests/CrudCategoryRequest.php`

```php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class CrudCategoryRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        if ($this->isMethod('post')) {
            return $this->storeRules();
        } elseif ($this->isMethod('put') || $this->isMethod('patch')) {
            return $this->updateRules();
        }
        return [];
    }

    protected function storeRules(): array
    {
        return [
            'name'       => 'required|string|max:255|unique:crud7s,name',
            'serial_no'  => 'required|integer|unique:crud7s,serial_no',
            'status'     => 'required|in:active,inactive',
        ];
    }

    protected function updateRules(): array
    {
        $crud7 = $this->route('crud7') ?? $this->route('crud_7') ?? $this->route('crud-7');
        $id = $crud7?->id ?? $crud7;

        return [
            'name'       => ['required', 'string', 'max:255', Rule::unique('crud7s', 'name')->ignore($id)],
            'serial_no'  => ['required', 'integer', Rule::unique('crud7s', 'serial_no')->ignore($id)],
            'status'     => 'required|in:active,inactive',
        ];
    }

    public function messages(): array
    {
        return [
            'name.required'      => 'Category name is required.',
            'name.unique'        => 'This category name already exists.',
            'serial_no.required' => 'Serial number is required.',
            'serial_no.unique'   => 'This serial number already exists.',
            'status.required'    => 'Please select a status.',
        ];
    }
}
```

> Subcategory এর জন্য নিচেরটা প্রায় একই, শুধু `crud8s` টেবিলের নাম ও `crud7_id` যোগ হবে।

---

`app/Http/Requests/CrudSubCategoryRequest.php`

```php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class CrudSubCategoryRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        if ($this->isMethod('post')) {
            return $this->storeRules();
        } elseif ($this->isMethod('put') || $this->isMethod('patch')) {
            return $this->updateRules();
        }
        return [];
    }

    protected function storeRules(): array
    {
        return [
            'crud7_id'   => 'required|exists:crud7s,id',
            'name'       => 'required|string|max:255|unique:crud8s,name',
            'serial_no'  => 'required|integer|unique:crud8s,serial_no',
            'status'     => 'required|in:active,inactive',
        ];
    }

    protected function updateRules(): array
    {
        $crud8 = $this->route('crud8') ?? $this->route('crud_8') ?? $this->route('crud-8');
        $id = $crud8?->id ?? $crud8;

        return [
            'crud7_id'   => 'required|exists:crud7s,id',
            'name'       => ['required', 'string', 'max:255', Rule::unique('crud8s', 'name')->ignore($id)],
            'serial_no'  => ['required', 'integer', Rule::unique('crud8s', 'serial_no')->ignore($id)],
            'status'     => 'required|in:active,inactive',
        ];
    }

    public function messages(): array
    {
        return [
            'crud7_id.required'  => 'Please select a category.',
            'name.required'      => 'Subcategory name is required.',
            'name.unique'        => 'This subcategory already exists.',
            'serial_no.required' => 'Serial number is required.',
            'serial_no.unique'   => 'This serial number already exists.',
        ];
    }
}
```

---

## ⚙️ Step 5: Controllers

### Category Controller

`app/Http/Controllers/Crud7Controller.php`

```php
namespace App\Http\Controllers;

use App\Http\Requests\CrudCategoryRequest;
use App\Models\Crud7;

class Crud7Controller extends Controller
{
    public function index()
    {
        $crud7 = Crud7::orderBy('serial_no', 'asc')->paginate(10);
        return view('components.CRUD-7.index', compact('crud7'));
    }

    public function create()
    {
        return view('components.CRUD-7.create');
    }

    public function store(CrudCategoryRequest $request)
    {
        Crud7::create($request->validated());
        return redirect()->route('dashboard.crud-7.index')->with('success', 'Category created successfully.');
    }

    public function edit(Crud7 $crud7)
    {
        return view('components.CRUD-7.edit', compact('crud7'));
    }

    public function update(CrudCategoryRequest $request, Crud7 $crud7)
    {
        $crud7->update($request->validated());
        return redirect()->route('dashboard.crud-7.index')->with('success', 'Category updated successfully.');
    }

    public function destroy(Crud7 $crud7)
    {
        $crud7->delete();
        return redirect()->back()->with('success', 'Category deleted successfully.');
    }
}
```

---

### Subcategory Controller

`app/Http/Controllers/Crud8Controller.php`

```php
namespace App\Http\Controllers;

use App\Http\Requests\CrudSubCategoryRequest;
use App\Models\Crud8;
use App\Models\Crud7;

class Crud8Controller extends Controller
{
    public function index()
    {
        $crud8 = Crud8::with('category')->orderBy('serial_no', 'asc')->paginate(10);
        return view('components.CRUD-8.index', compact('crud8'));
    }

    public function create()
    {
        $categories = Crud7::orderBy('name')->get();
        return view('components.CRUD-8.create', compact('categories'));
    }

    public function store(CrudSubCategoryRequest $request)
    {
        Crud8::create($request->validated());
        return redirect()->route('dashboard.crud-8.index')->with('success', 'Subcategory created successfully.');
    }

    public function edit(Crud8 $crud8)
    {
        $categories = Crud7::orderBy('name')->get();
        return view('components.CRUD-8.edit', compact('crud8', 'categories'));
    }

    public function update(CrudSubCategoryRequest $request, Crud8 $crud8)
    {
        $crud8->update($request->validated());
        return redirect()->route('dashboard.crud-8.index')->with('success', 'Subcategory updated successfully.');
    }

    public function destroy(Crud8 $crud8)
    {
        $crud8->delete();
        return redirect()->back()->with('success', 'Subcategory deleted successfully.');
    }
}
```

---

## 🪶 Step 6: Views (Example)

তুমি চাইলে Blade ফাইলগুলো Bootstrap বা Ace Admin template অনুযায়ী তৈরি করতে পারো:

```
resources/views/components/CRUD-7/index.blade.php
resources/views/components/CRUD-7/create.blade.php
resources/views/components/CRUD-7/edit.blade.php

resources/views/components/CRUD-8/index.blade.php
resources/views/components/CRUD-8/create.blade.php
resources/views/components/CRUD-8/edit.blade.php
```

---

চাওলে আমি পরবর্তী ধাপে Bootstrap ভিত্তিক Blade ফর্মগুলো (`create`, `edit`, `index`) auto তৈরি করে দিতে পারি।  
👉 চাও কি আমি সেই Blade গুলোও তৈরি করে দিই (Category + Subcategory)?