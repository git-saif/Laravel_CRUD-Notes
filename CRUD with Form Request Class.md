
Laravel -এ CRUD Operation এর জন্য ৫টি Step Follow করতে হয়। সেগুলো হলোঃ

1. Routes               =>  `routes/web.php`
2. Model                => `app\Models\Crud.php`
3. Migration          => `database\migrations\2025_04_14_164123_create_cruds_table.php`
4. Controller         => `app\Http\Controllers\CrudController.php`
5. Views                =>  `index.blade.php` , `create.blade.php` , `edit.blade.php`
   
নিচে এগুলোর বিস্তারিত বর্ণনা দেয়া হলোঃ

#### **Workflow**:
```scss
		(Route → Controller → FormRequest → Model → View)
```
____

## Step-1:

`routes/web.php`:
```php
Route::group(['prefix' => 'dashboard', 'as' => 'dashboard.'], function () {

    Route::resources([

        // CRUD With Advance Validation 
        'crud-4' => Crud4Controller::class,
    ]);
});
```
_____
## Step-2:

`app\Models\Crud0.php`:
```php
class Crud4 extends Model
{
    use HasFactory;

    protected $fillable = [
        'name',
        'email',
        'phone',
    ];
}
```
____
## Step-3:

`database\migrations\2025_05_05_100447_create_crud0s_table.php`:
```php
public function up(): void
    {
        Schema::create('crud4s', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email');
            $table->string('phone');
            $table->timestamps();
        });
    }
```
_____
## Step-4:

`app\Http\Controllers\Crud0Controller.php`:
```php
class Crud4Controller extends Controller
{
    /**
     * Display a listing of the resource.
     */
    public function index()
    {
        $crud4 = Crud4::orderby('id', 'asc')->paginate(3);
        return view('components.CRUD-4.index', compact('crud4'));
        // resources\views\components\CRUD-4\index.blade.php
    }

    /**
     * Show the form for creating a new resource.
     */
    public function create()
    {
        return view('components.CRUD-4.create');
    }

    /**
     * Store a newly created resource in storage.
     */
    public function store(Crud4Request $request)
    {
        Crud4::create($request->validated());

        return redirect()
        ->route('dashboard.crud-4.index')
        ->with('success', 'Data created successfully!');
        
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
        $crud4 = Crud4::findOrFail($id);
        return view('components.CRUD-4.edit', compact('crud4'));
        
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(Crud4Request $request, Crud4 $crud_4)
    {
        $crud_4->update($request->validated());

        return redirect()
            ->route('dashboard.crud-4.index')
            ->with('success', 'Data updated successfully!');
    }

    /**
     * Remove the specified resource from storage.
     */
    public function destroy(string $id)
    {
        try {
            $crud_4 = Crud4::findOrFail($id);
            $crud_4->delete();
            return redirect()
            ->route('dashboard.crud-4.index')
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

## Step-5:

```php
class Crud4Request extends FormRequest
{
    // Determine if the user is authorized to make this request.
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

    // Validation for STORE (POST)
    protected function storeRules(): array
    {
        return [
            'name'  => 'required|string|max:255',
            'email' => 'required|email|unique:crud4s,email',
            'phone' => 'required|string|max:20',
        ];
    }

    // Validation for UPDATE (PUT/PATCH)
    protected function updateRules(): array
    {
        // Get current model instance or ID from route binding
        $crud4 = $this->route('crud_4') ?? $this->route('crud-4');
        $id = $crud4?->id ?? $crud4;

        return [
            'name'  => 'required|string|max:255',
            'email' => [
                'required',
                'email',
                Rule::unique('crud4s', 'email')->ignore($id),
            ],
            'phone' => 'required|string|max:20',
        ];
    }


    public function messages(): array
    {
        return [
            'name.required'  => 'Please enter a name.',
            'email.required' => 'Email address is required.',
            'email.email'    => 'Please enter a valid email address.',
            'email.unique'   => 'This email already exists.',
            'phone.required' => 'Phone number is required.',
        ];
    }
}
```
## Step-6:

`index.blade.php`:
```html
<div>
	<table id="dynamic-table" class="table table-striped table-bordered table-hover">
	  <thead>
		<tr>
		  <th style="font-weight: bold;">Sl No</th>
		  <th>Name</th>
		  <th>Email</th>
		  <th>Phone</th>
		  <th>Action</th>
		</tr>
	  </thead>

	  <tbody>
		@php
		$sl = $crud4->firstItem() ?? 0;
		@endphp

		@forelse ($crud4 as $item)
		<tr>
		  <td style="font-weight: bold;">{{ $sl++ }}.</td>
		  <td>{{ $item->name }}</td>
		  <td>{{ $item->email }}</td>
		  <td>{{ $item->phone }}</td>

		  <td>
			<div class="hidden-sm hidden-xs action-buttons">
			  <a class="blue" href="#">
				<i class="ace-icon fa fa-eye bigger-130"></i>
			  </a>

			  <a class="green" href="{{ route('dashboard.crud-4.edit', $item->id) }}">
				<i class="ace-icon fa fa-pencil bigger-130"></i>
			  </a>

			  <form action="{{ route('dashboard.crud-4.destroy', $item->id) }}" method="POST" style="display:inline;" onsubmit="return confirm('Are you sure you want to delete this item?');">

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
		  <td colspan="5" class="text-center text-danger">No data found.</td>
		</tr>
		@endforelse
	  </tbody>
	</table>

	<div class="text-center">
	  {{ $crud4->links('pagination::bootstrap-4') }}
	</div>
</div>
```

`create.blade.php`:
```html
<div class="widget-body">
	<div class="widget-main">

	  {{-- Show validation errors --}}
	  @if ($errors->any())
	  <div class="alert alert-danger">
		<strong>Whoops!</strong> There were some problems with your input.<br><br>
		<ul>
		  @foreach ($errors->all() as $error)
		  <li>{{ $error }}</li>
		  @endforeach
		</ul>
	  </div>
	  @endif

	  {{-- Create Form --}}
	  <form action="{{ route('dashboard.crud-4.store') }}" method="POST" class="form-horizontal" role="form">
		@csrf

		<div class="form-group">
		  <label class="col-sm-2 control-label no-padding-right" for="name">Name</label>
		  <div class="col-sm-9">

			<input type="text" id="name" name="name" placeholder="Enter name" value="{{ old('name') }}" class="col-xs-10 col-sm-5 {{ $errors->has('name') ? 'is-invalid' : '' }}" />
			@error('name')
			<span class="text-danger small">{{ $message }}</span>
			@enderror
			
		  </div>
		</div>

		<div class="space-4"></div>

		<div class="form-group">
		  <label class="col-sm-2 control-label no-padding-right" for="email">Email</label>
		  <div class="col-sm-9">
			<input type="text" id="email" name="email" placeholder="Enter email" value="{{ old('email') }}" class="col-xs-10 col-sm-5 {{ $errors->has('email') ? 'is-invalid' : '' }}" />
			@error('email')
			<span class="text-danger small pt-2">{{ $message }}</span>
			@enderror
		  </div>
		</div>

		<div class="space-4"></div>

		<div class="form-group">
		  <label class="col-sm-2 control-label no-padding-right" for="phone">Phone</label>
		  <div class="col-sm-9">
			<input type="text" id="phone" name="phone" placeholder="Enter phone number" value="{{ old('phone') }}" class="col-xs-10 col-sm-5 {{ $errors->has('phone') ? 'is-invalid' : '' }}" />
			@error('phone')
			<span class="text-danger small">{{ $message }}</span>
			@enderror
		  </div>
		</div>

		<div class="space-4"></div>

		<div class="clearfix form-actions">
		  <div class="col-md-offset-2 col-md-9">
			<button class="btn btn-info" type="submit">
			  <i class="ace-icon fa fa-check bigger-110"></i>
			  Submit
			</button>

			&nbsp; &nbsp; &nbsp;
			<button class="btn" type="reset">
			  <i class="ace-icon fa fa-undo bigger-110"></i>
			  Reset
			</button>
		  </div>
		</div>
	  </form>
	</div>
  </div>
```

`edit.blade.php`:
```html
<div class="widget-body">
	<div class="widget-main">

	  {{-- Show validation errors --}}
	  @if ($errors->any())
	  <div class="alert alert-danger">
		<strong>Whoops!</strong> There were some problems with your input.<br><br>
		<ul>
		  @foreach ($errors->all() as $error)
		  <li>{{ $error }}</li>
		  @endforeach
		</ul>
	  </div>
	  @endif

	  {{-- Edit Form --}}
	  <form action="{{ route('dashboard.crud-4.update', $crud4->id) }}" method="POST" class="form-horizontal" role="form">
		@csrf
		@method('PUT')

		<div class="form-group">
		  <label class="col-sm-2 control-label no-padding-right" for="name">Name</label>
		  <div class="col-sm-9">
			<input type="text" id="name" name="name" placeholder="Enter name" value="{{ old('name', $crud4->name) }}" class="col-xs-10 col-sm-5 {{ $errors->has('name') ? 'is-invalid' : '' }}" />
			@error('name')
			<span class="text-danger small">{{ $message }}</span>
			@enderror
		  </div>
		</div>

		<div class="space-4"></div>

		<div class="form-group">
		  <label class="col-sm-2 control-label no-padding-right" for="email">Email</label>
		  <div class="col-sm-9">
			<input type="text" id="email" name="email" placeholder="Enter email" value="{{ old('email', $crud4->email) }}" class="col-xs-10 col-sm-5 {{ $errors->has('email') ? 'is-invalid' : '' }}" />
			@error('email')
			<span class="text-danger small pt-2">{{ $message }}</span>
			@enderror
		  </div>
		</div>

		<div class="space-4"></div>

		<div class="form-group">
		  <label class="col-sm-2 control-label no-padding-right" for="phone">Phone</label>
		  <div class="col-sm-9">
			<input type="text" id="phone" name="phone" placeholder="Enter phone number" value="{{ old('phone', $crud4->phone) }}" class="col-xs-10 col-sm-5 {{ $errors->has('phone') ? 'is-invalid' : '' }}" />
			@error('phone')
			<span class="text-danger small">{{ $message }}</span>
			@enderror
		  </div>
		</div>

		<div class="space-4"></div>

		<div class="clearfix form-actions">
		  <div class="col-md-offset-2 col-md-9">
			<button class="btn btn-info" type="submit">
			  <i class="ace-icon fa fa-check bigger-110"></i>
			  Update
			</button>

			&nbsp; &nbsp; &nbsp;
			<a href="{{ route('dashboard.crud-4.index') }}" class="btn">
			  <i class="ace-icon fa fa-undo bigger-110"></i>
			  Cancel
			</a>
		  </div>
		</div>
	  </form>

	</div>
</div>
```
_____
