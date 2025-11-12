এই CRUD এ আমরা Company-Settings manage করতে পারবো। Laravel -এ এই CRUD Operation এর জন্য 7টি Step Follow করতে হয়। সেগুলো হলোঃ

1. Routes               =>  `routes\web.php`
2. Model                => `app\Models\Company.php`
3. Migration          => `database\migrations\2025_10_15_184849_create_companies_table.php`
4. Controller         => `app\Http\Controllers\CompanyController.php`
5. Request form   =>  `app\Http\Requests\CompanyRequest.php`
6. Views                =>  `index.blade.php` , `create.blade.php` , `edit.blade.php`
7. Global Sharing =>  `app\Providers\AppServiceProvider.php:`
   
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
    
        // Company Settings
        'company' => CompanyController::class,
    ]);
});
```
_____
## Step-2: (Model)

##### `app\Models\Company.php`:
```php
class Company extends Model
{
    use HasFactory;

    protected $fillable = [
        'company_name',
        'company_tagline',
        'company_email',
        'company_phone',
        'company_address',
        'company_favicon',
        'company_logo',
        'company_booking_link',
        'company_whatsapp_link',
        'company_facebook_link',
        'company_instagram_link',
        'company_twitter_link',
        'company_youtube_link',
        'company_linkedin_link',
        'company_google_map_link',
        'status',
    ];
}
```
____
## Step-3: (Migration)

##### `database\migrations\2025_10_15_184849_create_companies_table.php`:
```php
	Schema::create('companies', function (Blueprint $table) {
            $table->id();
            $table->string('company_name');
            $table->string('company_tagline');
            $table->string('company_email');
            $table->string('company_phone');
            $table->string('company_address');
            $table->string('company_favicon')->nullable();
            $table->string('company_logo')->nullable();
            $table->string('company_booking_link')->nullable();
            $table->string('company_whatsapp_link')->nullable();
            $table->string('company_facebook_link')->nullable();
            $table->string('company_instagram_link')->nullable();
            $table->string('company_twitter_link')->nullable();
            $table->string('company_youtube_link')->nullable();
            $table->string('company_linkedin_link')->nullable();
            $table->string('company_google_map_link')->nullable();
            $table->enum('status', ['active', 'inactive'])->default('active');
            $table->timestamps();
        });
```
_____
## Step-4: (Controller)

##### `app\Http\Controllers\CompanyController.php`:
```php
class CompanyController extends Controller
{
    /**
     * Display a listing of the resource.
     */
    public function index()
    {
        $company = Company::orderBy('id', 'asc')->paginate(5);
        return view('components.company-settings.index', compact('company'));
    }

    /**
     * Show the form for creating a new resource.
     */
    public function create()
    {
        return view('components.company-settings.create');
    }

    /**
     * Store a newly created resource in storage.
     */
    public function store(CompanyRequest $request)
    {
        try {
            $data = $request->validated();

            // Handle images
            $data['company_logo'] = $this->uploadImage($request, 'company_logo');
            $data['company_favicon'] = $this->uploadImage($request, 'company_favicon');

            Company::create($data);

            return redirect()->route('dashboard.company.index')
                ->with('success', 'Company setting created successfully!');
        } catch (\Exception $e) {
            return redirect()->back()->with('error', $e->getMessage());
        }
    }

    /**
     * Show the form for editing the specified resource.
     */
    public function edit(string $id)
    {
        $company = Company::findOrFail($id);
        return view('components.company-settings.edit', compact('company'));
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(CompanyRequest $request, string $id) 
    {
        try {
            $company = Company::findOrFail($id);
            $data = $request->validated();

            // Update images if new ones are uploaded
            if ($request->hasFile('company_logo')) {
                $this->deleteOldFile($company->company_logo);
                $data['company_logo'] = $this->uploadImage($request, 'company_logo');
            }

            if ($request->hasFile('company_favicon')) {
                $this->deleteOldFile($company->company_favicon);
                $data['company_favicon'] = $this->uploadImage($request, 'company_favicon');
            }

            $company->update($data);

            return redirect()->route('dashboard.company.index')
                ->with('success', 'Company setting updated successfully!');
        } catch (\Exception $e) {
            return redirect()->back()->with('error', $e->getMessage());
        }
    }

    /**
     * Remove the specified resource from storage.
     */
    public function destroy(string $id)
    {
        try {
            $company = Company::findOrFail($id);
            $this->deleteOldFile($company->company_logo);
            $this->deleteOldFile($company->company_favicon);
            $company->delete();

            return redirect()->route('dashboard.company.index')
                ->with('success', 'Company setting deleted successfully!');
        } catch (\Exception $e) {
            return redirect()->back()->with('error', $e->getMessage());
        }
    }

    /*
    |--------------------------------------------------------------------------
    |                       Helper Functions
    |--------------------------------------------------------------------------
    */
    private function uploadImage($request, $field)
    {
        if ($request->hasFile($field)) {
            $file = $request->file($field);
            $name = time() . '_' . $file->getClientOriginalName();
            $path = 'uploads/images/company-settings/';
            $file->move(public_path($path), $name);
            return $path . $name;
        }
        return null;
    }

    private function deleteOldFile($filePath)
    {
        if ($filePath && file_exists(public_path($filePath))) {
            unlink(public_path($filePath));
        }
    }
}
```
______

## Step-5: (Form Request)

##### **`app\Http\Requests\CompanyRequest.php:`**
```php
class CompanyRequest extends FormRequest
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
            'company_name'           => 'required|string|max:255',
            'company_tagline'        => 'required|string|max:255',
            'company_email'          => 'required|email|max:255|unique:companies,company_email',
            'company_phone'          => 'required|string|max:255|unique:companies,company_phone',
            'company_address'        => 'required|string|max:255',
            'company_favicon'        => 'required|image|mimes:ico,png,jpg,jpeg|max:2048',
            'company_logo'           => 'required|image|mimes:jpeg,png,jpg,svg|max:2048',
            'company_booking_link'   => 'required|string|max:255',
            'company_whatsapp_link'  => 'required|string|max:255',
            'company_facebook_link'  => 'required|string|max:255',
            'company_instagram_link' => 'required|string|max:255',
            'company_twitter_link'   => 'required|string|max:255',
            'company_youtube_link'   => 'required|string|max:255',
            'company_linkedin_link'  => 'required|string|max:255',
            'company_google_map_link' => 'required|string|max:255',
            'status'                 => 'required|in:active,inactive',
        ];
    }

    protected function updateRules(): array
    {
        $setting = $this->route('company'); // resource route parameter
        $id = $setting?->id ?? $setting;

        return [
            'company_name'           => 'required|string|max:255',
            'company_tagline'        => 'required|string|max:255',
            'company_email'          => [
                'required',
                'email',
                'max:255',
                Rule::unique('companies', 'company_email')->ignore($id),
            ],
            'company_phone'          => [
                'required',
                'string',
                'max:255',
                Rule::unique('companies', 'company_phone')->ignore($id),
            ],
            'company_address'        => 'required|string|max:255',
            'company_favicon'        => 'nullable|image|mimes:ico,png,jpg,jpeg|max:2048',
            'company_logo'           => 'nullable|image|mimes:jpeg,png,jpg,svg|max:2048',
            'company_booking_link'   => 'required|string|max:255',
            'company_whatsapp_link'  => 'required|string|max:255',
            'company_facebook_link'  => 'required|string|max:255',
            'company_instagram_link' => 'required|string|max:255',
            'company_twitter_link'   => 'required|string|max:255',
            'company_youtube_link'   => 'required|string|max:255',
            'company_linkedin_link'  => 'required|string|max:255',
            'company_google_map_link' => 'required|string|max:255',
            'status'                 => 'required|in:active,inactive',
        ];
    }

    public function messages(): array
    {
        return [
            'required' => 'The :attribute field is required.',
            'email.email' => 'Please enter a valid email address.',
            'company_email.unique' => 'This email already exists.',
            'company_phone.unique' => 'This phone number already exists.',
            'company_favicon.image' => 'Favicon must be an image file.',
            'company_logo.image' => 'Logo must be an image file.',
        ];
    }
}
```
-------

## Step-6: (View Create)

##### `index.blade.php`: 
```html
@extends('layouts.app')
@section('title', 'Company Settings')
@section('content')
<div class="main-content">
  <div class="col-md-12">
    <div class="widget-box">
      <div class="widget-header d-flex justify-content-between align-items-center">
        <h4 class="widget-title">Company Settings</h4>

        <span class="widget-toolbar">
          <a href="{{ route('dashboard.company.create') }}">
            <i class="ace-icon fa fa-plus"></i> Create Company Settings
          </a>
        </span>
      </div>

      <div class="widget-body">
        <div class="widget-main">

          <div class="row" style="margin: 15px">
            @forelse ($company as $setting)
            <div class="col-md-12 mb-3">
              <div class="widget-box">
                <div class="widget-header">
                  <h5 class="widget-title">Company Info (ID: {{ $setting->id }})</h5>
                </div>

                <div class="widget-body">
                  <div class="widget-main">
                    <div class="row">

                      {{-- Basic Info --}}
                      @php
                      $fields = [
                      ['Company Name', $setting->company_name],
                      ['Tagline', $setting->company_tagline],
                      ['Email', $setting->company_email],
                      ['Phone', $setting->company_phone],
                      ['Address', $setting->company_address],
                      ];
                      @endphp

                      @foreach ($fields as $field)
                      <div class="col-md-4 mb-2">
                        <div class="well well-sm">
                          <strong>{{ $field[0] }}:</strong>
                          <div>{{ $field[1] ?? 'N/A' }}</div>
                        </div>
                      </div>
                      @endforeach

                      {{-- Logo --}}
                      <div class="col-md-4 mb-2">
                        <div class="well well-sm">
                          <strong>Logo:</strong><br>
                          @if ($setting->company_logo)
                          <img src="{{ asset($setting->company_logo) }}" width="60" alt="Logo">
                          @else
                          N/A
                          @endif
                        </div>
                      </div>

                      {{-- Favicon --}}
                      <div class="col-md-4 mb-2">
                        <div class="well well-sm">
                          <strong>Favicon:</strong><br>
                          @if ($setting->company_favicon)
                          <img src="{{ asset($setting->company_favicon) }}" width="30" alt="Favicon">
                          @else
                          N/A
                          @endif
                        </div>
                      </div>

                      {{-- Social and Booking Links --}}
                      @php
                      $links = [
                      ['Booking Link', $setting->company_booking_link, 'fa-link'],
                      ['WhatsApp', $setting->company_whatsapp_link, 'fa-whatsapp'],
                      ['Facebook', $setting->company_facebook_link, 'fa-facebook'],
                      ['Instagram', $setting->company_instagram_link, 'fa-instagram'],
                      ['Twitter', $setting->company_twitter_link, 'fa-twitter'],
                      ['YouTube', $setting->company_youtube_link, 'fa-youtube'],
                      ['LinkedIn', $setting->company_linkedin_link, 'fa-linkedin'],
                      ['Map Link', $setting->company_google_map_link, 'fa-map-marker'],
                      ];
                      @endphp

                      @foreach ($links as $link)
                      <div class="col-md-4 mb-2">
                        <div class="well well-sm">
                          <strong>{{ $link[0] }}:</strong>
                          <div>
                            @if ($link[1])
                            <a href="{{ $link[1] }}" target="_blank">
                              <i class="fa {{ $link[2] }} fa-lg"></i>
                            </a>
                            @else
                            N/A
                            @endif
                          </div>
                        </div>
                      </div>
                      @endforeach

                      {{-- Status --}}
                      <div class="col-md-4 mb-2">
                        <div class="well well-sm">
                          <strong>Status:</strong>
                          <div>
                            @if ($setting->status === 'active')
                            <span class="label label-success">Active</span>
                            @else
                            <span class="label label-danger">Inactive</span>
                            @endif
                          </div>
                        </div>
                      </div>

                      {{-- Actions --}}
                      <div class="col-md-12 text-center">
                        <div class="well well-sm">
                          <strong>Actions:</strong><br>
                          <a href="{{ route('dashboard.company.edit', $setting->id) }}" class="btn btn-primary btn-sm">
                            <i class="fa fa-edit"></i> Edit
                          </a>

                          <form action="{{ route('dashboard.company.destroy', $setting->id) }}" method="POST" style="display:inline;" onsubmit="return confirm('Are you sure you want to delete this?');">
                            @csrf
                            @method('DELETE')
                            <button type="submit" class="btn btn-danger btn-sm">
                              <i class="fa fa-trash"></i> Delete
                            </button>
                          </form>
                        </div>
                      </div>

                    </div>
                  </div>
                </div>
              </div>
            </div>
            @empty
            <div class="col-md-12 text-center">
              <p>No company settings found.</p>
            </div>
            @endforelse

            {{-- Pagination --}}
            <div class="col-md-12 text-center">
              {{ $company->links() }}
            </div>
          </div>

        </div>
      </div>
    </div>
  </div>
</div>
@endsection
```
---

##### `create.blade.php`: 
```html
@extends('layouts.app')
@section('title', 'Company Settings Create')
@section('content')
<div class="main-content">
    <div class="col-md-12">
        <div class="widget-box">
            <div class="widget-header">
                <h4 class="widget-title">Create Company Setting</h4>
                <span class="widget-toolbar">
                    <a href="{{ route('dashboard.company.index') }}">
                        <i class="ace-icon fa fa-list"></i> Company Settings
                    </a>
                </span>
            </div>

            <div class="widget-body">
                <div class="widget-main no-padding">
                    <div class="row" style="margin: 25px;">
                        <div class="col-md-12">
                            {{-- Success / Error Message --}}
                            @if (session('success'))
                                <div class="alert alert-success">{{ session('success') }}</div>
                            @endif
                            @if (session('error'))
                                <div class="alert alert-danger">{{ session('error') }}</div>
                            @endif

                            <form action="{{ route('dashboard.company.store') }}" 
                                  method="POST" enctype="multipart/form-data">
                                @csrf

                                <div class="row">
                                    {{-- Company Info --}}
                                    <div class="col-md-4 mb-3">
                                        <label>Company Name</label>
                                        <input type="text" name="company_name" class="form-control"
                                            value="{{ old('company_name') }}" required>
                                        @error('company_name') <span class="text-danger">{{ $message }}</span> @enderror
                                    </div>

                                    <div class="col-md-4 mb-3">
                                        <label>Company Tagline</label>
                                        <input type="text" name="company_tagline" class="form-control"
                                            value="{{ old('company_tagline') }}" required>
                                    </div>

                                    <div class="col-md-4 mb-3">
                                        <label>Company Email</label>
                                        <input type="email" name="company_email" class="form-control"
                                            value="{{ old('company_email') }}" required>
                                    </div>

                                    <div class="col-md-4 mb-3">
                                        <label>Company Phone</label>
                                        <input type="number" name="company_phone" class="form-control"
                                            value="{{ old('company_phone') }}" required>
                                    </div>

                                    <div class="col-md-4 mb-3">
                                        <label>Company Address</label>
                                        <textarea name="company_address" class="form-control" required>{{ old('company_address') }}</textarea>
                                    </div>

                                    {{-- Links --}}
                                    @php
                                        $links = [
                                            'company_booking_link' => 'Booking Link',
                                            'company_whatsapp_link' => 'WhatsApp Link',
                                            'company_facebook_link' => 'Facebook Link',
                                            'company_instagram_link' => 'Instagram Link',
                                            'company_twitter_link' => 'Twitter Link',
                                            'company_youtube_link' => 'YouTube Link',
                                            'company_linkedin_link' => 'LinkedIn Link',
                                            'company_google_map_link' => 'Google Map Link',
                                        ];
                                    @endphp

                                    @foreach ($links as $name => $label)
                                        <div class="col-md-4 mb-3">
                                            <label>{{ $label }}</label>
                                            <input type="url" name="{{ $name }}" class="form-control"
                                                value="{{ old($name) }}">
                                        </div>
                                    @endforeach

                                    {{-- Status --}}
                                    <div class="col-md-4 mb-3">
                                        <label>Status</label>
                                        <select name="status" class="form-control" required>
                                            <option value="active" {{ old('status') == 'active' ? 'selected' : '' }}>Active</option>
                                            <option value="inactive" {{ old('status') == 'inactive' ? 'selected' : '' }}>Inactive</option>
                                        </select>
                                    </div>

                                    {{-- Images --}}
                                    <div class="col-md-4 mb-3">
                                        <label>Company Logo</label>
                                        <input type="file" name="company_logo" class="form-control" required>
                                    </div>

                                    <div class="col-md-4 mb-3">
                                        <label>Company Favicon</label>
                                        <input type="file" name="company_favicon" class="form-control" required>
                                    </div>
                                </div>

                                <div class="mt-3">
                                    <button type="submit" class="btn btn-sm btn-success">
                                        <i class="ace-icon fa fa-check"></i> Submit
                                    </button>
                                    <a href="{{ route('dashboard.company.index') }}" class="btn btn-sm btn-secondary">
                                        <i class="ace-icon fa fa-arrow-left"></i> Back
                                    </a>
                                </div>
                            </form>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```
___

##### `edit.blade.php`:
```html
@extends('layouts.app')
@section('content')
@section('title', 'Company Settings Edit')
<div class="main-content">
  <div class="col-md-12">
    <div class="widget-box">
      <div class="widget-header">
        <h4 class="widget-title">Edit Company-Settings</h4>
        <span class="widget-toolbar">
          <a href="{{ route('dashboard.company.index') }}">
            <i class="ace-icon fa fa-list"></i> Go to Company-Settings
          </a>
        </span>
      </div>

      <div class="widget-body">
        <div class="widget-main no-padding">
          <div class="row" style="margin: 25px;">
            <div class="col-md-12 ">
              <form action="{{ route('dashboard.company.update', $company->id) }}" method="POST" enctype="multipart/form-data">
                @csrf
                @method('PUT')

                <div class="row">
                  {{-- Company Name --}}
                  <div class="col-md-4 mb-3">
                    <label>Company Name</label>
                    <input type="text" name="company_name" class="form-control" value="{{ old('company_name', $company->company_name) }}" required>
                  </div>

                  {{-- Company Tagline --}}
                  <div class="col-md-4 mb-3">
                    <label>Company Tagline</label>
                    <input type="text" name="company_tagline" class="form-control" value="{{ old('company_tagline', $company->company_tagline) }}" required>
                  </div>

                  {{-- Email --}}
                  <div class="col-md-4 mb-3">
                    <label>Company Email</label>
                    <input type="email" name="company_email" class="form-control" value="{{ old('company_email', $company->company_email) }}" required>
                  </div>

                  {{-- Phone --}}
                  <div class="col-md-4 mb-3">
                    <label>Company Phone</label>
                    <input type="text" name="company_phone" class="form-control" value="{{ old('company_phone', $company->company_phone) }}" required>
                  </div>

                  {{-- Address --}}
                  <div class="col-md-4 mb-3">
                    <label>Company Address</label>
                    <textarea name="company_address" class="form-control" required>{{ old('company_address', $company->company_address) }}</textarea>
                  </div>

                  {{-- Links --}}
                  <div class="col-md-4 mb-3">
                    <label>Booking Link</label>
                    <input type="url" name="company_booking_link" class="form-control" value="{{ old('company_booking_link', $company->company_booking_link) }}">
                  </div>
                  <div class="col-md-4 mb-3">
                    <label>WhatsApp Link</label>
                    <input type="url" name="company_whatsapp_link" class="form-control" value="{{ old('company_whatsapp_link', $company->company_whatsapp_link) }}">
                  </div>
                  <div class="col-md-4 mb-3">
                    <label>Facebook Link</label>
                    <input type="url" name="company_facebook_link" class="form-control" value="{{ old('company_facebook_link', $company->company_facebook_link) }}">
                  </div>
                  <div class="col-md-4 mb-3">
                    <label>Instagram Link</label>
                    <input type="url" name="company_instagram_link" class="form-control" value="{{ old('company_instagram_link', $company->company_instagram_link) }}">
                  </div>
                  <div class="col-md-4 mb-3">
                    <label>Twitter Link</label>
                    <input type="url" name="company_twitter_link" class="form-control" value="{{ old('company_twitter_link', $company->company_twitter_link) }}">
                  </div>
                  <div class="col-md-4 mb-3">
                    <label>YouTube Link</label>
                    <input type="url" name="company_youtube_link" class="form-control" value="{{ old('company_youtube_link', $company->company_youtube_link) }}">
                  </div>
                  <div class="col-md-4 mb-3">
                    <label>LinkedIn Link</label>
                    <input type="url" name="company_linkedin_link" class="form-control" value="{{ old('company_linkedin_link', $company->company_linkedin_link) }}">
                  </div>
                  <div class="col-md-4 mb-3">
                    <label>Google Map Link</label>
                    <input type="url" name="company_google_map_link" class="form-control" value="{{ old('company_google_map_link', $company->company_google_map_link) }}">
                  </div>

                  {{-- Status --}}
                  <div class="col-md-4 mb-3">
                    <label>Status</label>
                    <select name="status" class="form-control" required>
                      <option value="active" @selected(old('status', $company->status) == 'active')>Active</option>
                      <option value="inactive" @selected(old('status', $company->status) == 'inactive')>Inactive</option>
                    </select>
                  </div>

                  {{-- Logo --}}
                  <div class="col-md-4 mb-3">
                    <label>Company Logo</label><br>
                    @if ($company->company_logo)
                    <img src="{{ asset($company->company_logo) }}" width="60" class="mb-2">
                    @endif
                    <input type="file" name="company_logo" class="form-control">
                  </div>

                  {{-- Favicon --}}
                  <div class="col-md-4 mb-3">
                    <label>Company Favicon</label><br>
                    @if ($company->company_favicon)
                    <img src="{{ asset($company->company_favicon) }}" width="30" class="mb-2">
                    @endif
                    <input type="file" name="company_favicon" class="form-control">
                  </div>

                </div>

                <div class="mt-2">
                  <button type="submit" class="btn btn-sm btn-primary">
                    Update <i class="ace-icon fa fa-check icon-on-right bigger-110"></i>
                  </button>
                </div>
              </form>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
@endsection
```
____

## Step-7: (Create Global View Data Sharing System)

##### `app\Providers\AppServiceProvider.php:`
```php
namespace App\Providers;

use App\Models\Company;
use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Schema; // <-- add this

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        //
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        
        // শুধুমাত্র যদি companies table থাকে
        $companySettings = null;
        if (Schema::hasTable('companies')) {
            $companySettings = Company::where('status', 'active')->first();
        }

        // সব Blade view-এ share করবে
        View::share('companySettings', $companySettings);
    }
}
```
_________

##### **Use in Blade View:**
```php
{{ $activeCompany?->company_name ?? '' }}
{{ $activeCompany?->company_tagline ?? '' }}
{{ $activeCompany?->company_email ?? '' }}
{{ $activeCompany?->company_phone ?? '' }}
{{ $activeCompany?->company_address ?? '' }}
{{ $activeCompany?->company_booking_link ?? '' }}
{{ $activeCompany?->company_whatsapp_link ?? '' }}
{{ $activeCompany?->company_facebook_link ?? '' }}
{{ $activeCompany?->company_instagram_link ?? '' }}
{{ $activeCompany?->company_twitter_link ?? '' }}
{{ $activeCompany?->company_youtube_link ?? '' }}
{{ $activeCompany?->company_linkedin_link ?? '' }}
{{ $activeCompany?->company_google_map_link ?? '' }}
{{ $activeCompany?->status ?? '' }}

<img src="{{ $activeCompany?->company_logo ? asset($activeCompany->company_logo) : asset('default-logo.png') }}" alt="Logo" width="60">
<img src="{{ $activeCompany?->company_favicon ? asset($activeCompany->company_favicon) : asset('favicon.ico') }}" alt="Favicon" width="30">
```
____
