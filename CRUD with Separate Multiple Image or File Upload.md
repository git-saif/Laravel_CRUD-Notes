
Laravel -এ CRUD Operation এর জন্য ৫টি Step Follow করতে হয়। সেগুলো হলোঃ

1. Routes               =>  `routes/web.php`
2. Model                => `app\Models\Crud.php`
3. Migration          => `database\migrations\2025_04_14_164123_create_cruds_table.php`
4. Controller         => `app\Http\Controllers\CrudController.php`
5. Views                =>  `index.blade.php` , `create.blade.php` , `edit.blade.php`
   
যেহেতু এখানে আমরা Separately Multiple Image / File আপলোড করা দেখবো, সেজন্য এখানে extra কিছু কাজ করতে হবে। Multiple file গ্রহণের জন্য "Array" Use করবো। File গুলো আলাদাভাবে Input দেয়ার জন্য extra field যোগ করতে JavaScript ব্যবহার করবো। নিচে এগুলোর বিস্তারিত বর্ণনা দেয়া হলোঃ
## Step-1:

`routes/web.php`:
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

        'crud-2' => Crud2Controller::class,

    ]);
});
```
______

## Step-2:

`app\Models\Crud2.php`:
```php
class Crud2 extends Model
{
    use HasFactory;

    protected $guarded = [];


    // Accessor for image field
    public function getImageAttribute($value)
    {
        return json_decode($value, true); // Always return array
    }
}
```
_____

## Step-3:

`database\migrations\2025_04_22_153012_create_crud2s_table.php`:
```php
Schema::create('crud2s', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email');
            $table->string('phone');
            $table->text('image'); // পরিবর্তন: string → text (to store multiple image paths as JSON)
            $table->enum('status', ['active', 'inactive'])->default('active');
            $table->timestamps();
        });
```
___

## Step-4:

`app\Http\Controllers\Crud2Controller.php`:
```php
class Crud2Controller extends Controller
{
    
    public function index()
    {
        $crud2 = Crud2::orderby('id', 'asc')->paginate(3);
        return view('components.CRUD-2.index', compact('crud2'));
    }

    
    public function create()
    {
        return view('components.CRUD-2.create');
    }

    
    public function store(Request $request)
    {
        try {
            $validated = $request->validate([
                'name' => 'required|string|max:255',
                'email' => 'required|string|email|max:255',
                'phone' => 'required|string|max:255',
                'image' => 'required|array|min:1', // min 1 image required
                'image.*' => 'image|mimes:jpeg,png,jpg,gif|max:2048',
                'status' => 'required|in:active,inactive',
            ]);

            $image_paths = [];

            if ($request->hasFile('image')) {
                foreach ($request->file('image') as $image) {
                    if ($image->isValid()) {
                        $imgName = time() . '_' . uniqid() . '.' . $image->getClientOriginalExtension();
                        $image->move(public_path('uploads/images/crud2'), $imgName);
                        $image_paths[] = 'uploads/images/crud2/' . $imgName;
                    }
                }
            }

            // কমপক্ষে ১টি ইমেজ আপলোড হয়েছে কিনা চেক করা:
            if (empty($image_paths)) {
                return redirect()->back()->with('error', 'At least one valid image is required.');
            }

            Crud2::create([
                'name' => $validated['name'],
                'email' => $validated['email'],
                'phone' => $validated['phone'],
                'image' => json_encode($image_paths),
                'status' => $validated['status'],
            ]);

            return redirect()->route('dashboard.crud-2.index')->with('success', 'Data saved successfully!');
        } catch (\Exception $e) {
            return redirect()->back()->with('error', 'Something went wrong: ' . $e->getMessage());
        }
    }


    public function show(string $id)
    {
        //
    }

    public function edit(string $id)
    {
        $crud2 = Crud2::findOrFail($id);
        return view('components.CRUD-2.edit', compact('crud2'));
    }

    public function update(Request $request, string $id)
    {
        $crud2 = Crud2::findOrFail($id);

        try {
            $validated = $request->validate([
                'name' => 'required|string|max:255',
                'email' => 'required|string|email|max:255',
                'phone' => 'required|string|max:255',
                'existing_images' => 'sometimes|array',
                'existing_images.*' => 'sometimes|string',
                'delete_images' => 'sometimes|array',
                'delete_images.*' => 'sometimes|string',
                'replace_images' => 'sometimes|array',
                'replace_images.*' => 'sometimes|image|mimes:jpeg,png,jpg,gif|max:2048',
                'new_images' => 'sometimes|array',
                'new_images.*' => 'sometimes|image|mimes:jpeg,png,jpg,gif|max:2048',
                'status' => 'required|in:active,inactive',
            ]);

            // Handle existing images
            $existingImages = is_array($crud2->image) ? $crud2->image : (json_decode($crud2->image, true) ?? []);

            $updatedImages = [];

            // Process existing images
            foreach ($request->existing_images ?? [] as $index => $existingImage) {
                // Check if image is marked for deletion
                if (in_array($existingImage, $request->delete_images ?? [])) {
                    if (file_exists(public_path($existingImage))) {
                        unlink(public_path($existingImage));
                    }
                    continue;
                }

                // Check if image is being replaced
                if ($request->hasFile("replace_images.{$index}")) {
                    $newImage = $request->file("replace_images.{$index}");
                    $imgName = time() . '_' . $index . '_' . uniqid() . '.' . $newImage->getClientOriginalExtension();
                    $newImage->move(public_path('uploads/images/crud2'), $imgName);

                    // Delete old image
                    if (file_exists(public_path($existingImage))) {
                        unlink(public_path($existingImage));
                    }

                    $updatedImages[] = 'uploads/images/crud2/' . $imgName;
                } else {
                    $updatedImages[] = $existingImage;
                }
            }

            // Process new images
            if ($request->hasFile('new_images')) {
                foreach ($request->file('new_images') as $newImage) {
                    if ($newImage->isValid()) {
                        $imgName = time() . '_n_' . uniqid() . '.' . $newImage->getClientOriginalExtension();
                        $newImage->move(public_path('uploads/images/crud2'), $imgName);
                        $updatedImages[] = 'uploads/images/crud2/' . $imgName;
                    }
                }
            }

            // Update the record
            $crud2->update([
                'name' => $validated['name'],
                'email' => $validated['email'],
                'phone' => $validated['phone'],
                'image' => !empty($updatedImages) ? json_encode($updatedImages) : null,
                'status' => $validated['status'],
            ]);

            return redirect()->route('dashboard.crud-2.index')->with('success', 'Data updated successfully!');
        } catch (\Exception $e) {
            return redirect()->back()->with('error', 'Something went wrong: ' . $e->getMessage())->withInput();
        }
    }

    public function destroy(string $id)
    {
        try {
            $crud3 = Crud3::findOrFail($id);

            if ($crud3->image) {
                $images = json_decode($crud3->image, true) ?? [];
                foreach ($images as $imagePath) {
                    $imagePath = ltrim($imagePath, '/');
                    if (file_exists(public_path($imagePath))) {
                        unlink(public_path($imagePath));
                    }
                }
            }

            $crud3->delete();
            return redirect()->route('dashboard.crud-3.index')->with('success', 'Data deleted successfully!');
        } catch (\Exception $e) {
            return redirect()->back()->with('error', 'Something went wrong: ' . $e->getMessage());
        }
    }
}
```
____
## Step-5:

`index.blade.php`:
```html
@extends('layouts.app')

@section('content')
@section('title', 'Smart ERP - CRUD')
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
                <li class="active">Simple &amp; Dynamic</li>
            </ul><!-- /.breadcrumb -->

            <div class="nav-search" id="nav-search">
                <form class="form-search">
                    <span class="input-icon">
                        <input type="text" placeholder="Search ..." class="nav-search-input" id="nav-search-input"
                            autocomplete="off" />
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
                        Index of Table
                    </small>
                </h1>
            </div>

            <div class="row">
                <div class="col-xs-12">
                    <!-- PAGE CONTENT BEGINS -->



                    <div class="row">
                        <div class="col-xs-12">
                            <div class="clearfix">
                                <div class="pull-right tableTools-container"></div>
                            </div>
                            <div class="d-flex justify-content-between align-items-center ">
                                <div class="widget-header widget-header-flat" style="background-color: #618f8f;">
                                    <h4 class="widget-title " style="color: #fff;">CRUD List</h4>

                                    <span class="widget-toolbar">
                                        <a href="{{ route('dashboard.crud-2.create') }}" style="color: #fff;">
                                            <i class="ace-icon fa fa-plus"></i> Create Position
                                        </a>
                                    </span>
                                </div>

                            </div>

                            <!-- div.table-responsive -->

                            <!-- div.dataTables_borderWrap -->
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
                                            $sl = $crud2->firstItem() ?? 0;
                                        @endphp

                                        @forelse ($crud2 as $crud)
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
                                                            href="{{ route('dashboard.crud-2.edit', $crud->id) }}">
                                                            <i class="ace-icon fa fa-pencil bigger-130"></i>
                                                        </a>

                                                        <form
                                                            action="{{ route('dashboard.crud-2.destroy', $crud->id) }}"
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
                                    {{ $crud2->links('pagination::bootstrap-4') }}
                                </div>

                            </div>
                        </div>
                    </div>

                    <!-- PAGE CONTENT ENDS -->
                </div><!-- /.col -->
            </div>
        </div><!-- /.page-content -->
    </div>
</div>
<!-- /.main-content -->

<!-- /.main-container -->

@endsection
```

`create.blade.php`:
```html
@extends('layouts.app')

@section('content')
@section('title', 'Smart ERP - CRUD')
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
                <li class="active">Simple &amp; Dynamic</li>
            </ul><!-- /.breadcrumb -->

            <div class="nav-search" id="nav-search">
                <form class="form-search">
                    <span class="input-icon">
                        <input type="text" placeholder="Search ..." class="nav-search-input" id="nav-search-input"
                            autocomplete="off" />
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
                        Create New Table
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
                                <div class="d-flex justify-content-between align-items-center ">
                                    <div class="widget-header widget-header-flat " style="background-color: #618f8f;">
                                        <h4 class="widget-title" style="color: #fff;">Create Data</h4>

                                        <span class="widget-toolbar">
                                            <a href="{{ route('dashboard.crud-2.index') }}" style="color: #fff;">
                                                <i class="ace-icon fa fa-plus"></i> Go To Index
                                            </a>
                                        </span>
                                    </div>

                                </div>

                                <!-- div.table-responsive -->

                                <!-- div.dataTables_borderWrap -->
                                <form action="{{ route('dashboard.crud-2.store') }}" method="POST"
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
                                        <div class="form-group">
                                            <label for="image">Image (required)</label>
                                            <input type="file" name="image[]" class="form-control" required>
                                        </div>
                                    </div>

                                    <button type="button" id="add-image" class="btn btn-info">+ Add Another
                                        Image</button>

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

                                        <a href="{{ route('dashboard.crud.index') }}" class="btn btn-warning">
                                            <i class="ace-icon fa fa-arrow-left bigger-110"></i>
                                            Back
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
        let newInput = document.createElement('div');
        newInput.classList.add('form-group', 'mt-2');
        newInput.innerHTML = `<input type="file" name="image[]" class="form-control">`;
        document.getElementById('image-upload-wrapper').appendChild(newInput);
    });
</script>
@endpush
```

`edit.blade.php`:
```html
@extends('layouts.app')

@section('content')
@section('title', 'Smart ERP - Edit Entry')

<div class="main-content">
    <div class="main-content-inner">
        <div class="breadcrumbs ace-save-state" id="breadcrumbs">
            <ul class="breadcrumb">
                <li><i class="ace-icon fa fa-home home-icon"></i><a href="#">Home</a></li>
                <li><a href="#">Tables</a></li>
                <li class="active">Edit Entry</li>
            </ul>
        </div>

        <div class="page-content">
            <div class="page-header">
                <h1>
                    Edit Entry
                    <small><i class="ace-icon fa fa-angle-double-right"></i> Update Existing Data</small>
                </h1>
            </div>

            <div class="row">
                <div class="col-xs-12">
                    <div class="row">
                        <div class="col-xs-12">
                            <div class="col-md-2"></div>

                            <div class="col-md-8">
                                <div class="d-flex justify-content-between align-items-center">
                                    <div class="table-header d-flex justify-content-between align-items-stretch">
                                        <span>Edit Entry</span>
                                        <a href="{{ route('dashboard.crud-2.index') }}"
                                            class="pull-right btn btn-sm btn-white h-100 d-flex align-items-center"
                                            style="margin-left: auto;">
                                            <i class="fa fa-list me-1"></i> Back to List
                                        </a>
                                    </div>
                                </div>

                                <!-- Edit Form Start -->
                                <form action="{{ route('dashboard.crud-2.update', $crud2->id) }}" method="POST"
                                    enctype="multipart/form-data">
                                    @csrf
                                    @method('PUT')

                                    <div class="form-group">
                                        <label for="name">Name</label>
                                        <input type="text" name="name" class="form-control"
                                            value="{{ $crud2->name }}" required>
                                    </div>

                                    <div class="form-group">
                                        <label for="phone">Phone No</label>
                                        <input type="text" name="phone" class="form-control"
                                            value="{{ $crud2->phone }}" required>
                                    </div>

                                    <div class="form-group">
                                        <label for="email">Email</label>
                                        <input type="email" name="email" class="form-control"
                                            value="{{ $crud2->email }}" required>
                                    </div>

                                    <!-- Current Images Section -->
                                    <div class="form-group">
                                        <label>Current Images</label><br>
                                        @php
                                            $existingImages = is_array($crud2->image)
                                                ? $crud2->image
                                                : json_decode($crud2->image, true) ?? [];
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
                                                {{ $crud2->status == 'active' ? 'checked' : '' }}> Active
                                        </label>
                                        <label>
                                            <input type="radio" name="status" value="inactive"
                                                {{ $crud2->status == 'inactive' ? 'checked' : '' }}> Inactive
                                        </label>
                                    </div>

                                    <div class="form-actions center">
                                        <button type="submit" class="btn btn-primary">
                                            <i class="ace-icon fa fa-save bigger-110"></i>
                                            Update
                                        </button>

                                        <a href="{{ route('dashboard.crud.index') }}" class="btn btn-warning">
                                            <i class="ace-icon fa fa-arrow-left bigger-110"></i>
                                            Back
                                        </a>
                                    </div>
                                </form>
                                <!-- Edit Form End -->
                            </div>
                            <div class="col-md-2"></div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

@endsection

@push('scripts')  // এর জন্য master blade -এ গিয়ে " @stack('scripts') " লিখতে হবে।
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

