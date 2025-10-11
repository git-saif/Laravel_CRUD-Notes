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