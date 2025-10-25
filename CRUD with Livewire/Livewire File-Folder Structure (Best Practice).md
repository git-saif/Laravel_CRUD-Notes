## Livewire View File / Folder Structure:

```pgsql
resources/
└── views/
    ├── components/              <-- Livewire page components layout / partials
    │   ├── layouts/
    │   │   └── app.blade.php    <-- main layout
    │   └── partials/
    │       ├── header.blade.php
    │       ├── sidebar.blade.php
    │       └── footer.blade.php
    └── livewire/                 <-- Livewire component views
        ├── dashboard.blade.php
        ├── Crud-1/
        │   ├── main.blade.php
        │   └── components
        │          ├── index.blade.php
        │          ├── create-edit.blade.php
        │          └── view.blade.php
        └── users/
            ├── list.blade.php
            └── edit.blade.php
```
____
