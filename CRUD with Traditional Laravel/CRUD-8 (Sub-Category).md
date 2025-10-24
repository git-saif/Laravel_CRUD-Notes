Laravel -এর এই CRUD Operation এ আমরা **Product/Post** এর **Sub Category** Create করা শিখবো। এর জন্য 6টি Step Follow করতে হয়। সেগুলো হলোঃ

1. Routes               =>  `routes\web.php`
2. Model                => `app\Models\Crud8.php`
3. Migration          => `database\migrations\2025_04_14_164123_create_crud8s_table.php`
4. Controller         => `app\Http\Controllers\Crud8Controller.php`
5. Request form   =>  `app\Http\Requests\Crud8Request.php`
6. Views                =>  `index.blade.php` , `create.blade.php` , `edit.blade.php`
   
নিচে এগুলোর বিস্তারিত বর্ণনা দেয়া হলোঃ
#### **Workflow**:
```scss
						(Route → Middleware (if any) → Controller → FormRequest → Model → View)
```
____
## Step-1: (Web Route)

##### `routes/web.php`:
```php
Route::group(['prefix' => 'dashboard', 'as' => 'dashboard.'], function () {
    Route::resources([
        'crud-8' => Crud8Controller::class,  // Sub-category
    ]);
});
```
_____
## Step-2: (Model)

##### `app\Models\Crud8.php`:
```php
class Crud8 extends Model
{
    use HasFactory;

    protected $fillable = [
        'crud7_id',
        'name',
        // 'slug',
        'serial_no',
        'status',
    ];

    // Auto Slug Generation 
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

    // relation -> parent Category (Crud7)
    public function category()
    {
        return $this->belongsTo(Crud7::class, 'crud7_id');
    }

    // relation -> parent Subcategory (Crud8)
    public function subsubcategories()
    {
        return $this->hasMany(Crud9::class, 'crud8_id');
    }
}
```
____
## Step-3: (Migration)

##### `database\migrations\2025_05_05_100447_create_crud8s_table.php`:
```php
	Schema::create('crud8s', function (Blueprint $table) {
		$table->id();
		$table->foreignId('crud7_id')->constrained('crud7s')->onUpdate('cascade')->onDelete('cascade');
		$table->string('name')->unique();
		$table->string('slug')->unique();
		$table->integer('serial_no')->unique();
		$table->enum('status', ['active', 'inactive'])->default('active');
		$table->timestamps();
	});
```
_____
## Step-4: (Controller)

##### `app\Http\Controllers\Crud8Controller.php`:
```php
class Crud8Controller extends Controller
{
    public function index()
    {
        $crud8 = Crud8::with('category')->orderBy('serial_no', 'asc')->paginate(3);
        return view('components.CRUD-8.index', compact('crud8'));
    }

    public function create()
    {
        $categories = Crud7::where('status', 'active')->orderBy('serial_no')->get();
        return view('components.CRUD-8.create', compact('categories'));
    }

    public function store(Crud8Request $request)
    {
        try {
            Crud8::create($request->validated());

            return redirect()
                ->route('dashboard.crud-8.index')
                ->with('success', 'Category created successfully!');
        } catch (\Throwable $th) {
            return redirect()->back()->with('error', 'Something went wrong: ' . $th->getMessage());
        }
    }

    public function edit(string $id)
    {
        try {
            $crud8 = Crud8::findOrFail($id);
            $categories = Crud7::where('status', 'active')->orderBy('serial_no')->get();
            return view('components.CRUD-8.edit', compact('crud8', 'categories'));
        } catch (\Throwable $th) {
            return redirect()->back()->with('error', 'Something went wrong: ' . $th->getMessage());
        }
    }

    public function update(Crud8Request $request, string $id)
    {
        try {
            $crud8 = Crud8::findOrFail($id);
            $crud8->update($request->validated());
            return redirect()->route('dashboard.crud-8.index')->with('success', 'SubCategory updated successfully.');
        } catch (\Throwable $th) {
            return redirect()->back()->with('error', 'Something went wrong: ' . $th->getMessage());
        }
    }

    public function destroy(string $id)
    {
        try {
            $crud8 = Crud8::findOrFail($id);
            $crud8->delete();
            return redirect()->route('dashboard.crud-8.index')->with('success', 'SubCategory deleted successfully.');
        } catch (\Throwable $th) {
            return redirect()->back()->with('error', 'Something went wrong: ' . $th->getMessage());
        }
    }
}
```
______
## Step-5: (Form Request)

##### **`app\Http\Requests\Crud8Request.php:`**
```php
class Crud8Request extends FormRequest
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
------
## Step-6: (View Create)

##### `index.blade.php`:
```html
@extends('layouts.app')
@section('content')
@section('title', 'CRUD8 - Sub-Category List')
<div class="main-content">
  <div class="widget-header widget-header-flat" style="background-color: #618f8f;">
	<h4 class="widget-title" style="color: #fff;">Sub Category List</h4>
	<span class="widget-toolbar">
	  <a href="{{ route('dashboard.crud-8.create') }}" style="color: #fff;">
		<i class="ace-icon fa fa-plus"></i> Create New Sub-Category
	  </a>
	</span>
  </div><!-- /.Table Header -->
  <div>
	<table id="dynamic-table" class="table table-striped table-bordered table-hover">
	  <thead>
		<tr>
		  <th style="font-weight: bold;">Sl No</th>
		  <th>Category (From Crud7)</th>
		  <th>Category Serial</th>
		  <th>Sub-Category Name</th>
		  <th>Sub-Category Slug</th>
		  <th>Serial No</th>
		  <th>Status</th>
		  <th>Action</th>
		</tr>
	  </thead>

	  <tbody>
		@php $sl = $crud8->firstItem() ?? 1; @endphp
		@forelse ($crud8 as $item)
		<tr>
		  <td style="font-weight: bold;">{{ $sl++ }}.</td>

		  {{-- Foreign key data from Crud7 --}}
		  <td>{{ $item->category->name ?? 'N/A' }}</td>
		  <td>{{ $item->category->serial_no ?? 'N/A' }}</td>
		  <td>{{ $item->name }}</td>
		  <td>{{ $item->slug }}</td>
		  <td>{{ $item->serial_no }}</td>
		  <td>
			@if ($item->status === 'active')
			<span class="badge badge-success">Active</span>
			@else
			<span class="badge badge-danger">Inactive</span>
			@endif
		  </td>
		  <td>
			<div class="hidden-sm hidden-xs action-buttons">
			  <a class="blue" href="#">
				<i class="ace-icon fa fa-eye bigger-130"></i>
			  </a>

			  <a class="green" href="{{ route('dashboard.crud-8.edit', $item->id) }}">
				<i class="ace-icon fa fa-pencil bigger-130"></i>
			  </a>

			  <form action="{{ route('dashboard.crud-8.destroy', $item->id) }}" 
				    method="POST" style="display:inline;" onsubmit="return confirm('Are you sure ?');">
				@csrf
				@method('DELETE')
				<button type="submit" class="red" style="border: none; background: none;">
				  <i class="ace-icon fa fa-trash-o bigger-130"></i>
				</button>
			  </form>
			</div>
		  </td>
		</tr>
		@empty
		<tr>
		  <td colspan="8" class="text-center text-danger">No data found.</td>
		</tr>
		@endforelse
	  </tbody>
	</table>
	<div class="text-center">
	  {{ $crud8->links('pagination::bootstrap-4') }}
	</div>
  </div>
</div>
@endsection
```
-----
##### `create.blade.php`:
```html
@extends('layouts.app')
@section('content')
@section('title', 'CRUD7 - Category Create')
<!-- Table is here -->
<div class="main-content">
	<div class="widget-header widget-header-flat " style="background-color: #618f8f;">
	  <h4 class="widget-title" style="color: #fff;">Create Sub-Category</h4>
	  <span class="widget-toolbar">
		<a href="{{ route('dashboard.crud-8.index') }}" style="color: #fff;">
		  <i class="ace-icon fa fa-plus"></i> Go To Index
		</a>
	  </span>
	</div>
	<!-- Main Create Form Start -->
	<form action="{{ route('dashboard.crud-8.store') }}" method="POST">
	  @csrf

	  {{-- Parent Category (From Crud7) --}}
	  <div class="form-group">
		<label for="crud7_id">Select Category</label>
		<select name="crud7_id" id="crud7_id" 
			    class="form-control @error('crud7_id') is-invalid @enderror" required>
		  <option value="">-- Choose Category --</option>
		  @foreach($categories as $category)
		  <option value="{{ $category->id }}" {{ old('crud7_id') == $category->id ? 'selected' : '' }}>
			{{ $category->name }}
		  </option>
		  @endforeach
		</select>
		@error('crud7_id')
		<small class="text-danger">{{ $message }}</small>
		@enderror
	  </div>

	  {{-- Name --}}
	  <div class="form-group">
		<label for="name">Subcategory Name</label>
		<input type="text" id="name" name="name" 
			   class="form-control @error('name') is-invalid @enderror" value="{{ old('name') }}"
			   placeholder="Enter subcategory name" required>
		@error('name')
		<small class="text-danger">{{ $message }}</small>
		@enderror
	  </div>

	  {{-- Slug (auto-generated) --}}
	  <div class="form-group">
		<label for="slug">Slug (Auto Generated)</label>
		<input type="text" name="slug" id="slug" class="form-control" value="{{ old('slug') }}"
			   placeholder="Auto generated slug" readonly>
	  </div>

	  {{-- Serial Number --}}
	  <div class="form-group">
		<label for="serial_no">Serial No</label>
		<input type="number" name="serial_no" 
			   class="form-control @error('serial_no') is-invalid @enderror" 
			   value="{{ old('serial_no') }}" placeholder="Enter serial number" required>
		@error('serial_no')
		<small class="text-danger">{{ $message }}</small>
		@enderror
	  </div>

	  {{-- Status --}}
	  <div class="form-group">
		<label for="status">Status</label><br>
		<label>
		  <input type="radio" name="status" value="active" 
			     {{ old('status', 'active') == 'active' ? 'checked' : '' }}>
		  Active
		</label>
		&nbsp;&nbsp;
		<label>
		  <input type="radio" name="status" value="inactive"
			     {{ old('status') == 'inactive' ? 'checked' : '' }}>
		  Inactive
		</label>
		@error('status')
		<br><small class="text-danger">{{ $message }}</small>
		@enderror
	  </div>

	  {{-- Action Buttons --}}
	  <div class="form-actions center mt-3">
		<button type="submit" class="btn btn-sm btn-success">
		  Save
		  <i class="ace-icon fa fa-save bigger-110"></i>
		</button>

		<a href="{{ route('dashboard.crud-8.index') }}" class="btn btn-sm btn-warning">
		  <i class="ace-icon fa fa-arrow-left bigger-110"></i> Back
		</a>
	  </div>
	</form>
	<!-- Form End -->
  </div>
</div><!-- /.main-content -->
@endsection
@push('scripts')
  
  {{-- Slug Auto Generator Script --}}
  <script>
    document.addEventListener('DOMContentLoaded', function() {
      const nameInput = document.getElementById('name');
      const slugInput = document.getElementById('slug');

      nameInput.addEventListener('keyup', function() {
        let slug = nameInput.value
          .toLowerCase()
          .replace(/[^a-z0-9\s-]/g, '') // remove special chars
          .replace(/\s+/g, '-') // replace spaces with -
          .replace(/-+/g, '-'); // collapse multiple -
        slugInput.value = slug;
      });
    });

    nameInput.addEventListener('input', function() {
    // same slug generation logic
    });
  </script>
@endpush
```
----
##### `edit.blade.php`:
```html
@extends('layouts.app')
@section('content')
@section('title', 'CRUD8 - Sub-Category Edit')
<!-- Edit form -->
<div class="main-content">
	<!-- Edit Form Start -->
	<form action="{{ route('dashboard.crud-8.update', $crud8->id) }}" method="POST" 
		  enctype="multipart/form-data">
	  @csrf
	  @method('PUT')

	  {{-- Parent Category --}}
	  <div class="form-group">
		<label for="crud7_id">Parent Category</label>
		<select name="crud7_id" id="crud7_id" 
				class="form-control @error('crud7_id') is-invalid @enderror" required>
		  <option value="">-- Select Category --</option>
		  @foreach($categories as $category)
		  <option value="{{ $category->id }}" 
				  {{ old('crud7_id', $crud8->crud7_id) == $category->id ? 'selected' : '' }}>
			{{ $category->name }}
		  </option>
		  @endforeach
		</select>
		@error('crud7_id')
		<small class="text-danger">{{ $message }}</small>
		@enderror
	  </div>

	  {{-- Name --}}
	  <div class="form-group">
		<label for="name">Subcategory Name</label>
		<input type="text" id="name" name="name" class="form-control @error('name') is-invalid @enderror" 
			   value="{{ old('name', $crud8->name) }}" placeholder="Enter subcategory name" required>
		@error('name')
		<small class="text-danger">{{ $message }}</small>
		@enderror
	  </div>

	  {{-- Slug (auto-generated) --}}
	  <div class="form-group">
		<label for="slug">Slug (Auto Generated)</label>
		<input type="text" id="slug" name="slug" class="form-control"
			   value="{{ old('slug', $crud8->slug) }}" readonly>
	  </div>

	  {{-- Serial Number --}}
	  <div class="form-group">
		<label for="serial_no">Serial No</label>
		<input type="number" name="serial_no" 
			   class="form-control @error('serial_no') is-invalid @enderror"
			   value="{{ old('serial_no', $crud8->serial_no) }}"
			   placeholder="Enter serial number" required>
		@error('serial_no')
		<small class="text-danger">{{ $message }}</small>
		@enderror
	  </div>

	  {{-- Status --}}
	  <div class="form-group">
		<label for="status">Status</label><br>
		<label>
		  <input type="radio" name="status" value="active" 
				 {{ old('status', $crud8->status) == 'active' ? 'checked' : '' }}>
		  Active
		</label>
		&nbsp;&nbsp;
		<label>
		  <input type="radio" name="status" value="inactive" 
				 {{ old('status', $crud8->status) == 'inactive' ? 'checked' : '' }}>
		  Inactive
		</label>
		@error('status')
		<br><small class="text-danger">{{ $message }}</small>
		@enderror
	  </div>
	  <div class="form-actions center">
		<button type="submit" class="btn btn-sm btn-success">
		  Update
		  <i class="ace-icon fa fa-check icon-on-right bigger-110"></i>
		</button>

		<a href="{{ route('dashboard.crud-8.index') }}" class="btn btn-sm btn-warning">
		  <i class="ace-icon fa fa-arrow-left bigger-110"></i> Back
		</a>
	  </div>
	</form>
	<!-- Edit Form End -->
</div><!-- /.main-content -->
@endsection
@push('scripts')
{{-- Slug Auto Generator Script --}}
<script>
  document.addEventListener('DOMContentLoaded', function() {
    const nameInput = document.getElementById('name');
    const slugInput = document.getElementById('slug');

    nameInput.addEventListener('keyup', function() {
      let slug = nameInput.value
        .toLowerCase()
        .replace(/[^a-z0-9\s-]/g, '') // remove special chars
        .replace(/\s+/g, '-') // replace spaces with -
        .replace(/-+/g, '-'); // collapse multiple -
      slugInput.value = slug;
    });
  });

  nameInput.addEventListener('input', function() {
    // same slug generation logic
  });
</script>
@endpush
```
_____
