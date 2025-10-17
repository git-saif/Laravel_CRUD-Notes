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

##### `routes/web.php`:
```php
Route::group(['prefix' => 'dashboard', 'as' => 'dashboard.'], function () {

    Route::resources([

        // CRUD (Post)
        'crud-10' => Crud10Controller::class,  // Post
    ]);
    
    // image upload for CKEditor (AJAX)
    Route::post('crud-10/upload-image', [Crud10Controller::class, 'uploadImage'])->name('crud-10.uploadImage');
});
```
_____

## Step-2: (Controller -> add this function)

##### **`app\Http\Requests\Crud10Request.php:`**
```php
/*
    |--------------------------------------------------------------------------
    |               Image upload handler for CKEditor
    |--------------------------------------------------------------------------
    */
    public function uploadImage(Request $request)
    {
        if ($request->hasFile('upload')) {
            $file = $request->file('upload');
            $filename = time() . '_' . $file->getClientOriginalName();

            // Ensure the folder exists
            $folder = public_path('uploads/ckeditor');
            if (!file_exists($folder)) {
                mkdir($folder, 0775, true);
            }

            $file->move($folder, $filename);
            $url = asset('uploads/ckeditor/' . $filename);

            // CKEditor expects this JSON
            return response()->json([
                'url' => $url
            ]);
        }

        return response()->json(['error' => 'No file uploaded.'], 400);
    }
```
----

## Step-6: (View Create)

##### `show.blade.php`: (form only)
```html
	{{-- ✅ Render HTML content safely --}}
	<td>{!! $crud10->post !!}</td>
```
---

##### `create.blade.php`: (form only)
```html
{{-- Post Content --}}
<div class="form-group mb-3">
	<label>Full Post *</label>
	<textarea id="editor" name="post" class="form-control" rows="6">{{ old('post', $crud10->post ?? '') }}</textarea>
	@error('post') <small class="text-danger">{{ $message }}</small> @enderror
</div>

@push('scripts')
<!-- CKEditor 5 Classic (CDN) -->
<script src="https://cdn.ckeditor.com/ckeditor5/39.0.1/classic/ckeditor.js"></script>
<script>
  ClassicEditor
    .create(document.querySelector('#editor'), {
      simpleUpload: {
        uploadUrl: '{{ route("dashboard.crud-10.uploadImage") }}'
        , headers: {
          'X-CSRF-TOKEN': '{{ csrf_token() }}'
        }
      }
    })
    .then(editor => console.log('Editor initialized', editor))
    .catch(error => console.error(error));
</script>
```
___

##### `edit.blade.php`: (form only)
```html
{{-- Post Content --}}
<div class="mb-3">
	<label>Post Content *</label>
	<textarea id="editor" name="post" class="form-control" rows="6">{{ $crud10->post }}</textarea>
</div>

@push('scripts')
<!-- CKEditor 5 Classic (CDN) -->
<script src="https://cdn.ckeditor.com/ckeditor5/39.0.1/classic/ckeditor.js"></script>
<script>
  document.addEventListener('DOMContentLoaded', function() {
    ClassicEditor
      .create(document.querySelector('#editor'), {
        simpleUpload: {
          uploadUrl: '{{ route("dashboard.crud-10.uploadImage") }}'
          , headers: {
            'X-CSRF-TOKEN': '{{ csrf_token() }}'
          }
        }
      })
      .catch(error => {
        console.error(error);
      });
  });
</script>
@endpush
```
____
