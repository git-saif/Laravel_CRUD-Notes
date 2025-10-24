**Laravel Livewire** হলো একটি **full-stack framework** যা Laravel এর সাথে কাজ করে। এর সাহায্যে **JavaScript লিখা ছাড়া interactive UI** তৈরি করা যায়। 

- এটি **Blade templates + PHP** ব্যবহার করে রিয়েল-টাইম DOM আপডেট করে।
- ফর্ম সাবমিশন, টেবিল ফিল্টার, modal ইত্যাদি **reload ছাড়া** কাজ করে।
- Livewire মূলত **Server-driven UI** দেয়, তাই তুমি সহজেই **component-based structure** তৈরি করতে পারো।

সংক্ষেপে:
> Livewire = Laravel + Realtime UI + JS কম ব্যবহার।

নিচে Laravel Livewire Installation Process step by step আলোচনা করা হলোঃ

# Step 1: (Laravel Project Setup)

প্রথমেই যেভাবে সাধারণত Laravel Install & setup করা হয়, সেভাবেই Setup করে নিতে হবে।
____
# Step-2: (Livewire Install & Setup)

**`Terminal:`**
```bash
composer require livewire/livewire
```

>`resources/views/components/layouts/app.blade.php` এর `<head>`-এ এবং `</body>`-এর আগে যুক্ত করতে হবে:

**`<head>`** এর মধ্যে:
```blade
@livewireStyles
```

**`</body>`** এর আগে:
```blade
@livewireScripts
```
---
# Step-3: (Master Layout File Create)

##### `resources/views/components/layouts/app.blade.php: (Default Path)`
```html
<!DOCTYPE html>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Livewire App</title>
    @livewireStyles
</head>
<body>

    {{-- Header --}}
    @include('components.partials.header')

    {{-- Sidebar --}}
    @include('components.partials.sidebar')

    {{-- Main Content --}}
    <div class="main-content">
        {{ $slot }}  {{-- এখানে Livewire component inject হবে --}}
    </div>

    {{-- Footer --}}
    @include('components.partials.footer')

    @livewireScripts
</body>
</html>

```
_____
# Step-4: (Component Create)

##### `Terminal:`
```bash
php artisan make:livewire HelloWorld
```
or,
```bash
# Specific Location >>>>>>>>>>>
php artisan make:livewire Demo/HelloWorld
```

- এটি দুইটি ফাইল তৈরি করবে:
  
>`app/Livewire/HelloWorld.php` (class)
 >`resources/views/livewire/hello-world.blade.php` (view) 
___
# Step-5: (Component Method)

##### `app/Livewire/HelloWorld.php` (class for default master layout file view)
```php
<?php

namespace App\Livewire;

use Livewire\Component;

class HelloWorld extends Component
{
    public function render()
    {
        return view('livewire.hello-world');
    }
}
```

##### `app/Livewire/HelloWorld.php` (class for Custom master layout file view)
```php
class HelloWorld extends Component
{
    protected $layout = 'layouts.app';  // view path => resources/views/layouts/app.blade.php
    public function render()
    {
        return view('livewire.hello-world');
    }
}
```

##### `app/Livewire/HelloWorld.php` (class for Dynamically custom master layout file view)
```php
public function render()
{
    return view('livewire.hello-world')
           ->layout('layouts.app');  // view path => resources/views/layouts/app.blade.php
}
```
________
# Step-5: (View File Create)

##### `resources/views/livewire/hello-world.blade.php` (view) 
```html
<div>
  <h1>Hello World — This is From Livewire</h1>
</div>
```
_______
# Step-5: (Route Create)

##### `Web.php:`
```php
use App\Livewire\HelloWorld;

Route::get('/hello', function() {
    return view('livewire.hello-world');
});
```
_____
