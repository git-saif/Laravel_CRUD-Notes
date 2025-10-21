Laravel -এ CRUD with Image / File Operation এর জন্যও ৫টি Step Follow করতে হয়। সেগুলো হলোঃ

1. Routes               =>  `routes\web.php`
2. Model                => `app\Models\Crud2.php`
3. Migration          => `database\migrations\2025_04_14_164123_create_crud2s_table.php`
4. Controller         => `app\Http\Controllers\Crud2Controller.php`
5. Views                =>  `index.blade.php` , `create.blade.php` , `edit.blade.php`

 তবে এখানে Image / File গুলোকে আলাদাভাবে Handling করতে হয়। নিচে এটি বিস্তারিত বর্ণনা করা হলোঃ  
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
| Resource Routes.
|--------------------------------------------------------------------------
*/
Route::group(['prefix' => 'dashboard', 'as' => 'dashboard.'], function () {
    Route::resources([
        'crud-2' => CrudController::class,
    ]);
});
```
_______
## Step-2: (Model)

1. Create Model & Migration by 1 command line:
```bash
php artisan make:model Crud2 -m
```

2. Update Code
##### `app\Models\Crud2.php`: (Model)
```php
class Crud extends Model
{
    use HasFactory;
    
    protected $guarded = [];
}
```
_____
## Step-3: (Migration)

 ##### `database\migrations\2025_04_14_164123_create_crud2s_table.php`:
```php
	Schema::create('crud2s', function (Blueprint $table) {
		$table->id();
		$table->string('name');
		$table->string('email');
		$table->string('phone');
		$table->string('image');
		$table->enum('status', ['active', 'inactive'])->default('active');
		$table->timestamps();
	});
```
____
## Step-4: (Controller)

##### `app\Http\Controllers\Crud2Controller.php`: (Controller)
```php
class CrudController extends Controller
{
    public function index()
    {
        $crudData = Crud::orderBy('id', 'asc')->paginate(3);
        return view('components.CRUD.index', compact('crudData'));
    }

    public function create()
    {
        return view('components.CRUD.create');
    }

    public function store(Request $request)
    {
        try {
            $validated = $request->validate([
                'name'   => 'required|string|max:255',
                'email'  => 'required|string|email|max:255',
                'phone'  => 'required|string|max:255',
                'image'  => 'required|image|mimes:jpeg,png,jpg,gif',
                'status' => 'required|in:active,inactive',
            ]);

            $img_path = null;
            if ($request->hasFile('image')) {
                $image = $request->file('image');
                $filename = time() . '.' . $image->getClientOriginalExtension();
                $image->move(public_path('uploads/images/crud'), $filename);
                $img_path = 'uploads/images/crud/' . $filename; 
            }


            Crud::create([
                'name'   => $validated['name'],
                'email'  => $validated['email'],
                'phone'  => $validated['phone'],
                'image'  => $img_path,
                'status' => $validated['status'],
            ]);


            return redirect()
	            ->route('dashboard.crud.index')
	            ->with('success', 'Data created successfully!');
        } catch (\Exception $e) {
            return redirect()
	            ->back()
	            ->with('error', 'Something went wrong: ' . $e->getMessage());
        }

    }

    public function show(string $id)
    {
        //
    }

    public function edit(string $id)
    {
        $crud = Crud::findOrFail($id);
        return view('components.CRUD.edit', compact('crud'));
    }

    public function update(Request $request, string $id)
    {
        $crud = Crud::findOrFail($id);

        try {
            $validated = $request->validate([
                'name'   => 'required|string|max:255',
                'email'  => 'required|string|email|max:255',
                'phone'  => 'required|string|max:255',
                'image'  => 'nullable|image|mimes:jpeg,png,jpg,gif',
                'status' => 'required|in:active,inactive',
            ]);

            $img_path = $crud->image;
            if ($request->hasFile('image')) {
                if ($crud->image && file_exists(public_path($crud->image))) {
                    unlink(public_path($crud->image));
                }
                $image = $request->file('image');
                $filename = time() . '.' . $image->getClientOriginalExtension();
                $image->move(public_path('uploads/images/crud'), $filename);
                $img_path = 'uploads/images/crud/' . $filename; 
            }

            $crud->update([
                'name'   => $validated['name'],
                'email'  => $validated['email'],
                'phone'  => $validated['phone'],
                'image'  => $img_path,
                'status' => $validated['status'],
            ]);

            return redirect()
	            ->route('dashboard.crud.index')
	            ->with('success', 'Data updated successfully!');
        } catch (\Exception $e) {
            return redirect()
	            ->back()
	            ->with('error', 'Something went wrong: ' . $e->getMessage());
        }
    }

    public function destroy(string $id)
    {
        try {
            $crud = Crud::findOrFail($id);

            if ($crud->image && file_exists(public_path($crud->image))) {
                unlink(public_path($crud->image));
            }

            $crud->delete();
            return redirect()
	            ->route('dashboard.crud.index')
	            ->with('success', 'Data deleted successfully!');
        } catch (\Throwable $th) {
            return redirect()
	            ->back()
	            ->with('error', 'Something went wrong: ' . $th->getMessage());
        }
    }
}
```
______
## Step-5: (Create View)

##### `index.blade.php`:
```html
@extends('layouts.app')
@section('content')
@section('title', 'Smart ERP - CRUD')
<!-- Main Table is here -->
<div>
	<table id="dynamic-table" class="table table-striped table-bordered table-hover">
		<thead>
			<tr>
				<th style="font-weight: bold;"> Sl No </th>
				<th> Name </th>
				<th> Pnone No </th>
				<th> Email </th>
				<th> Image </th>
				<th> Status </th>
				<th> Action </th>
			</tr>
		</thead>
		<tbody>
			@php
				$sl = $crudData->firstItem() ?? 0;
			@endphp
			
			@forelse ($crudData as  $item)
				<tr>
					<td style="font-weight: bold;"> {{ $sl++ }}. </td>
					<td> {{ $item->name }} </td>
					<td> {{ $item->phone }} </td>
					<td> {{ $item->email }} </td>
					<td>
						<img src="{{ asset($item->image) ?? '' }}" alt="Image"
							width="100">
					</td>
					<td>
						@if ($item->status == 'active')
							<span
								class="label label-sm label-success arrowed-in">Active</span>
						@else
							<span
								class="label label-sm label-danger arrowed-in">Inactive</span>
						@endif
					</td>
					<td>
						<div class="hidden-sm hidden-xs action-buttons">
							<a class="blue" href="#">
								<i class="ace-icon fa fa-eye bigger-130"></i>
							</a>
							<a class="green"
								href="{{ route('dashboard.crud.edit', $item->id) }}">
								<i class="ace-icon fa fa-pencil bigger-130"></i>
							</a>
							<form action="{{ route('dashboard.crud.destroy', $item->id) }}"
								method="POST" style="display:inline;"
								onsubmit="return confirm('Are you sure you want to delete this item?');">
								@csrf
								@method('DELETE')
								<button type="submit" class="red"
									style="border: none; background: none;">
									<i class="ace-icon fa fa-trash-o bigger-130"></i>
								</button>
							</form>

						</div>
					</td>
				</tr>
			@empty
				<tr>
					<td colspan="7" class="text-center text-danger">No data found.</td>
				</tr>
			@endforelse
		</tbody>
	</table>
	<div class="text-center">
		{{ $crudData->links('pagination::bootstrap-4') }}
	</div>
</div>
@endsection
```
____
##### `create.blade.php`:
```html
@extends('layouts.app')
@section('content')
@section('title', 'Smart ERP - CRUD')
<div class="main-content">
	<!-- Main Form Start -->
	<form action="{{ route('dashboard.crud.store') }}" method="POST"
		enctype="multipart/form-data">
		@csrf
	
		<div class="form-group">
			<label for="name">Name</label>
			<input type="text" name="name" class="form-control"
				placeholder="Enter name" required>
		</div>
		<div class="form-group">
			<label for="phone">Phone No</label>
			<input type="number" name="phone" class="form-control"
				placeholder="+8801XXXXXXXXX" required>
		</div>
		<div class="form-group">
			<label for="email">Email</label>
			<input type="email" name="email" class="form-control"
				placeholder="example@email.com" required>
		</div>
		<div class="form-group">
			<label for="image">Image</label>
			<input type="file" name="image" class="form-control" required>
		</div>
		<div class="form-group">
			<label for="status">Status</label><br>
			<label>
				<input type="radio" name="status" value="active" checked> Active
			</label>
			&nbsp;&nbsp;
			<label>
				<input type="radio" name="status" value="inactive"> Inactive
			</label>
		</div>
		<div class="form-actions center">
			<button type="submit" class="btn btn-sm btn-success">
				Save
				<i class="ace-icon fa fa-save bigger-110"></i>
			</button>
	
			<a href="{{ route('dashboard.crud.index') }}" class="btn btn-sm btn-warning">
				<i class="ace-icon fa fa-arrow-left bigger-110"></i> Back
			</a>
		</div>
	</form>
	<!-- Main From End -->
</div>
@endsection
```
_______
##### `edit.blade.php`:
```html
@extends('layouts.app')
@section('content')
@section('title', 'Smart ERP - Edit Entry')
<!-- Edit form -->
<div class="main-content">
	<form action="{{ route('dashboard.crud.update', $crud->id) }}" 
		  method="POST" enctype="multipart/form-data">
		@csrf
		@method('PUT')

		<div class="form-group">
			<label for="name">Name</label>
			<input type="text" name="name" class="form-control"value="{{ old('name', $crud->name) }}" required>
		</div>
		<div class="form-group">
			<label for="phone">Phone No</label>
			<input type="text" name="phone" class="form-control"
				value="{{ old('phone', $crud->phone) }}" required>
		</div>
		<div class="form-group">
			<label for="email">Email</label>
			<input type="email" name="email" class="form-control"
				value="{{ old('email', $crud->email) }}" required>
		</div>
		<div class="form-group">
			<label for="image">Current Image</label><br>
			@if ($crud->image)
				{{-- <img src="{{ asset('uploads/'.$crud->image) }}" 
						  alt="Current Image" width="100"> --}}
				<img src="{{ asset($crud->image) }}" alt="Current Image" width="100">
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
			<label>
				<input type="radio" name="status" value="active"
					{{ $crud->status == 'active' ? 'checked' : '' }}> Active
			</label>
			{{-- &nbsp;&nbsp; --}}
			<label>
				<input type="radio" name="status" value="inactive"
					{{ $crud->status == 'inactive' ? 'checked' : '' }}> Inactive
			</label>
		</div>
		<div class="form-actions center">
			<button type="submit" class="btn btn-sm btn-success">
				Update
				<i class="ace-icon fa fa-check icon-on-right bigger-110"></i>
			</button>
			<a href="{{ route('dashboard.crud.index') }}" 
			   class="btn btn-sm btn-warning">
				<i class="ace-icon fa fa-arrow-left bigger-110"></i> Back
			</a>
		</div>
	</form>
	<!-- Edit Form End -->
</div>
@endsection
```
-----
