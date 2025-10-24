Laravel -এর এই CRUD Operation এ আমরা **Product/Post** এর **Sub-Sub Category** Create করা শিখবো। এর জন্য ৬টি Step Follow করতে হয়। সেগুলো হলোঃ

1. Routes               =>  `routes\web.php`
2. Model                => `app\Models\Crud9.php`
3. Migration          => `database\migrations\2025_04_14_164123_create_crud9s_table.php`
4. Controller         => `app\Http\Controllers\Crud9Controller.php`
5. Request form   =>  `app\Http\Requests\Crud9Request.php`
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
    
        'crud-9' => Crud9Controller::class,  // Sub-Sub-Category
    ]);
    // Ajax route to get subcategories for a category [This route use only when we use ajax rendering system]
    Route::get('crud-9/subcategories/{category}', [Crud9Controller::class, 'getSubcategories'])
        ->name('crud-9.subcategories');
});
```
_____
## Step-2: (Model)

##### `app\Models\Crud9.php`:
```php
class Crud9 extends Model
{
    use HasFactory;

    protected $fillable = [
        'crud8_id',
        'name',
        // 'slug',
        'serial_no',
        'status',
    ];

    // Auto slug generation
    public static function boot()
    {
        parent::boot();

        static::creating(function ($model) {
            $model->slug = Str::slug($model->name);
        });

        static::updating(function ($model) {
            $model->slug = Str::slug($model->name);
        });
    }

    // relation -> parent Subcategory (Crud8)
    public function subcategory()
    {
        return $this->belongsTo(Crud8::class, 'crud8_id');
    }
}
```
____
## Step-3: (Migration)

##### `database\migrations\2025_05_05_100447_create_crud9s_table.php`:
```php
	Schema::create('crud9s', function (Blueprint $table) {
		$table->id();
		// parent points to subcategory (crud8s)
		$table->foreignId('crud8_id')->constrained('crud8s')->onUpdate('cascade')->onDelete('cascade');
		$table->string('name')->unique();
		$table->string('slug')->unique();
		$table->integer('serial_no')->unique();
		$table->enum('status', ['active', 'inactive'])->default('inactive');
		$table->timestamps();
	});
```
_____
## Step-4: (Controller)

#### **Using Server Side Rendering :**

##### **`app\Http\Requests\Crud9Request.php:`**
```php
class Crud9Controller extends Controller
{
    public function index()
    {
        // eager load subcategory and its category for display
        $crud9 = Crud9::with('subcategory.category')->orderBy('serial_no', 'asc')->paginate(6);
        return view('components.CRUD-9.index', compact('crud9'));
    }

    public function create()
    {
        // সব category আনবে
	    $categories = Crud7::orderBy('name')->get();
	
	    // যেটা selected category
	    $selectedCategory = request()->query('category');
	
	    // যদি category select করা থাকে → শুধু সেই category-র subcategory গুলো আনবে
	    $subcategories = $selectedCategory 
	        ? Crud8::where('crud7_id', $selectedCategory)->orderBy('name')->get()
	        : collect(); // খালি collection
	
	    return view('components.CRUD-9.create', compact('categories', 'subcategories', 'selectedCategory'));
    }

    public function store(Crud9Request $request)
    {
        try {
            Crud9::create($request->validated());

            return redirect()
                ->route('dashboard.crud-9.index')
                ->with('success', 'Data saved successfully.');
        } catch (\Exception $e) {
            return redirect()->back()->with('error', 'An error occurred while saving data.');
        }
    }

    public function edit(string $id)
    {
        try {
            $crud9 = Crud9::findOrFail($id);
            $categories = Crud7::orderBy('name')->get();

            // previously selected category & its subcategories
            $selectedCategory = $crud9->subcategory ? $crud9->subcategory->crud7_id : null;
            $subcategories = $selectedCategory
                ? Crud8::where('crud7_id', $selectedCategory)->orderBy('name')->get()
                : collect();

            return view('components.CRUD-9.edit', compact(
											            'crud9',
											            'categories', 
											            'subcategories', 
											            'selectedCategory'));
        } catch (\Exception $e) {
            return redirect()->back()->with('error', 'Data not found.');
        }
    }

    public function update(Crud9Request $request, string $id)
    {
        try {
            $crud9 = Crud9::findOrFail($id);
            $crud9->update($request->validated());

            return redirect()
                ->route('dashboard.crud-9.index')
                ->with('success', 'Updated successfully.');
        } catch (\Exception $e) {
            return redirect()->back()->with('error', 'An error occurred while updating data.');
        }
    }

    public function destroy(string $id)
    {
        try {
            $crud9 = Crud9::findOrFail($id);
            $crud9->delete();

            return redirect()
                ->route('dashboard.crud-9.index')
                ->with('success', 'Deleted.');
        } catch (\Exception $e) {
            return redirect()->back()->with('error', 'An error occurred while deleting data.');
        }
    } 
}
```
----
#### **Using Ajax Rendering :**

##### `app\Http\Controllers\Crud9Controller.php`:
```php
class Crud9Controller extends Controller
{
    public function index()
    {
        // eager load subcategory and its category for display
        $crud9 = Crud9::with('subcategory.category')->orderBy('serial_no', 'asc')->paginate(10);
        return view('components.CRUD-9.index', compact('crud9'));
    }

    public function create()
    {
        // Load categories
        $categories = Crud7::where('status', 'active')->orderBy('name')->get();

        // Load subcategories by AJAX
        $subcategories = collect();

        // pass both variables to view
        return view('components.CRUD-9.create', compact('categories', 'subcategories'));
    }

    // NEW FUNCTION FOR AJAX
    public function getSubcategories($categoryId)
    {
        $subcategories = Crud8::where('crud7_id', $categoryId)
	        ->where('status', 'active')
            ->orderBy('name')
            ->get(['id', 'name']);

        return response()->json($subcategories);
    }

    public function store(Crud9Request $request)
    {
        try {
            Crud9::create($request->validated());

            return redirect()
                ->route('dashboard.crud-9.index')
                ->with('success', 'Data saved successfully.');
        } catch (\Exception $e) {
            return redirect()->back()->with('error', 'An error occurred while saving data.');
        }
    }

    public function edit(string $id)
    {
        try {
            $crud9 = Crud9::findOrFail($id);
            $categories = Crud7::
	            where('status', 'active')->orderBy('name')->get();
            $subcategories = Crud8::where
	            ('status', 'active')
	            ->where('crud7_id', $crud9->category_id ?? $crud9->subcategory->crud7_id ?? 0)
                ->orderBy('name')->get();

            return view('components.CRUD-9.edit', compact('crud9', 'categories', 'subcategories'));
        } catch (\Exception $e) {
            return redirect()->back()->with('error', 'Data not found.');
        }
    }

    public function update(Crud9Request $request, string $id)
    {
        try {
            $crud9 = Crud9::findOrFail($id);
            $crud9->update($request->validated());

            return redirect()
                ->route('dashboard.crud-9.index')
                ->with('success', 'Updated successfully.');
        } catch (\Exception $e) {
            return redirect()->back()->with('error', 'An error occurred while updating data.');
        }
    }

    public function destroy(string $id)
    {
        try {
            $crud9 = Crud9::findOrFail($id);
            $crud9->delete();

            return redirect()
                ->route('dashboard.crud-9.index')
                ->with('success', 'Deleted.');
        } catch (\Exception $e) {
            return redirect()->back()->with('error', 'An error occurred while deleting data.');
        }
    }
}
```
______
## Step-5: (Form Request)

##### **`app\Http\Requests\Crud9Request.php:`**
```php
class Crud9Request extends FormRequest
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
-------
## Step-6: (View Create)

#### **Using Server Side Rendering :**

##### `index.blade.php`: (form only)
```html
<div>
	<table id="dynamic-table" class="table table-striped table-bordered table-hover">
	  <thead>
		<tr>
		  <th style="font-weight: bold;">Sl No</th>
		  <th>Category (From Crud7)</th>
		  <th>Subcategory</th>
		  <th>Sub-Sub Category</th>
		  <th>Slug</th>
		  <th>Serial No</th>
		  <th>Status</th>
		  <th>Action</th>
		</tr>
	  </thead>

	  <tbody>
		@php $sl = $crud9->firstItem() ?? 1; @endphp
		@forelse ($crud9 as $item)
		<tr>
		  <td style="font-weight: bold;">{{ $sl++ }}.</td>

		  {{-- Foreign key data from Crud7 --}}
		  <td>{{ $item->subcategory->category->name ?? 'N/A' }}</td>
		  <td>{{ $item->subcategory->name ?? 'N/A' }}</td>
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

			  <a class="green" href="{{ route('dashboard.crud-9.edit', $item->id) }}">
				<i class="ace-icon fa fa-pencil bigger-130"></i>
			  </a>

			  <form action="{{ route('dashboard.crud-9.destroy', $item->id) }}" method="POST" style="display:inline;" onsubmit="return confirm('Are you sure you want to delete this item?');">
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
	  {{ $crud9->links('pagination::bootstrap-4') }}
	</div>
</div>
```
---
##### `create.blade.php`: (form only)
```html
<!-- Form Start -->
<form action="{{ route('dashboard.crud-9.store') }}" method="POST">
  @csrf

  {{-- Parent Category (From Crud7) --}}
  <div class="form-group">
	<label for="category_id">Category</label>
	<select id="category" name="category" class="form-control" 
			onchange="window.location='{{ route('dashboard.crud-9.create') }}?category=' + this.value">
	  <option value="">-- Select Category --</option>
	  @foreach($categories as $cat)
	  <option value="{{ $cat->id }}" {{ $selectedCategory == $cat->id ? 'selected' : '' }}>
		{{ $cat->name }}
	  </option>
	  @endforeach
	</select>
	@error('category_id') <small class="text-danger">{{ $message }}</small> @enderror
  </div>

  {{-- Parent Subcategory (From Crud8) --}}
  <div class="form-group">
	<label for="crud8_id">Subcategory</label>
	 <select id="crud8_id" name="crud8_id" class="form-control" required 
			 {{ $selectedCategory ? '' : 'disabled' }}>
	   <option value="">-- Select Subcategory --</option>
	   @foreach($subcategories as $sub)
	   <option value="{{ $sub->id }}">{{ $sub->name }}</option>
	   @endforeach
	 </select>

	@error('crud8_id') <small class="text-danger">{{ $message }}</small> @enderror
  </div>

  {{-- Sub-Sub Category Name --}}
  <div class="form-group">
	<label for="name">Sub-Sub Category Name</label>
	<input type="text" id="name" name="name" class="form-control @error('name') is-invalid @enderror" 
		   value="{{ old('name') }}" required>
	@error('name') <small class="text-danger">{{ $message }}</small> @enderror
  </div>

  {{-- Slug (auto-generated) --}}
  <div class="form-group">
	<label for="slug">Slug (Auto)</label>
	<input type="text" id="slug" name="slug" class="form-control" value="{{ old('slug') }}" readonly>
  </div>

  {{-- Serial --}}
  <div class="form-group">
	<label for="serial_no">Serial No</label>
	<input type="number" name="serial_no" class="form-control @error('serial_no') is-invalid @enderror"
		   value="{{ old('serial_no') }}" required>
	@error('serial_no') <small class="text-danger">{{ $message }}</small> @enderror
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

	<a href="{{ route('dashboard.crud-9.index') }}" class="btn btn-sm btn-warning">
	  <i class="ace-icon fa fa-arrow-left bigger-110"></i> Back
	</a>
  </div>
</form>
{{-- Form End --}}
```
___
##### `edit.blade.php`: (form only)
```html
<!-- Edit Form Start -->
<form action="{{ route('dashboard.crud-9.update', $crud9->id) }}" method="POST" enctype="multipart/form-data">
  @csrf
  @method('PUT')

  {{-- Parent Category --}}
  <div class="form-group">
	<label for="category_id">Category</label>
	<select name="category_id" id="category_id" class="form-control"
	onchange="window.location='{{ route('dashboard.crud-9.edit', $crud9->id) }}?category=' + this.value">
	  <option value="">-- Select Category --</option>
	  @foreach($categories as $cat)
	  <option value="{{ $cat->id }}" {{ $selectedCategory == $cat->id ? 'selected' : '' }}>
		{{ $cat->name }}
	  </option>
	  @endforeach
	</select>
	@error('category_id') <small class="text-danger">{{ $message }}</small> @enderror
  </div>

  {{-- Parent Subcategory --}}
  <div class="form-group mt-3">
	<label>Subcategory</label>
	<select name="crud8_id" class="form-control" {{ $subcategories->isEmpty() ? 'disabled' : '' }}>
	  <option value="">-- Select Subcategory --</option>
	  @foreach($subcategories as $sub)
	  <option value="{{ $sub->id }}" {{ $crud9->crud8_id == $sub->id ? 'selected' : '' }}>
		{{ $sub->name }}
	  </option>
	  @endforeach
	</select>
  </div>

  {{-- Name --}}
  <div class="form-group">
	<label for="name">Subcategory Name</label>
	<input type="text" id="name" name="name" class="form-control @error('name') is-invalid @enderror" 
		   value="{{ old('name', $crud9->name) }}" placeholder="Enter subcategory name" required>
	@error('name')
	<small class="text-danger">{{ $message }}</small>
	@enderror
  </div>

  {{-- Slug (auto-generated) --}}
  <div class="form-group">
	<label for="slug">Slug (Auto Generated)</label>
	<input type="text" id="slug" name="slug" class="form-control" 
		   value="{{ old('slug', $crud9->slug) }}" readonly>
  </div>

  {{-- Serial Number --}}
  <div class="form-group">
	<label for="serial_no">Serial No</label>
	<input type="number" name="serial_no" class="form-control @error('serial_no') is-invalid @enderror" 
		   value="{{ old('serial_no', $crud9->serial_no) }}" placeholder="Enter serial number" required>
	@error('serial_no')
	<small class="text-danger">{{ $message }}</small>
	@enderror
  </div>

  {{-- Status --}}
  <div class="form-group">
	<label for="status">Status</label><br>
	<label>
	  <input type="radio" name="status" value="active" 
			 {{ old('status', $crud9->status) == 'active' ? 'checked' : '' }}>
	  Active
	</label>
	&nbsp;&nbsp;
	<label>
	  <input type="radio" name="status" value="inactive"
			 {{ old('status', $crud9->status) == 'inactive' ? 'checked' : '' }}>
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

	<a href="{{ route('dashboard.crud-9.index') }}" class="btn btn-sm btn-warning">
	  <i class="ace-icon fa fa-arrow-left bigger-110"></i> Back
	</a>
  </div>
</form>
<!-- Edit Form End -->
```
____
#### **Using Ajax Rendering :**

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
  </div><!-- Table Header -->
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

			  <form action="{{ route('dashboard.crud-8.destroy', $item->id) }}" method="POST" style="display:inline;" onsubmit="return confirm('Are you sure you want to delete this item?');">
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
____
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
	</div><!-- Table Header -->
	<!-- Main Form Start -->
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
		<input type="text" id="name" name="name" class="form-control @error('name') is-invalid @enderror"
			   value="{{ old('name') }}" placeholder="Enter subcategory name" required>
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
		<input type="number" name="serial_no" class="form-control @error('serial_no') is-invalid @enderror"
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
  
    {{-- Nested Dropdown AJAX --}}
  <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
  <script>
    $(function() {
      const $category = $('#category_id');
      const $sub = $('#crud8_id');

      // Helper: populate subcategory dropdown
      function populateSubcategories(data) {
        $sub.empty().append('<option value="">-- Select Subcategory --</option>');
        if (data.length) {
          data.forEach(item => $sub.append(`<option value="${item.id}">${item.name}</option>`));
          $sub.prop('disabled', false);
        } else {
          $sub.prop('disabled', true);
        }
      }

      // Function to fetch subcategories
      function fetchSubcategories(categoryId) {
        if (!categoryId) {
          populateSubcategories([]);
          return;
        }
        
        // 🔹 Show loading message before request
        $sub.html('<option value="">-- Loading... --</option>').prop('disabled', true);

        $.getJSON('{{ route("dashboard.crud-9.subcategories", ":id") }}'.replace(':id', categoryId))
          .done(data => populateSubcategories(data))
          .fail(() => populateSubcategories([]));
      }

      // On category change
      $category.on('change', function() {
        fetchSubcategories($(this).val());
      });

      // Trigger on page load if old selected category exists
      @if(isset($selectedCategory))
      fetchSubcategories('{{ $selectedCategory }}');
      @endif
    });
  </script>
@endpush
```
_____
##### `edit.blade.php`:
```html
@extends('layouts.app')
@section('content')
@section('title', 'CRUD8 - Sub-Category Edit')
<!-- Edit form -->
<div class="main-content">
	<div class="widget-header widget-header-flat " style="background-color: #618f8f;">
	  <h4 class="widget-title" style="color: #fff;">Edit Sub-Category</h4>

	  <span class="widget-toolbar">
		<a href="{{ route('dashboard.crud-8.index') }}" style="color: #fff;">
		  <i class="ace-icon fa fa-list"></i> Back to List
		</a>
	  </span>
	</div><!-- Table Header -->
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
		<input type="text" id="slug" name="slug" class="form-control" value="{{ old('slug', $crud8->slug) }}" readonly>
	  </div>

	  {{-- Serial Number --}}
	  <div class="form-group">
		<label for="serial_no">Serial No</label>
		<input type="number" name="serial_no" class="form-control @error('serial_no') is-invalid @enderror"
			   value="{{ old('serial_no', $crud8->serial_no) }}" placeholder="Enter serial number" required>
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
</div>
<!-- /.main-content -->
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


{{-- AJAX script for nested dropdown --}}
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script>
  $(function() {
    const $category = $('#category_id');
    const $sub = $('#crud8_id');

    function populateSubcategories(data, selectedId = null) {
      $sub.empty().append('<option value="">-- Select Subcategory --</option>');
      if (data.length) {
        data.forEach(item => {
          $sub.append(`<option value="${item.id}" ${item.id == selectedId ? 'selected':''}>${item.name}</option>`);
        });
        $sub.prop('disabled', false);
      } else {
        $sub.prop('disabled', true);
      }
    }

    $category.on('change', function() {
      const catId = $(this).val();
      if (!catId) {
        populateSubcategories([]);
        return;
      }
      
      // 🔹 Show loading message before request
        $sub.html('<option value="">-- Loading... --</option>').prop('disabled', true);

      $.getJSON('{{ route("dashboard.crud-9.subcategories", ":id") }}'.replace(':id', catId))
        .done(data => populateSubcategories(data))
        .fail(() => populateSubcategories([]));
    });

    // Trigger fetch for existing selected category on page load
    @if(isset($crud9))
    $category.trigger('change');
    // optionally pass selected subcategory id
    $.getJSON('{{ route("dashboard.crud-9.subcategories", ":id") }}'.replace(':id', '{{ $crud9->subcategory->crud7_id }}'))
      .done(data => populateSubcategories(data, '{{ $crud9->crud8_id }}'));
    @endif
  });
</script>
@endpush
```
_____
