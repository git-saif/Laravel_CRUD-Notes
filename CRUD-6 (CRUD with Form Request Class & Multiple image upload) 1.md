
Laravel -এ CRUD Operation এর জন্য 6টি Step Follow করতে হয়। সেগুলো হলোঃ

1. Routes               =>  `routes\web.php`
2. Model                => `app\Models\Crud6.php`
3. Migration          => `database\migrations\2025_04_14_164123_create_crud6s_table.php`
4. Controller         => `app\Http\Controllers\Crud6Controller.php`
5. Request form   =>  `app\Http\Requests\Crud6Request.php`
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
        // CRUD With Advance Validation 
        'crud-6' => Crud6Controller::class,
    ]);
});
```
_____
## Step-2: (Model)

##### `app\Models\Crud6.php`:
```php
class Crud6 extends Model
{
    use HasFactory;

    protected $fillable = [
        'name',
        'email',
        'phone',
        'image',
        'status',
    ];

    // Image field accessor (always return array)
    public function getImageAttribute($value)
    {
        return json_decode($value, true) ?? [];
    }
}
```
____
## Step-3: (Migration)

##### `database\migrations\2025_05_05_100447_create_crud6s_table.php`:
```php
	Schema::create('crud6s', function (Blueprint $table) {
		$table->id();
		$table->string('name');
		$table->string('email');
		$table->string('phone');
		$table->json('image');    // For Accept Multiple Image
		$table->enum('status', ['active', 'inactive'])->default('active');
		$table->timestamps();
	});
```
_____
## Step-4: (Controller)

##### `app\Http\Controllers\Crud6Controller.php`:
```php
class Crud6Controller extends Controller
{
    public function index()
    {
        $crud6 = Crud6::orderby('id', 'asc')->paginate(3);
        return view('components.CRUD-6.index', compact('crud6'));
    }

    public function create()
    {
        return view('components.CRUD-6.create');
    }

    public function store(Crud6Request $request)
    {
        $imagePaths = [];

        if ($request->hasFile('image')) {
            foreach ($request->file('image') as $image) {
                $imgName = time() . '_' . uniqid() . '.' . $image->getClientOriginalExtension();
                $image->move(public_path('uploads/images/crud6'), $imgName);
                $imagePaths[] = 'uploads/images/crud6/' . $imgName;
            }
        }

        Crud6::create([
            'name'   => $request->name,
            'email'  => $request->email,
            'phone'  => $request->phone,
            'image'  => json_encode($imagePaths),
            'status' => $request->status,
        ]);

        return redirect()->route('dashboard.crud-6.index')->with('success', 'Data saved successfully!');
    }

    public function show(string $id)
    {
        //
    }

    public function edit(string $id)
    {
        $crud6 = Crud6::findOrFail($id);
        return view('components.CRUD-6.edit', compact('crud6'));
    }

    public function update(Crud6Request $request, string $id)
    {
        $crud6 = Crud6::findOrFail($id);

        try {
            $existingImages = is_array($crud6->image) ? $crud6->image : (json_decode($crud6->image, true) ?? []);
            $updatedImages = [];

            // Existing images process
            foreach ($request->existing_images ?? [] as $index => $existingImage) {
                if (in_array($existingImage, $request->delete_images ?? [])) {
                    if (file_exists(public_path($existingImage))) unlink(public_path($existingImage));
                    continue;
                }

                if ($request->hasFile("replace_images.$index")) {
                    $newImage = $request->file("replace_images.$index");
                    $imgName = time() . "_r_" . uniqid() . '.' . $newImage->getClientOriginalExtension();
                    $newImage->move(public_path('uploads/images/crud6'), $imgName);

                    if (file_exists(public_path($existingImage))) unlink(public_path($existingImage));
                    $updatedImages[] = 'uploads/images/crud6/' . $imgName;
                } else {
                    $updatedImages[] = $existingImage;
                }
            }

            // New images process
            if ($request->hasFile('new_images')) {
                foreach ($request->file('new_images') as $newImage) {
                    if ($newImage->isValid()) {
                        $imgName = time() . "_n_" . uniqid() . '.' . $newImage->getClientOriginalExtension();
                        $newImage->move(public_path('uploads/images/crud6'), $imgName);
                        $updatedImages[] = 'uploads/images/crud6/' . $imgName;
                    }
                }
            }

            $crud6->update([
                'name' => $request->name,
                'email' => $request->email,
                'phone' => $request->phone,
                'image' => json_encode($updatedImages),
                'status' => $request->status
            ]);

            return redirect()->route('dashboard.crud-6.index')->with('success', 'Data updated successfully!');
        } catch (\Exception $e) {
            return back()->with('error', 'Something went wrong: ' . $e->getMessage());
        }
    }

    public function destroy(string $id)
    {
        try {
            $crud6 = Crud6::findOrFail($id);

            if ($crud6->image) {
                $images = json_decode($crud6->image, true) ?? [];
                foreach ($images as $imagePath) {
                    if (file_exists(public_path($imagePath))) unlink(public_path($imagePath));
                }
            }

            $crud6->delete();
            return redirect()->route('dashboard.crud-6.index')->with('success', 'Data deleted successfully!');
        } catch (\Exception $e) {
            return back()->with('error', 'Something went wrong: ' . $e->getMessage());
        }
    }
}
```
______
## Step-5: (Form Request)

##### **`app\Http\Requests\Crud6Request.php:`**
```php
class Crud6Request extends FormRequest
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
            'name'   => 'required|string|max:255',
            'email'  => 'required|email|unique:crud6s,email',
            'phone'  => 'required|string|max:20|unique:crud6s,phone',
            'image'  => 'required|array|min:1',
            'image.*' => 'image|mimes:jpeg,png,jpg,gif|max:2048',
            'status' => 'required|in:active,inactive',
        ];
    }

    protected function updateRules(): array
    {
        $crud6 = $this->route('crud6') ?? $this->route('crud_6') ?? $this->route('crud-6');
        $id = $crud6?->id ?? $crud6;

        return [
            'name'   => 'required|string|max:255',
            'email'  => ['required', 'email', Rule::unique('crud6s', 'email')->ignore($id)],
            'phone'  => ['required', 'string', 'max:20', Rule::unique('crud6s', 'phone')->ignore($id)],
            'existing_images' => 'sometimes|array',
            'existing_images.*' => 'sometimes|string',
            'delete_images' => 'sometimes|array',
            'delete_images.*' => 'sometimes|string',
            'replace_images' => 'sometimes|array',
            'replace_images.*' => 'sometimes|image|mimes:jpeg,png,jpg,gif|max:2048',
            'new_images' => 'sometimes|array',
            'new_images.*' => 'sometimes|image|mimes:jpeg,png,jpg,gif|max:2048',
            'status' => 'required|in:active,inactive',
        ];
    }

    public function messages(): array
    {
        return [
            'name.required' => 'Please enter a name.',
            'email.required' => 'Email is required.',
            'email.unique' => 'This email already exists.',
            'phone.required' => 'Phone number is required.',
            'phone.unique' => 'This phone number already exists.',
            'image.required' => 'Please upload at least one image.',
            'image.*.image' => 'Each file must be an image.',
            'image.*.mimes' => 'Allowed image formats: jpeg, png, jpg, gif.',
            'image.*.max' => 'Image must not exceed 2MB.',
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
@section('title', 'CRUD6 - List')
<!-- Table is here -->
<div class="main-content">
    <!-- Maint Table is Here -->
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
					$sl = $crud6->firstItem() ?? 0;
				@endphp
	
				@forelse ($crud6 as $crud)
					<tr>
						<td style="font-weight: bold;"> {{ $sl++ }}. </td>
						<td> {{ $crud->name }} </td>
						<td> {{ $crud->phone }} </td>
						<td> {{ $crud->email }} </td>
						{{-- <td>
							@if ($crud->image && is_array($crud->image))
								@foreach ($crud->image as $img)
									<img src="{{ asset($img) }}" width="100" class="mb-1"
										alt="">
								@endforeach
							@else
								<span>No image</span>
							@endif
						</td> --}}
	
						<td>
							@if ($crud->image)
								@foreach ($crud->image as $img)
									<img src="{{ asset($img) }}" width="100" height="60" class="mb-1"
										alt="">
								@endforeach
							@else
								<span>No image</span>
							@endif
						</td>
						<td>
							@if ($crud->status == 'active')
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
									href="{{ route('dashboard.crud-6.edit', $crud->id) }}">
									<i class="ace-icon fa fa-pencil bigger-130"></i>
								</a>
	
								<form
									action="{{ route('dashboard.crud-6.destroy', $crud->id) }}"
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
			{{ $crud6->links('pagination::bootstrap-4') }}
		</div>
	</div>
</div><!-- /.main-content -->
@endsection
```
_____
##### `create.blade.php`:
```html
@extends('layouts.app')
@section('content')
@section('title', 'CRUD6 - Create')
<!-- Table is here -->
<div class="main-content">
	<!-- div.dataTables_borderWrap -->
	<form action="{{ route('dashboard.crud-6.store') }}" method="POST"
		enctype="multipart/form-data">
		@csrf

		<div class="form-group">
			<label for="name">Name</label>
			<input type="text" name="name" class="form-control"
				placeholder="Enter name" required>
		</div>

		<div class="form-group">
			<label for="phone">Phone No</label>
			<input type="text" name="phone" class="form-control"
				placeholder="+8801XXXXXXXXX" required>
		</div>

		<div class="form-group">
			<label for="email">Email</label>
			<input type="email" name="email" class="form-control"
				placeholder="example@email.com" required>
		</div>

		{{-- Multiple Image Start --}}
		<div id="image-upload-wrapper">
		  <div class="form-group d-flex align-items-center mb-2">
			<label for="image" class="me-2">Image (required)</label>
			<input type="file" name="image[]" class="form-control me-2" required>
		  </div>
		</div>

		<button type="button" id="add-image" class="btn btn-info mt-2">+ Add Another Image</button>
		{{-- Multiple Image End --}}

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
			<button type="submit" class="btn btn-success">
				<i class="ace-icon fa fa-check bigger-110"></i>
				Submit
			</button>

			<a href="{{ route('dashboard.crud-6.index') }}" class="btn btn-warning">
				<i class="ace-icon fa fa-arrow-left bigger-110"></i>
				Back
			</a>
		</div>
	</form>
	{{-- End Main Form --}}
</div><!-- /.main-content -->

@if (session('error'))
    <div class="alert alert-danger">
        {{ session('error') }}
    </div>
@endif

@if (session('success'))
    <div class="alert alert-success">
        {{ session('success') }}
    </div>
@endif

@endsection
@push('scripts')
<script>
  document.getElementById('add-image').addEventListener('click', function() {
    let wrapper = document.getElementById('image-upload-wrapper');

    // নতুন div তৈরি করা হবে flexbox সহ
    let newInput = document.createElement('div');
    newInput.classList.add('form-group', 'd-flex', 'align-items-center', 'mt-2');

    // innerHTML সহ delete button
    newInput.innerHTML = `
        <input type="file" name="image[]" class="form-control me-2" style="flex:1;">
        <button type="button" class="btn btn-danger btn-sm remove-image">Delete</button>
    `;

    wrapper.appendChild(newInput);

    // Delete button এ click listener attach
    newInput.querySelector('.remove-image').addEventListener('click', function() {
      wrapper.removeChild(newInput);
    });
  });
</script>
@endpush

```
______
##### `edit.blade.php`:
```html
@extends('layouts.app')
@section('content')
@section('title', 'CRUD6 - Edit Entry')
<div class="main-content">
	<!-- Edit Form Start -->
	<form action="{{ route('dashboard.crud-6.update', $crud6->id) }}" method="POST"
		enctype="multipart/form-data">
		@csrf
		@method('PUT')

		<div class="form-group">
			<label for="name">Name</label>
			<input type="text" name="name" class="form-control"
				value="{{ $crud6->name }}" required>
		</div>

		<div class="form-group">
			<label for="phone">Phone No</label>
			<input type="text" name="phone" class="form-control"
				value="{{ $crud6->phone }}" required>
		</div>

		<div class="form-group">
			<label for="email">Email</label>
			<input type="email" name="email" class="form-control"
				value="{{ $crud6->email }}" required>
		</div>

		<!-- Current Images Section -->
		<div class="form-group">
			<label>Current Images</label><br>
			@php
				$existingImages = is_array($crud6->image)
					? $crud6->image
					: json_decode($crud6->image, true) ?? [];
			@endphp

			@if (count($existingImages) > 0)
				<div class="row mb-3">
					@foreach ($existingImages as $index => $image)
						<div class="col-md-4 mb-3">
							<div class="card">
								<img src="{{ asset($image) }}" class="card-img-top"
									style="height: 150px; object-fit: cover;">
								<div class="card-body p-2">
									<div class="form-check">
										<input class="form-check-input" type="checkbox"
											name="delete_images[]"
											value="{{ $image }}"
											id="delete_img_{{ $index }}">
										<label class="form-check-label"
											for="delete_img_{{ $index }}">
											Delete
										</label>
									</div>
									<div class="mt-2">
										<label class="btn btn-sm btn-outline-primary w-100">
											Change Image
											<input type="file"
												name="replace_images[{{ $index }}]"
												class="d-none replace-image"
												data-index="{{ $index }}">
										</label>
									</div>
									<input type="hidden" name="existing_images[]"
										value="{{ $image }}">
								</div>
							</div>
						</div>
					@endforeach
				</div>
			@else
				<p>No images uploaded.</p>
			@endif
		</div>

		<!-- New Images Section -->
		<div class="form-group">
			<label>Add New Images</label>
			<div id="new-images-container">
				<div class="input-group mb-2">
					<input type="file" name="new_images[]" class="form-control">
					<button type="button" class="btn btn-danger remove-new-image">× Remove
						Field</button>
				</div>
			</div>
			<button type="button" id="add-new-image" class="btn btn-sm btn-primary mt-2">
				<i class="fa fa-plus"></i> Add More Images
			</button>
		</div>

		<div class="form-group">
			<label for="status">Status</label><br>
			<label>
				<input type="radio" name="status" value="active"
					{{ $crud6->status == 'active' ? 'checked' : '' }}> Active
			</label>
			<label>
				<input type="radio" name="status" value="inactive"
					{{ $crud6->status == 'inactive' ? 'checked' : '' }}> Inactive
			</label>
		</div>

		<div class="form-actions center">
		  <button type="submit" class="btn btn-sm btn-success">
			Update
			<i class="ace-icon fa fa-check icon-on-right bigger-110"></i>
		  </button>

		  <a href="{{ route('dashboard.crud-6.index') }}" class="btn btn-sm btn-warning">
			<i class="ace-icon fa fa-arrow-left bigger-110"></i> Back
		  </a>
		</div>
	</form>
	<!-- Edit Form End -->
</div>
</div>
@endsection
@push('scripts')
<script>
    // Add new image field
    $('#add-new-image').click(function() {
        $('#new-images-container').append(`
            <div class="input-group mb-2">
                <input type="file" name="new_images[]" class="form-control">
                <button type="button" class="btn btn-danger remove-new-image">× Remove Field</button>
            </div>
        `);
    });

    // Remove new image field
    $(document).on('click', '.remove-new-image', function() {
        $(this).closest('.input-group').remove();
    });

    // Preview image when replacing
    $(document).on('change', '.replace-image', function(e) {
        const index = $(this).data('index');
        const file = e.target.files[0];
        if (file) {
            const reader = new FileReader();
            reader.onload = function(event) {
                $(`#delete_img_${index}`).closest('.card').find('img').attr('src', event.target.result);
            }
            reader.readAsDataURL(file);
        }
    });
</script>
@endpush
```
_____
