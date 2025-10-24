
Laravel -এ CRUD Operation এর জন্য 6টি Step Follow করতে হয়। সেগুলো হলোঃ

1. Routes               =>  `routes\web.php`
2. Model                => `app\Models\Crud5.php`
3. Migration          => `database\migrations\2025_04_14_164123_create_crud5_table.php`
4. Controller         => `app\Http\Controllers\Crud5Controller.php`
5. Request form   =>  `app\Http\Requests\Crud5Request.php`
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
        'crud-5' => Crud5Controller::class,
    ]);
});
```
_____
## Step-2: (Model)

##### `app\Models\Crud5.php`:
```php
class Crud5 extends Model
{
    use HasFactory;
    protected $fillable = [
        'name',
        'email',
        'phone',
        'image',
    ];
}
```
____
## Step-3: (Migration)

##### `database\migrations\2025_05_05_100447_create_crud5s_table.php`:
```php
    Schema::create('crud5s', function (Blueprint $table) {
		$table->id();
		$table->string('name');
		$table->string('email');
		$table->string('phone');
		$table->text('image');
		$table->timestamps();
	});
```
_____
## Step-4: (Controller)

##### `app\Http\Controllers\Crud5Controller.php`:
```php
class Crud5Controller extends Controller
{
    public function index()
    {
        $crud5 = Crud5::orderby('id', 'asc')->paginate(3);
        return view('components.CRUD-5.index', compact('crud5'));
    }

    public function create()
    {
        return view('components.CRUD-5.create');
    }

    public function store(Crud5Request $request)
    {
        try {
            $validated = $request->validated();
            $img_path = null;
            if ($request->hasFile('image')) {
                $image = $request->file('image');
                $filename = time() . '.' . $image->getClientOriginalExtension();
                $image->move(public_path('uploads/images/crud5'), $filename);
                $img_path = 'uploads/images/crud5/' . $filename;
            }
            Crud5::create([
                'name'  => $validated['name'],
                'email' => $validated['email'],
                'phone' => $validated['phone'],
                'image' => $img_path,
            ]);

            return redirect()->route('dashboard.crud-5.index')->with('success', 'Data created successfully!');
        } catch (\Exception $e) {
            return back()->with('error', 'Something went wrong: ' . $e->getMessage());
        }
    }

    public function edit(string $id)
    {
        $crud5 = Crud5::findOrFail($id);
        return view('components.CRUD-5.edit', compact('crud5'));
    }

    public function update(Crud5Request $request, string $id)
    {
        $crud5 = Crud5::findOrFail($id);

        try {
            $validated = $request->validated();

            $img_path = $crud5->image;
            if ($request->hasFile('image')) {
                if ($crud5->image && file_exists(public_path($crud5->image))) {
                    unlink(public_path($crud5->image));
                }
                $image = $request->file('image');
                $filename = time() . '.' . $image->getClientOriginalExtension();
                $image->move(public_path('uploads/images/crud5'), $filename);
                $img_path = 'uploads/images/crud5/' . $filename;
            }

            $crud5->update([
                'name'  => $validated['name'],
                'email' => $validated['email'],
                'phone' => $validated['phone'],
                'image' => $img_path,
            ]);

            return redirect()->route('dashboard.crud-5.index')->with('success', 'Data updated successfully!');
        } catch (\Exception $e) {
            return back()->with('error', 'Something went wrong: ' . $e->getMessage());
        }
    }

    public function destroy(string $id)
    {
        try {
            $crud5 = Crud5::findOrFail($id);

            if ($crud5->image && file_exists(public_path($crud5->image))) {
                unlink(public_path($crud5->image));
            }
            $crud5->delete();
            return redirect()->route('dashboard.crud-5.index')->with('success', 'Data deleted successfully!');
        } catch (\Throwable $th) {
            return redirect()->back()->with('error', 'Something went wrong: ' . $th->getMessage());
        }
    }
}
```
______
## Step-5: (Form Request)

##### **`app\Http\Requests\Crud5Request.php:`**
```php
class Crud5Request extends FormRequest
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
            'name'  => 'required|string|max:255',
            'email' => 'required|email|unique:crud5s,email',
            'phone' => 'required|string|max:20|unique:crud5s,phone',
            'image' => 'required|image|mimes:jpeg,png,jpg,gif|max:2048',
        ];
    }
    protected function updateRules(): array
    {
        $crud5 = $this->route('crud5') ?? $this->route('crud_5') ?? $this->route('crud-5');
        $id = $crud5?->id ?? $crud5;

        return [
            'name'  => 'required|string|max:255',
            'email' => [
                'required',
                'email',
                Rule::unique('crud5s', 'email')->ignore($id),
            ],
            'phone' => [
                'required',
                'string',
                'max:20',
                Rule::unique('crud5s', 'phone')->ignore($id),
            ],
            'image' => 'nullable|image|mimes:jpeg,png,jpg,gif|max:2048',
        ];
    }
    public function messages(): array
    {
        return [
            'name.required'  => 'Please enter a name.',
            'email.required' => 'Email is required.',
            'email.email'    => 'Please enter a valid email address.',
            'email.unique'   => 'This email already exists.',
            'phone.required' => 'Phone number is required.',
            'phone.unique'   => 'This phone number already exists.',
            'image.required' => 'Please upload an image.',
            'image.image'    => 'File must be an image type.',
            'image.mimes'    => 'Image must be a jpeg, png, jpg, or gif format.',
            'image.max'      => 'Image size must not exceed 2MB.',
        ];
    }
}
```
____
## Step-6: (View Create)

##### `index.blade.php`:
```html
@extends('layouts.app')
@section('content')
@section('title', 'CRUD5 - List')
<div class="main-content">
	<div class="widget-header widget-header-flat" style="background-color: #618f8f;">
		<h4 class="widget-title" style="color: #fff;">CRUD5 List</h4>
		<span class="widget-toolbar">
		  <a href="{{ route('dashboard.crud-5.create') }}" style="color: #fff;">
			<i class="ace-icon fa fa-plus"></i> Create CRUD-5
		  </a>
		</span>
	</div>
<table id="dynamic-table" class="table table-striped table-bordered table-hover">
  <thead>
	<tr>
	  <th style="font-weight: bold;">Sl No</th>
	  <th>Name</th>
	  <th>Email</th>
	  <th>Phone</th>
	  <th>Image</th>
	  <th>Action</th>
	</tr>
  </thead>
  <tbody>
	@php
	$sl = $crud5->firstItem() ?? 0;
	@endphp
	@forelse ($crud5 as $item)
	<tr>
	  <td style="font-weight: bold;">{{ $sl++ }}.</td>
	  <td>{{ $item->name }}</td>
	  <td>{{ $item->email }}</td>
	  <td>{{ $item->phone }}</td>
	  <td>
		@if($item->image)
		<img src="{{ asset( $item->image) }}" alt="Image" width="60" height="60" 
			 style="object-fit: cover; border-radius: 5px;">
		@else
		<span class="text-muted">No Image</span>
		@endif
	  </td>
	  <td>
		<div class="hidden-sm hidden-xs action-buttons">
		  <a class="blue" href="#">
			<i class="ace-icon fa fa-eye bigger-130"></i>
		  </a>

		  <a class="green" href="{{ route('dashboard.crud-5.edit', $item->id) }}">
			<i class="ace-icon fa fa-pencil bigger-130"></i>
		  </a>
		  <form action="{{ route('dashboard.crud-5.destroy', $item->id) }}" method="POST"
			    style="display:inline;" onsubmit="return confirm('Are you Confirmed?');">
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
  {{ $crud5->links('pagination::bootstrap-4') }}
</div>
</div>
@endsection
```
_____
##### `create.blade.php`:
```html
@extends('layouts.app')
@section('content')
@section('title', 'CRUD5- Create')
<!-- Table is here -->
<div class="main-content">
	<!-- Main Form Start -->
	<form action="{{ route('dashboard.crud-5.store') }}" method="POST" enctype="multipart/form-data">
	  @csrf
	  <div class="form-group">
		<label for="name">Name</label>
		<input type="text" name="name" class="form-control" placeholder="Enter name" required>
	  </div>
	  <div class="form-group">
		<label for="phone">Phone No</label>
		<input type="number" name="phone" class="form-control" placeholder="+8801XXXXXXXXX" required>
	  </div>
	  <div class="form-group">
		<label for="email">Email</label>
		<input type="email" name="email" class="form-control" placeholder="example@email.com" required>
	  </div>
	  <div class="form-group">
		<label for="image">Image</label>
		<input type="file" name="image" class="form-control" required>
	  </div>
	  <div class="form-actions center">
		<button type="submit" class="btn btn-sm btn-success">
		  Save
		  <i class="ace-icon fa fa-save bigger-110"></i>
		</button>
		<a href="{{ route('dashboard.crud-5.index') }}" class="btn btn-sm btn-warning">
		  <i class="ace-icon fa fa-arrow-left bigger-110"></i> Back
		</a>
	  </div>
	</form>
	{{-- Form End --}}            
</div>
@endsection
```
______
##### `edit.blade.php`:
```html
@extends('layouts.app')
@section('content')
@section('title', 'Smart ERP - Edit Entry')
<!-- Edit form -->
<div class="main-content">
	<!-- Main Edit Form Start -->
	<form action="{{ route('dashboard.crud-5.update', $crud5->id) }}"
		  method="POST" enctype="multipart/form-data">
	  @csrf
	  @method('PUT')

	  <div class="form-group">
		<label for="name">Name</label>
		<input type="text" name="name" class="form-control"
			   value="{{ old('name', $crud5->name) }}" required>
	  </div>
	  <div class="form-group">
		<label for="phone">Phone No</label>
		<input type="text" name="phone" class="form-control"
			   value="{{ old('phone', $crud5->phone) }}" required>
	  </div>
	  <div class="form-group">
		<label for="email">Email</label>
		<input type="email" name="email" class="form-control"
			   value="{{ old('email', $crud5->email) }}" required>
	  </div>
	  <div class="form-group">
		<label for="image">Current Image</label><br>
		@if ($crud5->image)
		{{-- <img src="{{ asset('uploads/'.$crud5->image) }}" alt="Current Image" width="100"> --}}
		<img src="{{ asset($crud5->image) }}" alt="Current Image" width="100">
		@else
		<p>No image uploaded.</p>
		@endif
	  </div>
	  <div class="form-group">
		<label for="image">Change Image (optional)</label>
		<input type="file" name="image" class="form-control">
	  </div>
	  <div class="form-group">
		<label for="status">Status</label><br>
		{{-- <label>
		  <input type="radio" name="status" value="active" 
				 {{ $crud5->status == 'active' ? 'checked' : '' }}> Active
		</label>
		&nbsp;&nbsp;
		<label>
		  <input type="radio" name="status" value="inactive" 
			     {{ $crud5->status == 'inactive' ? 'checked' : '' }}> Inactive
		</label> --}}
	  </div>
	  <div class="form-actions center">
		<button type="submit" class="btn btn-sm btn-success">
		  Update
		  <i class="ace-icon fa fa-check icon-on-right bigger-110"></i>
		</button>
		<a href="{{ route('dashboard.crud-5.index') }}" class="btn btn-sm btn-warning">
		  <i class="ace-icon fa fa-arrow-left bigger-110"></i> Back
		</a>
	  </div>
	</form>
	<!-- Edit Form End -->        
</div>
@endsection
```
_____
