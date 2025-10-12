
Laravel -এ CRUD Operation এর জন্য ৫টি Step Follow করতে হয়। সেগুলো হলোঃ

1. Routes               =>  `routes\web.php`
2. Model                => `app\Models\Crud7.php`
3. Migration          => `database\migrations\2025_04_14_164123_create_crud7s_table.php`
4. Controller         => `app\Http\Controllers\Crud7Controller.php`
5. Request form   =>  `app\Http\Requests\Crud7Request.php`
6. Views                =>  `index.blade.php` , `create.blade.php` , `edit.blade.php`
   
নিচে এগুলোর বিস্তারিত বর্ণনা দেয়া হলোঃ

#### **Workflow**:
```scss
						(Route → Middleware (if any) → Controller → FormRequest → Model → View)
```
____

## Step-1: (Route Create)

`routes/web.php`:
```php
Route::group(['prefix' => 'dashboard', 'as' => 'dashboard.'], function () {

    Route::resources([
    
        'crud-7' => Crud7Controller::class,  // Category
    ]);
});
```
_____
## Step-2:

`app\Models\Crud7.php`:
```php
class Crud7 extends Model
{
    use HasFactory;

    protected $fillable = [
        'name',
        // 'slug',
        'serial_no',
        'status',
    ];

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

    // Relationship with Crud8 (Subcategory)
    public function subcategories()
    {
        return $this->hasMany(Crud8::class, 'crud7_id');
    }
}
```
____
## Step-3:

`database\migrations\2025_05_05_100447_create_crud7s_table.php`:
```php
	Schema::create('crud7s', function (Blueprint $table) {
		$table->id();
		$table->string('name')->unique();
		$table->string('slug')->unique();
		$table->integer('serial_no')->unique();
		$table->enum('status', ['active', 'inactive'])->default('active');
		$table->timestamps();
	});
```
_____
## Step-4:

`app\Http\Controllers\Crud7Controller.php`:
```php
class Crud7Controller extends Controller
{
    /**
     * Display a listing of the resource.
     */
    public function index()
    {
        $crud7 = Crud7::orderBy('serial_no', 'asc')->paginate(10);
        return view('components.CRUD-7.index', compact('crud7'));
    }

    /**
     * Show the form for creating a new resource.
     */
    public function create()
    {
        return view('components.CRUD-7.create');
    }

    /**
     * Store a newly created resource in storage.
     */
    public function store(Crud7Request $request)
    {
        try {
            Crud7::create($request->validated());

            return redirect()
                ->route('dashboard.crud-7.index')
                ->with('success', 'Category created successfully!');
        } catch (\Throwable $th) {
            return redirect()->back()->with('error', 'Something went wrong: ' . $th->getMessage());
        }
    }

    /**
     * Display the specified resource.
     */
    public function show(string $id)
    {
        //
    }

    /**
     * Show the form for editing the specified resource.
     */
    public function edit(string $id)
    {
        $crud7 = Crud7::findOrFail($id);
        return view('components.CRUD-7.edit', compact('crud7'));
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(Crud7Request $request, string $id)
    {
        try {
            $crud7 = Crud7::findOrFail($id);
            $crud7->update($request->validated());
            return redirect()->route('dashboard.crud-7.index')->with('success', 'Category updated successfully.');
        } catch (\Throwable $th) {
            return redirect()->back()->with('error', 'Something went wrong: ' . $th->getMessage());
        }
    }

    /**
     * Remove the specified resource from storage.
     */
    public function destroy(string $id)
    {
        try {
            $crud7 = Crud7::findOrFail($id);
            $crud7->delete();
            return redirect()->route('dashboard.crud-7.index')->with('success', 'Category deleted successfully.');
        } catch (\Throwable $th) {
            return redirect()->back()->with('error', 'Something went wrong: ' . $th->getMessage());
        }
    }
}
```
______

## Step-5:

**`app\Http\Requests\Crud4Request.php:`**
```php
class Crud7Request extends FormRequest
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
## Step-6:

`index.blade.php`:
```html
@extends('layouts.app')
@section('content')
@section('title', 'CRUD7 - Category List')
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
        <li class="active">CRUD-7 Category List</li>
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
            CRUD-7 Category List
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
                <h4 class="widget-title" style="color: #fff;">Category List</h4>
                <span class="widget-toolbar">
                  <a href="{{ route('dashboard.crud-7.create') }}" style="color: #fff;">
                    <i class="ace-icon fa fa-plus"></i> Create New Category
                  </a>
                </span>
              </div>

              <div>
                <table id="dynamic-table" class="table table-striped table-bordered table-hover">
                  <thead>
                    <tr>
                      <th style="font-weight: bold;">Sl No</th>
                      <th>Name</th>
                      <th>Slug</th>
                      <th>Serial No</th>
                      <th>Status</th>
                      <th>Action</th>
                    </tr>
                  </thead>

                  <tbody>
                    @php
                    $sl = $crud7->firstItem() ?? 0;
                    @endphp

                    @forelse ($crud7 as $item)
                    <tr>
                      <td style="font-weight: bold;">{{ $sl++ }}.</td>
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

                          <a class="green" href="{{ route('dashboard.crud-7.edit', $item->id) }}">
                            <i class="ace-icon fa fa-pencil bigger-130"></i>
                          </a>

                          <form action="{{ route('dashboard.crud-7.destroy', $item->id) }}" method="POST" style="display:inline;" onsubmit="return confirm('Are you sure you want to delete this item?');">
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
                      <td colspan="6" class="text-center text-danger">No data found.</td>
                    </tr>
                    @endforelse
                  </tbody>

                </table>

                <div class="text-center">
                  {{ $crud7->links('pagination::bootstrap-4') }}
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

`create.blade.php`:
```html
@extends('layouts.app')
@section('content')
@section('title', 'CRUD7 - Category Create')
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
        <li class="active">Create New Category Table</li>
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
            Create New Category
          </small>
        </h1>
      </div>

      <div class="row">
        <div class="col-xs-12">
          <!-- PAGE CONTENT BEGINS -->



          <div class="row">
            <div class="col-xs-12 ">
              <div class="col-md-2"></div>

              <div class="col-md-8">
                <div class="clearfix">
                  <div class="pull-right tableTools-container"></div>
                </div>
                <div class="widget-header widget-header-flat " style="background-color: #618f8f;">
                  <h4 class="widget-title" style="color: #fff;">Create Category</h4>

                  <span class="widget-toolbar">
                    <a href="{{ route('dashboard.crud-7.index') }}" style="color: #fff;">
                      <i class="ace-icon fa fa-plus"></i> Go To Index
                    </a>
                  </span>
                </div>

                <!-- div.table-responsive -->

                <!-- div.dataTables_borderWrap -->
                <form action="{{ route('dashboard.crud-7.store') }}" method="POST">
                  @csrf

                  {{-- Name --}}
                  <div class="form-group">
                    <label for="name">Category Name</label>
                    <input type="text" id="name" {{-- ✅ এখানে id যোগ করা হলো --}} name="name" class="form-control @error('name') is-invalid @enderror" value="{{ old('name') }}" placeholder="Enter category name" required>
                    @error('name')
                    <small class="text-danger">{{ $message }}</small>
                    @enderror
                  </div>


                  {{-- Slug (auto-generated) --}}
                  <div class="form-group">
                    <label for="slug">Slug (Auto Generated)</label>
                    <input type="text" name="slug" id="slug" class="form-control" value="{{ old('slug') }}" placeholder="Auto generated slug" readonly>
                  </div>


                  {{-- Serial Number --}}
                  <div class="form-group">
                    <label for="serial_no">Serial No</label>
                    <input type="number" name="serial_no" class="form-control @error('serial_no') is-invalid @enderror" value="{{ old('serial_no') }}" placeholder="Enter serial number" required>
                    @error('serial_no')
                    <small class="text-danger">{{ $message }}</small>
                    @enderror
                  </div>

                  {{-- Status --}}
                  <div class="form-group">
                    <label for="status">Status</label><br>
                    <label>
                      <input type="radio" name="status" value="active" {{ old('status', 'active') == 'active' ? 'checked' : '' }}>
                      Active
                    </label>
                    &nbsp;&nbsp;
                    <label>
                      <input type="radio" name="status" value="inactive" {{ old('status') == 'inactive' ? 'checked' : '' }}>
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

                    <a href="{{ route('dashboard.crud-7.index') }}" class="btn btn-sm btn-warning">
                      <i class="ace-icon fa fa-arrow-left bigger-110"></i> Back
                    </a>
                  </div>
                </form>

                {{-- ফর্ম শেষ --}}
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

`edit.blade.php`:
```html
@extends('layouts.app')

@section('content')
@section('title', 'CRUD7 - Category Edit')
<!-- Edit form -->
<div class="main-content">
  <div class="main-content-inner">
    <div class="breadcrumbs ace-save-state" id="breadcrumbs">
      <ul class="breadcrumb">
        <li><i class="ace-icon fa fa-home home-icon"></i><a href="#">Home</a></li>
        <li><a href="#">Tables</a></li>
        <li class="active">Edit Category Table</li>
      </ul>
    </div>

    <div class="page-content">
      <div class="page-header">
        <h1>
          Edit Entry
          <small><i class="ace-icon fa fa-angle-double-right"></i> Update Category</small>
        </h1>
      </div>

      <div class="row">
        <div class="col-xs-12">
          <div class="row">
            <div class="col-xs-12 ">
              <div class="col-md-2"></div>

              <div class="col-md-8">
                <div class="widget-header widget-header-flat " style="background-color: #618f8f;">
                  <h4 class="widget-title" style="color: #fff;">Edit Category</h4>

                  <span class="widget-toolbar">
                    <a href="{{ route('dashboard.crud-7.index') }}" style="color: #fff;">
                      <i class="ace-icon fa fa-list"></i> Back to List
                    </a>
                  </span>
                </div>

                <!-- Edit Form Start -->
                <form action="{{ route('dashboard.crud-7.update', $crud7->id) }}" method="POST" enctype="multipart/form-data">
                  @csrf
                  @method('PUT')

                  {{-- Name --}}
                  <div class="form-group">
                    <label for="name">Category Name</label>
                    <input type="text" id="name" name="name" class="form-control @error('name') is-invalid @enderror" value="{{ old('name', $crud7->name) }}" placeholder="Enter category name" required>
                    @error('name')
                    <small class="text-danger">{{ $message }}</small>
                    @enderror
                  </div>

                  {{-- Slug (auto-generated) --}}
                  <div class="form-group">
                    <label for="slug">Slug (Auto Generated)</label>
                    <input type="text" id="slug" name="slug" class="form-control" value="{{ old('slug', $crud7->slug) }}" readonly>
                  </div>

                  {{-- Serial Number --}}
                  <div class="form-group">
                    <label for="serial_no">Serial No</label>
                    <input type="number" name="serial_no" class="form-control @error('serial_no') is-invalid @enderror" value="{{ old('serial_no', $crud7->serial_no) }}" placeholder="Enter serial number" required>
                    @error('serial_no')
                    <small class="text-danger">{{ $message }}</small>
                    @enderror
                  </div>

                  {{-- Status --}}
                  <div class="form-group">
                    <label for="status">Status</label><br>
                    <label>
                      <input type="radio" name="status" value="active" {{ old('status', $crud7->status) == 'active' ? 'checked' : '' }}> Active
                    </label>
                    &nbsp;&nbsp;
                    <label>
                      <input type="radio" name="status" value="inactive" {{ old('status', $crud7->status) == 'inactive' ? 'checked' : '' }}> Inactive
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

                    <a href="{{ route('dashboard.crud-7.index') }}" class="btn btn-sm btn-warning">
                      <i class="ace-icon fa fa-arrow-left bigger-110"></i> Back
                    </a>
                  </div>
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
_____
