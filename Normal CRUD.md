
Laravel -এ CRUD Operation এর জন্য ৫টি Step Follow করতে হয়। সেগুলো হলোঃ

1. Routes               =>  `routes/web.php`
2. Model                => `app\Models\Crud.php`
3. Migration          => `database\migrations\2025_04_14_164123_create_cruds_table.php`
4. Controller         => `app\Http\Controllers\CrudController.php`
5. Views                =>  `index.blade.php` , `create.blade.php` , `edit.blade.php`
   
নিচে এগুলোর বিস্তারিত বর্ণনা দেয়া হলোঃ

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

        'crud-0' => Crud0Controller::class, // we use this for normal CRUD.
        'crud' => CrudController::class,
        'crud-2' => Crud2Controller::class,

    ]);
});
```
_____
## Step-2:

`app\Models\Crud0.php`:
```php
class Crud0 extends Model
{
    use HasFactory;

    protected $guarded = [];
}
```
____
## Step-3:

`database\migrations\2025_05_05_100447_create_crud0s_table.php`:
```php
Schema::create('crud0s', function (Blueprint $table) {
            $table->id();
            $table->string('topic_name');
            $table->string('title');
            $table->string('description');
            $table->enum('status', ['active', 'inactive'])->default('active');
            $table->timestamps();
        });
```
_____
## Step-4:

`app\Http\Controllers\Crud0Controller.php`:
```php
class Crud0Controller extends Controller
{
    public function index()
    {
        $crud0 = Crud0::orderby('id', 'asc')->paginate(3);
        return view('components.crud0.index', compact('crud0'));
    }
    
    public function create()
    {
        return view('components.crud0.create');
    }

    public function store(Request $request)
    {
        try {
            $validated = $request->validate([
                'topic_name' => 'required',
                'title' => 'required',
                'description' => 'required',
                'status' => 'required',
            ]);
            
            Crud0::create($validated);
            
            return redirect()->route('dashboard.crud-0.index')->with('success', 'Data successfully stored.');


        } catch (\Throwable $e) {
            return redirect()->back()->with('error', 'Something went wrong: ' . $e->getMessage());
        }
    }

    public function show(string $id)
    {
        //
    }

    public function edit(string $id)
    {
        $crud0 = Crud0::findOrFail($id);
        return view('components.crud0.edit', compact('crud0'));
    }

    public function update(Request $request, string $id)
    {
        $crud0 = Crud0::findOrFail($id);
        
        try {
            $validated = $request->validate([
                'topic_name' => 'required',
                'title' => 'required',
                'description' => 'required',
                'status' => 'required',
            ]);

            $crud0->update($validated);
            return redirect()->route('dashboard.crud-0.index')->with('success', 'Data successfully updated.');

        } catch (\Throwable $th) {
            //throw $th;
        }
    }

    public function destroy(string $id)
    {
        try {
            $crud0 = Crud0::findOrFail($id);
            $crud0->delete();
            return redirect()->route('dashboard.crud-0.index')->with('success', 'Data successfully deleted.');
        } catch (\Throwable $th) {
            return redirect()->back()->with('error', 'Something went wrong: ' . $th->getMessage());
        }
    }
}
```
______
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
                            <div class="widget-header widget-header-flat" style="background-color: #618f8f;">
                                <h4 class="widget-title " style="color: #fff;">CRUD List</h4>

                                <span class="widget-toolbar">
                                    <a href="{{ route('dashboard.crud-0.create') }}" style="color: #fff;">
                                        <i class="ace-icon fa fa-plus"></i> Create CRUD - 0
                                    </a>
                                </span>
                            </div>


                            <!-- div.table-responsive -->

                            <!-- div.dataTables_borderWrap -->
                            <div>
                                <table id="dynamic-table" class="table table-striped table-bordered table-hover">
                                    <thead>
                                        <tr>

                                            <th style="font-weight: bold;"> Sl No </th>
                                            <th> Topic </th>
                                            <th> Title </th>
                                            <th> Description </th>
                                            <th> Status </th>
                                            <th> Action </th>
                                        </tr>
                                    </thead>

                                    <tbody>

                                        @php
                                            $sl = $crud0->firstItem() ?? 0;
                                        @endphp



                                        @forelse ($crud0 as  $item)
                                            <tr>
                                                <td style="font-weight: bold;"> {{ $sl++ }}. </td>
                                                <td> {{ $item->topic_name ?? '' }} </td>
                                                <td> {{ $item->title ?? '' }} </td>
                                                <td> {{ $item->description ?? '' }} </td>
                                                
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
                                                            href="{{ route('dashboard.crud-0.edit', $item->id) }}">
                                                            <i class="ace-icon fa fa-pencil bigger-130"></i>
                                                        </a>

                                                        <form action="{{ route('dashboard.crud-0.destroy', $item->id) }}"
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
                                    {{ $crud0->links('pagination::bootstrap-4') }}
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
                                <div class="widget-header widget-header-flat " style="background-color: #618f8f;">
                                    <h4 class="widget-title" style="color: #fff;">Create Data</h4>

                                    <span class="widget-toolbar">
                                        <a href="{{ route('dashboard.crud.index') }}" style="color: #fff;">
                                            <i class="ace-icon fa fa-plus"></i> Go To Index
                                        </a>
                                    </span>
                                </div>

                                <!-- div.table-responsive -->

                                <!-- div.dataTables_borderWrap -->
                                <form action="{{ route('dashboard.crud-0.store') }}" method="POST"
                                    enctype="multipart/form-data">
                                    @csrf

                                    <div class="form-group">
                                        <label for="topic_name">Topic Name:</label>
                                        <input type="text" name="topic_name" class="form-control"
                                            placeholder="Enter Topic Name Here" required>
                                    </div>

                                    
                                    <div class="form-group">
                                        <label for="title">Title</label>
                                        <input type="text" name="title" class="form-control"
                                            placeholder="Enter title" required>
                                    </div>

                                    <div class="form-group">
                                        <label for="description">Description</label>
                                        <input type="text" name="description" class="form-control"
                                            placeholder="Enter Description" required>
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

                                        <a href="{{ route('dashboard.crud-0.index') }}" class="btn btn-sm btn-warning">
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
```

`edit.blade.php`:
```html
@extends('layouts.app')

@section('content')
@section('title', 'Smart ERP - Edit Entry')
<!-- Edit form -->
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
                        <div class="col-xs-12 ">
                            <div class="col-md-2"></div>

                            <div class="col-md-8">
                                <div class="widget-header widget-header-flat " style="background-color: #618f8f;">
                                    <h4 class="widget-title" style="color: #fff;">Edit Data</h4>

                                    <span class="widget-toolbar">
                                        <a href="{{ route('dashboard.crud.index') }}" style="color: #fff;">
                                            <i class="ace-icon fa fa-list"></i> Back to List
                                        </a>
                                    </span>
                                </div>

                                <!-- Edit Form Start -->
                                <form action="{{ route('dashboard.crud-0.update', $crud0->id) }}" method="POST"
                                    enctype="multipart/form-data">
                                    @csrf
                                    @method('PUT')

                                    <div class="form-group">
                                        <label for="name">Topic Name:</label>
                                        <input type="text" name="topic_name" class="form-control"
                                            value="{{ old('topic_name', $crud0->topic_name) }}" required>
                                    </div>

                                    
                                    <div class="form-group">
                                        <label for="name">Title</label>
                                        <input type="text" name="title" class="form-control"
                                            value="{{ old('title', $crud0->title) }}" required>
                                    </div>

                                    
                                    <div class="form-group">
                                        <label for="name">Description</label>
                                        <input type="text" name="description" class="form-control"
                                            value="{{ old('description', $crud0->description) }}" required>
                                    </div>

                                    

                                    <div class="form-group">
                                        <label for="status">Status</label><br>

                                        <label>
                                            <input type="radio" name="status" value="active"
                                                {{ $crud0->status == 'active' ? 'checked' : '' }}> Active
                                        </label>
                                        {{-- &nbsp;&nbsp; --}}
                                        <label>
                                            <input type="radio" name="status" value="inactive"
                                                {{ $crud0->status == 'inactive' ? 'checked' : '' }}> Inactive
                                        </label>
                                    </div>


                                    <div class="form-actions center">
                                        <button type="submit" class="btn btn-sm btn-success">
                                            Update
                                            <i class="ace-icon fa fa-check icon-on-right bigger-110"></i>
                                        </button>

                                        <a href="{{ route('dashboard.crud-0.index') }}" class="btn btn-sm btn-warning">
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
```
_____
