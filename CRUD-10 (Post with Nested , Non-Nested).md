এই CRUD এ আমরা Nested / Non-Nested দুইভাবেই পোষ্ট করতে পারবো। এর কার্যপদ্ধতি নিম্নরূপঃ

```scss
				category -> Post (minimum cate এর অধীনে পোষ্ট করতে হবে।) 
				category -> sub-category -> Post
				category -> sub-sub-category -> Post
```

Laravel -এ এই CRUD Operation এর জন্য ৬টি Step Follow করতে হয়। সেগুলো হলোঃ

1. Routes               =>  `routes\web.php`
2. Model                => `app\Models\Crud10.php`
3. Migration          => `database\migrations\2025_04_14_164123_create_crud10s_table.php`
4. Controller         => `app\Http\Controllers\Crud10Controller.php`
5. Request form   =>  `app\Http\Requests\Crud10Request.php`
6. Views                =>  `index.blade.php` , `create.blade.php` , `edit.blade.php`
   
নিচে এগুলোর বিস্তারিত বর্ণনা দেয়া হলোঃ

#### **Workflow**:
```scss
						(Route → Middleware (if any) → Controller → FormRequest → Model → View)
```
____

## Step-1: (Web Route)

`routes/web.php`:
```php
Route::group(['prefix' => 'dashboard', 'as' => 'dashboard.'], function () {

    Route::resources([
    
        // CRUD (Post)
        'crud-10' => Crud10Controller::class,  // Post
    ]);
});
```
_____
## Step-2: (Model)

`app\Models\Crud10.php`:
```php
class Crud10 extends Model
{
    use HasFactory;

    protected $fillable = [
        'crud7_id',
        'crud8_id',
        'crud9_id',
        'post_serial',
        'post_name',
        'post_title',
        'short_description',
        'post',
        'status',
    ];

    public function category()
    {
        return $this->belongsTo(Crud7::class, 'crud7_id');
    }

    public function subcategory()
    {
        return $this->belongsTo(Crud8::class, 'crud8_id');
    }

    public function subsubcategory()
    {
        return $this->belongsTo(Crud9::class, 'crud9_id');
    }
}
```
____
## Step-3: (Migration)

`database\migrations\2025_05_05_100447_create_crud10s_table.php`:
```php
	Schema::create('crud10s', function (Blueprint $table) {
		$table->id();
		// Foreign keys
		$table->foreignId('crud7_id')->constrained('crud7s')->onDelete('cascade'); // category required
		$table->foreignId('crud8_id')->nullable()->constrained('crud8s')->onDelete('set null'); // sub-category optional
		$table->foreignId('crud9_id')->nullable()->constrained('crud9s')->onDelete('set null'); // sub-sub optional

		// Post Fields
		$table->string('post_serial')->unique();
		$table->string('post_name')->unique();
		$table->string('post_title');
		$table->text('short_description')->nullable();
		$table->longText('post');
		
		$table->enum('status', ['active', 'inactive'])->default('active');
		$table->timestamps();
	});
```
_____
## Step-4: (Controller)

#### **Using Server Side Rendering :**

**`app\Http\Requests\Crud10Request.php:`**
```php
class Crud10Controller extends Controller
{
    public function index()
    { 
        $crud10 = Crud10::with(['category', 'subcategory', 'subsubcategory'])
            ->orderBy('post_serial', 'asc')
            ->paginate(10);

        return view('components.CRUD-10.index', compact('crud10'));
    }

    public function create()
    {
        // সব category
        $categories = Crud7::orderBy('name')->get();

        // যদি কোনো category select করা থাকে (query string থেকে)
        $selectedCategory = request()->query('category');

        // যদি category select করা থাকে → তার subcategories আনবে
        $subcategories = $selectedCategory
            ? Crud8::where('crud7_id', $selectedCategory)->orderBy('name')->get()
            : collect();

        // যদি subcategory select করা থাকে → তার sub-subcategories আনবে
        $selectedSubcategory = request()->query('subcategory');
        $subsubcategories = $selectedSubcategory
            ? Crud9::where('crud8_id', $selectedSubcategory)->orderBy('name')->get()
            : collect();

        return view('components.CRUD-10.create', compact(
            'categories',
            'subcategories',
            'subsubcategories',
            'selectedCategory',
            'selectedSubcategory'
        ));
    }

    public function store(Crud10Request $request)
    {
        try {
            Crud10::create($request->validated());
            return redirect()->route('dashboard.crud-10.index')
                ->with('success', 'Post created successfully.');

            // return redirect()->route('dashboard.crud-10.index')->with('success', 'Post created successfully.');
        } catch (\Throwable $th) {
            return back()->withInput()->with('error', 'Error: ' . $th->getMessage());
        }
    }

    public function show(string $id)
    {
        //
    }

    public function edit(string $id)
    {
        try {
            $crud10 = Crud10::findOrFail($id);
            $categories = Crud7::orderBy('name')->get();
            $subcategories = Crud8::orderBy('name')->get();
            $subsubcategories = Crud9::orderBy('name')->get();

            return view('components.CRUD-10.edit', compact(
                'crud10',
                'categories',
                'subcategories',
                'subsubcategories'
            ));
        } catch (\Throwable $th) {
            return back()->with('error', 'Error: ' . $th->getMessage());
        }
    }

    public function update(Crud10Request $request, string $id)
    {
        try {
            $crud10 = Crud10::findOrFail($id);
            $crud10->update($request->validated());
            return redirect()->route('dashboard.crud-10.index')->with('success', 'Post updated successfully.');
        } catch (\Throwable $th) {
            return back()->with('error', 'Error: ' . $th->getMessage());
        }
    }

    public function destroy(string $id)
    {
        try {
            Crud10::findOrFail($id)->delete();
            return redirect()->route('dashboard.crud-10.index')->with('success', 'Post deleted successfully.');
        } catch (\Throwable $th) {
            return back()->with('error', 'Error: ' . $th->getMessage());
        }
    }
}
```
----

#### **Using Ajax Rendering :**

`app\Http\Controllers\Crud10Controller.php`:
```php
		 Under Construction!!
```
______

## Step-5: (Form Request)


**`app\Http\Requests\Crud10Request.php:`**
```php
class Crud10Request extends FormRequest
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
            'crud7_id' => 'required|exists:crud7s,id',
            'crud8_id' => 'nullable|exists:crud8s,id',
            'crud9_id' => 'nullable|exists:crud9s,id',
            'post_serial' => 'required|integer|unique:crud10s,post_serial',
            'post_name'   => 'required|string|max:255|unique:crud10s,post_name',
            'post_title'  => 'required|string|max:255',
            'short_description' => 'nullable|string|max:500',
            'post'        => 'required|string',
            'status'      => 'required|in:active,inactive',
        ];
    }

    protected function updateRules(): array
    {
        $crud10 = $this->route('crud10') ?? $this->route('crud_10') ?? $this->route('crud-10');
        $id = $crud10?->id ?? $crud10;

        return [
            'crud7_id'    => 'required|exists:crud7s,id',
            'crud8_id'    => 'nullable|exists:crud8s,id',
            'crud9_id'    => 'nullable|exists:crud9s,id',
            'post_serial' => ['required', 'integer', Rule::unique('crud10s', 'post_serial')->ignore($id)],
            'post_name'   => ['required', 'string', 'max:255', Rule::unique('crud10s', 'post_name')->ignore($id)],
            'post_title'  => 'required|string|max:255',
            'short_description' => 'nullable|string|max:500',
            'post'        => 'required|string',
            'status'      => 'required|in:active,inactive',
        ];
    }

    public function messages(): array
    {
        return [
            'crud7_id.required' => 'Please select a category.',
            'post_name.required' => 'Post name is required.',
            'post_serial.unique' => 'This serial number already exists.',
        ];
    }
}
```
-------

## Step-6: (View Create)

#### **Using Server Side Rendering :**

`index.blade.php`: (form only)
```html
@extends('layouts.app')
@section('content')
@section('title', 'CRUD10 - Post List')
<div class="main-content">
  <div class="main-content-inner">
    <div class="breadcrumbs ace-save-state" id="breadcrumbs">
      <ul class="breadcrumb">
        <li>
          <i class="ace-icon fa fa-home home-icon"></i>
          <a href="#">Home</a>
        </li>

        <li>
          <a href="#">Tables</a>
        </li>
        <li class="active">CRUD-10 List</li>
      </ul>

      <div class="nav-search" id="nav-search">
        <form class="form-search">
          <span class="input-icon">
            <input type="text" placeholder="Search ..." class="nav-search-input" id="nav-search-input" autocomplete="off" />
            <i class="ace-icon fa fa-search nav-search-icon"></i>
          </span>
        </form>
      </div>
    </div>

    <div class="page-content">

      <div class="page-header">
        <h1>
          Tables
          <small>
            <i class="ace-icon fa fa-angle-double-right"></i>
            CRUD-10 List
          </small>
        </h1>
      </div>

      <div class="row">
        <div class="col-xs-12">

          <div class="row">
            <div class="col-xs-12">
              <div class="clearfix">
                <div class="pull-right tableTools-container"></div>
              </div>

              <div class="widget-header widget-header-flat" style="background-color: #618f8f;">
                <h4 class="widget-title" style="color: #fff;">Sub Sub Category List</h4>
                <span class="widget-toolbar">
                  <a href="{{ route('dashboard.crud-10.create') }}" style="color: #fff;">
                    <i class="ace-icon fa fa-plus"></i> Create New Sub-Category
                  </a>
                </span>
              </div>

              <div>
                <table id="dynamic-table" class="table table-striped table-bordered table-hover">
                  <thead class="table-dark text-center">
                    <tr>
                      <th>SL</th>
                      <th>Category</th>
                      <th>Subcategory</th>
                      <th>Sub-Subcategory</th>
                      <th>Post Serial</th>
                      <th>Post Name</th>
                      <th>Post Title</th>
                      <th>Short Description</th>
                      <th>Post</th>
                      <th>Status</th>
                      <th width="140">Action</th>
                    </tr>

                  </thead>

                  <tbody>
                    @php $sl = $crud10->firstItem() ?? 1; @endphp
                    @forelse ($crud10 as $item)
                    <tr>
                      <td style="font-weight: bold;">{{ $sl++ }}.</td>

                      <td>{{ $item->category->name ?? '-' }}</td>
                      <td>{{ $item->subcategory->name ?? '-' }}</td>
                      <td>{{ $item->subsubcategory->name ?? '-' }}</td>
                      <td>{{ $item->post_serial }}</td>

                      <td>{{ $item->post_name }}</td>
                      <td>{{ $item->post_title }}</td>
                      <td> {{ $item->short_description }}</td>

                      <td>{{ $item->post }}</td>
                      
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

                          <a class="green" href="{{ route('dashboard.crud-10.edit', $item->id) }}">
                            <i class="ace-icon fa fa-pencil bigger-130"></i>
                          </a>

                          <form action="{{ route('dashboard.crud-10.destroy', $item->id) }}" method="POST" style="display:inline;" onsubmit="return confirm('Are you sure you want to delete this item?');">
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
                      <td colspan="9" class="text-center text-danger">No data found.</td>
                    </tr>
                    @endforelse
                  </tbody>
                </table>
                <div class="text-center">
                  {{ $crud10->links('pagination::bootstrap-4') }}
                </div>
              </div>
            </div>
          </div>
        </div><!-- /.col -->
      </div>
    </div><!-- /.page-content -->
  </div>
</div>
@endsection
```
---

`create.blade.php`: (form only)
```html
@extends('layouts.app')
@section('content')
@section('title', 'CRUD10 - Post Create')
<!-- Table is here -->
<div class="main-content">
  <div class="main-content-inner">
    <div class="breadcrumbs ace-save-state" id="breadcrumbs">
      <ul class="breadcrumb">
        <li>
          <i class="ace-icon fa fa-home home-icon"></i>
          <a href="#">Home</a>
        </li>

        <li>
          <a href="#">Tables</a>
        </li>
        <li class="active">Create New Post</li>
      </ul><!-- /.breadcrumb -->

      <div class="nav-search" id="nav-search">
        <form class="form-search">
          <span class="input-icon">
            <input type="text" placeholder="Search ..." class="nav-search-input" id="nav-search-input" autocomplete="off" />
            <i class="ace-icon fa fa-search nav-search-icon"></i>
          </span>
        </form>
      </div><!-- /.nav-search -->
    </div>

    <div class="page-content">

      <div class="page-header">
        <h1>
          Tables
          <small>
            <i class="ace-icon fa fa-angle-double-right"></i>
            Create New Post
          </small>
        </h1>
      </div>

      <div class="row">
        <div class="col-xs-12">
          <!-- PAGE CONTENT BEGINS -->

          @if(session('error'))
          <div class="alert alert-danger">{{ session('error') }}</div>
          @endif

          @if(session('success'))
          <div class="alert alert-success">{{ session('success') }}</div>
          @endif
          
          <div class="row">
            <div class="col-xs-12 ">
              <div class="col-md-2"></div>

              <div class="col-md-8">
                <div class="clearfix">
                  <div class="pull-right tableTools-container"></div>
                </div>
                <div class="widget-header widget-header-flat " style="background-color: #618f8f;">
                  <h4 class="widget-title" style="color: #fff;">Create Post</h4>

                  <span class="widget-toolbar">
                    <a href="{{ route('dashboard.crud-10.index') }}" style="color: #fff;">
                      <i class="ace-icon fa fa-plus"></i> Go To Index
                    </a>
                  </span>
                </div>
                <!-- div.table-responsive -->
                <!-- Create Form Start -->
                <form action="{{ route('dashboard.crud-10.store') }}" method="POST">
                  @csrf

                  {{-- Category --}}
                  <div class="form-group">
                    <label>Category <span class="text-danger">*</span></label>
                    <select id="crud7_id" name="crud7_id" class="form-control" onchange="window.location='{{ route('dashboard.crud-10.create') }}?category=' + this.value">
                      <option value="">-- Select Category --</option>
                      @foreach($categories as $cat)
                      <option value="{{ $cat->id }}" {{ old('crud7_id', $selectedCategory) == $cat->id ? 'selected' : '' }}>
                        {{ $cat->name }}
                      </option>
                      @endforeach
                    </select>
                    @error('crud7_id') <small class="text-danger">{{ $message }}</small> @enderror
                  </div>

                  {{-- Subcategory --}}
                  <div class="form-group">
                    <label>Subcategory</label>
                    <select name="crud8_id" id="crud8_id" class="form-control" onchange="window.location='{{ route('dashboard.crud-10.create') }}?category={{ $selectedCategory }}&subcategory=' + this.value">
                      <option value="">-- Select Subcategory --</option>
                      @foreach($subcategories as $sub)
                      <option value="{{ $sub->id }}" {{ old('crud8_id', $selectedSubcategory) == $sub->id ? 'selected' : '' }}>
                        {{ $sub->name }}
                      </option>
                      @endforeach
                    </select>
                    @error('crud8_id') <small class="text-danger">{{ $message }}</small> @enderror
                  </div>

                  {{-- Sub-Subcategory --}}
                  <div class="form-group">
                    <label>Sub-Subcategory (optional)</label>
                    <select name="crud9_id" id="crud9_id" class="form-control">
                      <option value="">-- Select Sub-Subcategory --</option>
                      @foreach($subsubcategories as $subsub)
                      <option value="{{ $subsub->id }}" {{ old('crud9_id') == $subsub->id ? 'selected' : '' }}>
                        {{ $subsub->name }}
                      </option>
                      @endforeach
                    </select>
                    @error('crud9_id') <small class="text-danger">{{ $message }}</small> @enderror
                  </div>

                  {{-- Post Serial --}}
                  <div class="form-group mb-3">
                    <label>Post Serial *</label>
                    <input type="number" name="post_serial" class="form-control" value="{{ old('post_serial') }}" required>
                    @error('post_serial') <span class="text-danger">{{ $message }}</span> @enderror
                  </div>

                  {{-- Post Name --}}
                  <div class="form-group mb-3">
                    <label>Post Name *</label>
                    <input type="text" name="post_name" class="form-control" value="{{ old('post_name') }}" required>
                    @error('post_name') <span class="text-danger">{{ $message }}</span> @enderror
                  </div>

                  {{-- Post Title --}}
                  <div class="form-group mb-3">
                    <label>Post Title *</label>
                    <input type="text" name="post_title" class="form-control" value="{{ old('post_title') }}" required>
                    @error('post_title') <span class="text-danger">{{ $message }}</span> @enderror
                  </div>

                  {{-- Short Description --}}
                  <div class="form-group mb-3">
                    <label>Short Description</label>
                    <textarea name="short_description" class="form-control" rows="2">{{ old('short_description') }}</textarea>
                    @error('short_description') <small class="text-danger">{{ $message }}</small> @enderror
                  </div>

                  {{-- Full Post --}}
                  <div class="form-group mb-3">
                    <label>Full Post *</label>
                    <textarea name="post" class="form-control" rows="5">{{ old('post') }}</textarea>
                    @error('post') <small class="text-danger">{{ $message }}</small> @enderror
                  </div>

                  {{-- Status --}}
                  <div class="form-group mb-3">
                    <label>Status *</label>
                    <select name="status" class="form-select" required>
                      <option value="active" {{ old('status') == 'active' ? 'selected' : '' }}>Active</option>
                      <option value="inactive" {{ old('status') == 'inactive' ? 'selected' : '' }}>Inactive</option>
                    </select>
                    @error('status') <small class="text-danger">{{ $message }}</small> @enderror
                  </div>

                  {{-- Submit --}}
                  <div class="form-actions text-center mt-3">
                    <button type="submit" class="btn btn-success btn-sm">
                      <i class="fa fa-save"></i> Save
                    </button>
                    <a href="{{ route('dashboard.crud-10.index') }}" class="btn btn-warning btn-sm">
                      <i class="fa fa-arrow-left"></i> Back
                    </a>
                  </div>
                </form>
                {{-- Form End --}}
              </div>
              <div class="col-md-2"></div>
            </div>
          </div>
          <!-- PAGE CONTENT ENDS -->
        </div><!-- /.col -->
      </div>
    </div><!-- /.page-content -->
  </div>
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
@endpush
```
___

`edit.blade.php`: (form only)
```html
@extends('layouts.app')
@section('content')
@section('title', 'CRUD10 - Post Edit')
<!-- Edit form -->
<div class="main-content">
  <div class="main-content-inner">
    <div class="breadcrumbs ace-save-state" id="breadcrumbs">
      <ul class="breadcrumb">
        <li><i class="ace-icon fa fa-home home-icon"></i><a href="#">Home</a></li>
        <li><a href="#">Tables</a></li>
        <li class="active">Edit Sub-Sub-Category</li>
      </ul>
    </div>

    <div class="page-content">
      <div class="page-header">
        <h1>
          Edit Entry
          <small><i class="ace-icon fa fa-angle-double-right"></i> Update Existing Sub-Sub-Category</small>
        </h1>
      </div>
      <div class="row">
        <div class="col-xs-12">
          <div class="row">
            <div class="col-xs-12 ">
              <div class="col-md-2"></div>

              <div class="col-md-8">
                <div class="widget-header widget-header-flat " style="background-color: #618f8f;">
                  <h4 class="widget-title" style="color: #fff;">Edit Sub-Sub-Category</h4>

                  <span class="widget-toolbar">
                    <a href="{{ route('dashboard.crud-9.index') }}" style="color: #fff;">
                      <i class="ace-icon fa fa-list"></i> Back to List
                    </a>
                  </span>
                </div>

                <!-- Edit Form Start -->
                <form action="{{ route('dashboard.crud-10.update', $crud10->id) }}" method="POST">
                  @csrf
                  @method('PUT')

                  <div class="mb-3">
                    <label>Category *</label>
                    <select name="crud7_id" class="form-control" required>
                      <option value="">-- Select Category --</option>
                      @foreach($categories as $cat)
                      <option value="{{ $cat->id }}" {{ $crud10->crud7_id == $cat->id ? 'selected' : '' }}>
                        {{ $cat->name }}
                      </option>
                      @endforeach
                    </select>
                  </div>

                  <div class="mb-3">
                    <label>Sub-Category</label>
                    <select name="crud8_id" class="form-control">
                      <option value="">-- Optional --</option>
                      @foreach($subcategories as $sub)
                      <option value="{{ $sub->id }}" {{ $crud10->crud8_id == $sub->id ? 'selected' : '' }}>
                        {{ $sub->name }}
                      </option>
                      @endforeach
                    </select>
                  </div>

                  <div class="mb-3">
                    <label>Sub-Sub-Category</label>
                    <select name="crud9_id" class="form-control">
                      <option value="">-- Optional --</option>
                      @foreach($subsubcategories as $subsub)
                      <option value="{{ $subsub->id }}" {{ $crud10->crud9_id == $subsub->id ? 'selected' : '' }}>
                        {{ $subsub->name }}
                      </option>
                      @endforeach
                    </select>
                  </div>

                  <div class="mb-3">
                    <label>Post Serial *</label>
                    <input type="number" name="post_serial" class="form-control" value="{{ $crud10->post_serial }}" required>
                  </div>

                  <div class="mb-3">
                    <label>Post Name *</label>
                    <input type="text" name="post_name" class="form-control" value="{{ $crud10->post_name }}" required>
                  </div>

                  <div class="mb-3">
                    <label>Post Title *</label>
                    <input type="text" name="post_title" class="form-control" value="{{ $crud10->post_title }}" required>
                  </div>

                  <div class="mb-3">
                    <label>Short Description</label>
                    <textarea name="short_description" class="form-control">{{ $crud10->short_description }}</textarea>
                  </div>

                  <div class="mb-3">
                    <label>Post Content *</label>
                    <textarea name="post" class="form-control" rows="5">{{ $crud10->post }}</textarea>
                  </div>

                  <div class="mb-3">
                    <label>Status *</label>
                    <select name="status" class="form-control" required>
                      <option value="active" {{ $crud10->status == 'active' ? 'selected' : '' }}>Active</option>
                      <option value="inactive" {{ $crud10->status == 'inactive' ? 'selected' : '' }}>Inactive</option>
                    </select>
                  </div>

                  <button type="submit" class="btn btn-success">Update Post</button>
                  <a href="{{ route('dashboard.crud-10.index') }}" class="btn btn-secondary">Cancel</a>
                </form>

                <!-- Edit Form End -->
              </div>
              <div class="col-md-2"></div>
            </div>
          </div>
        </div><!-- /.col -->
      </div>
    </div><!-- /.page-content -->
  </div>
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
@endpush
```
____

#### **Using Ajax Rendering :**

`index.blade.php`:
```html
		 Under Construction!!
```

`create.blade.php`:
```html
		 Under Construction!!
```

`edit.blade.php`:
```html
		 Under Construction!!
```
_____
