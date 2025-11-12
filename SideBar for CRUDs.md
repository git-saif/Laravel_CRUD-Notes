একটি **sidebar navigation menu**, যা সাধারণত “Admin Dashboard”-এ ব্যবহার করা হয় (এখানে Ace Admin Template ব্যবহার করা হয়েছে)।  নিচে **ধাপে ধাপে (line by line)** বর্ণনা করা হলো:

---
## 🧩 ১. Sidebar Wrapper

```html
<div id="sidebar" class="sidebar responsive ace-save-state">
```

➡️ এটি পুরো sidebar অংশের container।

- `id="sidebar"` → জাভাস্ক্রিপ্ট দিয়ে সহজে টার্গেট করার জন্য।
- `class="sidebar responsive ace-save-state"` →
    - `sidebar`: স্টাইলের জন্য (Ace Admin এর CSS ক্লাস)।
    - `responsive`: ছোট স্ক্রিনে রেসপনসিভ হয়।
    - `ace-save-state`: sidebar-এর অবস্থান (open/collapse) ব্রাউজারে সংরক্ষণ করে।

---

## 🧠 ২. Sidebar State Load Script

```html
<script type="text/javascript">
    try {
        ace.settings.loadState('sidebar')
    } catch (e) {}
</script>
```

➡️ এটি Ace Admin Template-এর JS ফাংশন।  
এই স্ক্রিপ্ট sidebar-এর আগের “state” (যেমন: collapsed/open) ব্রাউজারের localStorage থেকে লোড করে।

---

## 🧩 ৩. Sidebar Shortcuts

```html
<div class="sidebar-shortcuts" id="sidebar-shortcuts">
```

➡️ Sidebar-এর উপরে কিছু শর্টকাট বোতাম (Quick Access Buttons) রাখার জন্য এই সেকশন।

---

### বড় ভার্সন (Desktop)

```html
<div class="sidebar-shortcuts-large" id="sidebar-shortcuts-large">
    <button class="btn btn-success">
        <i class="ace-icon fa fa-signal"></i>
    </button>
    ...
</div>
```

➡️ এখানে ৪টি বোতাম:

- সবগুলোর আলাদা রঙ (`btn-success`, `btn-info`, `btn-warning`, `btn-danger`)
    
- প্রত্যেকটিতে একটি করে Font Awesome আইকন আছে (যেমন: signal, pencil, users, cogs)  
    💡 ডেস্কটপে দেখা যায়।
    

---

### ছোট ভার্সন (Mobile)

```html
<div class="sidebar-shortcuts-mini" id="sidebar-shortcuts-mini">
    <span class="btn btn-success"></span>
    ...
</div>
```

➡️ ছোট রূপে (icon ছাড়াই) শুধু রঙের ব্লক হিসেবে দেখানো হয়।

---

## 📋 ৪. Main Menu List শুরু

```html
<ul class="nav nav-list">
```

➡️ Sidebar-এর মূল ন্যাভিগেশন মেনু।

---

### Dashboard Item

```html
<li class="{{ Request::is('dashboard') ? 'active' : '' }}">
    <a href="{{ route('dashboard') }}">
        <i class="menu-icon fa fa-tachometer"></i>
        <span class="menu-text"> Dashboard </span>
    </a>
    <b class="arrow"></b>
</li>
```

➡️

- `Request::is('dashboard') ? 'active' : ''` → যদি ইউজার “dashboard” route এ থাকে, তবে `active` class যোগ হয় (menu highlight হয়)।
- `route('dashboard')` → Laravel route helper, ড্যাশবোর্ডের লিঙ্ক তৈরি করে।
- আইকন (`tachometer`) = speedometer symbol.

---

## 🧩 ৫. CRUD Main Menu (Dropdown)

```html
@php
$currentRoute = request()->route()->getName();
@endphp
```

➡️ এই লাইনটি বর্তমান route-এর নাম নিয়ে `$currentRoute` ভেরিয়েবল-এ রাখে।  
উদাহরণ: “dashboard.crud-3.index”

---

```html
<li class="{{ Str::startsWith($currentRoute, 'dashboard.crud') ? 'active open' : '' }}">
    <a href="javascript:void(0);" class="dropdown-toggle">
        <i class="menu-icon fa fa-list"></i>
        <span class="menu-text"> CRUD </span>
        <b class="arrow fa fa-angle-down"></b>
    </a>
```

➡️ এটি মূল “CRUD” parent menu।

- `Str::startsWith($currentRoute, 'dashboard.crud')` → যদি বর্তমান route “dashboard.crud” দিয়ে শুরু হয় (যেমন: crud-1, crud-2 ইত্যাদি), তাহলে `active open` class দেওয়া হয়।
- `dropdown-toggle` → সাবমেনু টগল করার জন্য।
- `fa-angle-down` → নিচের দিকের তীরচিহ্ন আইকন।

---

## 🔽 CRUD Submenus

```html
<ul class="submenu">
    <li class="{{ Str::startsWith($currentRoute, 'dashboard.crud-1') ? 'active' : '' }}">
        <a href="{{ route('dashboard.crud-1.index') }}">
            <i class="menu-icon fa fa-caret-right"></i>
            CRUD - 1
        </a>
        <b class="arrow"></b>
    </li>
    ...
</ul>
```

➡️  
প্রত্যেক সাবমেনু আইটেম CRUD 1, CRUD 2, CRUD 3 … এর জন্য।  
যদি route `dashboard.crud-3` দিয়ে শুরু হয় → তখন “CRUD - 3” আইটেমে `active` ক্লাস যুক্ত হবে।

---

## 🧱 ৬. CRUD Alt Menu (আরেকটা Dropdown)

```html
<li class="">
    <a href="#" class="dropdown-toggle">
        <i class="menu-icon fa fa-desktop"></i>
        <span class="menu-text"> CRUD Alt </span>
        <b class="arrow fa fa-angle-down"></b>
    </a>
```

➡️  
আরেকটা dropdown মেনু (এখানে ডেমো আইটেম “Layouts” ইত্যাদি আছে)।  
সবগুলোই static demo link (যেমন: `top-menu.html`, `two-menu-1.html`)।

---

## 🧱 ৭. Other Pages Menu

```html
<li class="">
    <a href="#" class="dropdown-toggle">
        <i class="menu-icon fa fa-file-o"></i>
        <span class="menu-text">
            Other Pages
            <span class="badge badge-primary">5</span>
        </span>
        <b class="arrow fa fa-angle-down"></b>
    </a>
```

➡️  
আরেকটি dropdown মেনু (FAQ, Error 404, Error 500, Grid, Blank Page)।  
`badge badge-primary` → ৫ সংখ্যাটি নীল ব্যাজ আকারে দেখায়।

---

## 🔽 Submenu items

```html
<ul class="submenu">
    <li><a href="faq.html"><i class="menu-icon fa fa-caret-right"></i> FAQ</a></li>
    ...
</ul>
```

➡️  এইগুলো static লিঙ্ক, সাধারণত ডেমো বা template-এর অংশ।

---

## ⚙️ ৮. Sidebar Collapse Button

```html
<div class="sidebar-toggle sidebar-collapse" id="sidebar-collapse">
    <i id="sidebar-toggle-icon" class="ace-icon fa fa-angle-double-left ace-save-state"
        data-icon1="ace-icon fa fa-angle-double-left"
        data-icon2="ace-icon fa fa-angle-double-right"></i>
</div>
```

➡️  এটি sidebar collapse/expand করার বোতাম।

- ক্লিক করলে sidebar ছোট-বড় হয়।
- `ace-save-state` sidebar-এর অবস্থান মনে রাখে।
- `data-icon1` এবং `data-icon2` → টগল করলে আইকন পরিবর্তন হয় (বাম ↔ ডান তীর)।

---

# Complete Sidebar:

```php
<div id="sidebar" class="sidebar responsive ace-save-state">
    <script type="text/javascript">
        try {
            ace.settings.loadState('sidebar')
        } catch (e) {}
    </script>

    <div class="sidebar-shortcuts" id="sidebar-shortcuts">
        <div class="sidebar-shortcuts-large" id="sidebar-shortcuts-large">
            <button class="btn btn-success">
                <i class="ace-icon fa fa-signal"></i>
            </button>

            <button class="btn btn-info">
                <i class="ace-icon fa fa-pencil"></i>
            </button>

            <button class="btn btn-warning">
                <i class="ace-icon fa fa-users"></i>
            </button>

            <button class="btn btn-danger">
                <i class="ace-icon fa fa-cogs"></i>
            </button>
        </div>

        <div class="sidebar-shortcuts-mini" id="sidebar-shortcuts-mini">
            <span class="btn btn-success"></span>

            <span class="btn btn-info"></span>

            <span class="btn btn-warning"></span>

            <span class="btn btn-danger"></span>
        </div>
    </div><!-- /.sidebar-shortcuts -->

    <ul class="nav nav-list">
      {{-- Dashboard --}}
      <li class="{{ Request::is('dashboard') ? 'active' : '' }}">
        <a href="{{ route('dashboard') }}">
          <i class="menu-icon fa fa-tachometer"></i>
          <span class="menu-text"> Dashboard </span>
        </a>
        <b class="arrow"></b>
      </li>

      @php
      $currentRoute = request()->route()->getName();
      @endphp

      
        {{-- CRUD SECTION (1–6) --}}
      <li class="{{ request()->routeIs('dashboard.crud-1.*') || request()->routeIs('dashboard.crud-2.*') || request()->routeIs('dashboard.crud-3.*') || request()->routeIs('dashboard.crud-4.*') || request()->routeIs('dashboard.crud-5.*') || request()->routeIs('dashboard.crud-6.*') ? 'active open' : '' }}">
        <a href="#" class="dropdown-toggle">
          <i class="menu-icon fa fa-list"></i>
          <span class="menu-text"> CRUD </span>
          <b class="arrow fa fa-angle-down"></b>
        </a>

        <b class="arrow"></b>

        <ul class="submenu">
          {{-- CRUD 1–6 Simple --}}
          <li class="{{ request()->routeIs('dashboard.crud-1.*') ? 'active' : '' }}">
            <a href="{{ route('dashboard.crud-1.index') }}">
              <i class="menu-icon fa fa-caret-right"></i> CRUD - 1
            </a>
          </li>
          <li class="{{ request()->routeIs('dashboard.crud-2.*') ? 'active' : '' }}">
            <a href="{{ route('dashboard.crud-2.index') }}">
              <i class="menu-icon fa fa-caret-right"></i> CRUD - 2
            </a>
          </li>
          <li class="{{ request()->routeIs('dashboard.crud-3.*') ? 'active' : '' }}">
            <a href="{{ route('dashboard.crud-3.index') }}">
              <i class="menu-icon fa fa-caret-right"></i> CRUD - 3
            </a>
          </li>
          <li class="{{ request()->routeIs('dashboard.crud-4.*') ? 'active' : '' }}">
            <a href="{{ route('dashboard.crud-4.index') }}">
              <i class="menu-icon fa fa-caret-right"></i> CRUD - 4
            </a>
          </li>
          <li class="{{ request()->routeIs('dashboard.crud-5.*') ? 'active' : '' }}">
            <a href="{{ route('dashboard.crud-5.index') }}">
              <i class="menu-icon fa fa-caret-right"></i> CRUD - 5
            </a>
          </li>
          <li class="{{ request()->routeIs('dashboard.crud-6.*') ? 'active' : '' }}">
            <a href="{{ route('dashboard.crud-6.index') }}">
              <i class="menu-icon fa fa-caret-right"></i> CRUD - 6
            </a>
          </li>
        </ul>
      </li>



    {{-- ==========================
        BLOG MANAGEMENT (CRUD 7–10)
    =========================== --}}
    <li class="{{ (Str::startsWith($currentRoute, 'dashboard.crud-7') || Str::startsWith($currentRoute, 'dashboard.crud-8') || Str::startsWith($currentRoute, 'dashboard.crud-9') || Str::startsWith($currentRoute, 'dashboard.crud-10')) ? 'active open' : '' }}">
      <a href="#" class="dropdown-toggle">
        <i class="menu-icon fa fa-desktop"></i>
        <span class="menu-text"> Blog Management </span>
        <b class="arrow fa fa-angle-down"></b>
      </a>

      <b class="arrow"></b>

      <ul class="submenu">
        {{-- Category Section --}}
        <li class="{{ (request()->routeIs('dashboard.crud-7.*') || request()->routeIs('dashboard.crud-8.*') || request()->routeIs('dashboard.crud-9.*')) ? 'active open' : '' }}">
          <a href="#" class="dropdown-toggle">
            <i class="menu-icon fa fa-caret-right"></i>
            Categories
            <b class="arrow fa fa-angle-down"></b>
          </a>

          <b class="arrow"></b>
          <ul class="submenu">
            <li class="{{ request()->routeIs('dashboard.crud-7.*') ? 'active' : '' }}">
              <a href="{{ route('dashboard.crud-7.index') }}">
                <i class="menu-icon fa fa-caret-right"></i> Category
              </a>
            </li>

            <li class="{{ request()->routeIs('dashboard.crud-8.*') ? 'active' : '' }}">
              <a href="{{ route('dashboard.crud-8.index') }}">
                <i class="menu-icon fa fa-caret-right"></i> Sub-Category
              </a>
            </li>

            <li class="{{ request()->routeIs('dashboard.crud-9.*') ? 'active' : '' }}">
              <a href="{{ route('dashboard.crud-9.index') }}">
                <i class="menu-icon fa fa-caret-right"></i> Sub-Sub-Category
              </a>
            </li>
          </ul>
        </li>

        {{-- Posts --}}
        <li class="{{ request()->routeIs('dashboard.crud-10.*') ? 'active' : '' }}">
          <a href="{{ route('dashboard.crud-10.index') }}">
            <i class="menu-icon fa fa-caret-right"></i>
            Posts Management
          </a>
        </li>
      </ul>
    </li>

    {{-- ==========================
        OTHER PAGES SECTION
    =========================== --}}
    <li class="{{ Str::startsWith($currentRoute, 'dashboard.company') ? 'active open' : '' }}">
      <a href="#" class="dropdown-toggle">
        <i class="menu-icon fa fa-file-o"></i>
        <span class="menu-text">
          Other Pages
          <span class="badge badge-primary">1</span>
        </span>
        <b class="arrow fa fa-angle-down"></b>
      </a>

      <b class="arrow"></b>

      <ul class="submenu">
        <li class="{{ request()->routeIs('dashboard.company.*') ? 'active' : '' }}">
          <a href="{{ route('dashboard.company.index') }}">
            <i class="menu-icon fa fa-caret-right"></i>
            Company Settings
          </a>
        </li>
      </ul>
    </li>
    </ul>



    <div class="sidebar-toggle sidebar-collapse" id="sidebar-collapse">
        <i id="sidebar-toggle-icon" class="ace-icon fa fa-angle-double-left ace-save-state" data-icon1="ace-icon fa fa-angle-double-left" data-icon2="ace-icon fa fa-angle-double-right"></i>
    </div>
</div>
```

## 🧾 সারাংশ

|অংশ|কাজ|
|---|---|
|`sidebar` div|পুরো সাইডবারের wrapper|
|`ace.settings.loadState('sidebar')`|পূর্বের অবস্থান পুনরুদ্ধার|
|`sidebar-shortcuts`|চারটি রঙিন শর্টকাট বোতাম|
|Dashboard menu|ড্যাশবোর্ড লিঙ্ক ও Active অবস্থা|
|CRUD dropdown|একাধিক CRUD পেজের সাবমেনু|
|CRUD Alt / Other Pages|ডেমো মেনু সেকশন|
|sidebar-collapse|Sidebar টগল বোতাম|

---