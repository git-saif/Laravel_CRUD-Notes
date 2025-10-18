**Postman** দিয়ে তোমার API test করতে হলে নিচের step গুলো follow করে পুরো CRUD system সহজে টেস্ট করা যাবে। 

---

## ⚙️ Route Setup:

ধরে নেই, আমাদের route setup হলো :

`api.php:`
```php
Route::prefix('dashboard')->group(function () {

    Route::apiResources([
        'crud-1' => Crud1Controller::class,
    ]);
});
```

এবং তুমি app চালাচ্ছো:

```
php artisan serve
```

তাহলে base URL হবে  
👉 `http://127.0.0.1:8000`

তাহলে full API endpoint হবে  
👉 `http://127.0.0.1:8000/api/dashboard/crud-1`

---

## 🔹 এখন Postman দিয়ে প্রতিটি Route Test করার নিয়ম

| কাজ                                 | HTTP Method | URL                                            | Body / Data                                                                                                                                  | বর্ণনা                      |
| ----------------------------------- | ----------- | ---------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- |
| 🟢 **১. সব ডেটা দেখা**              | `GET`       | `http://127.0.0.1:8000/api/dashboard/crud-1`   | ❌ কোনো body নেই                                                                                                                              | সব রেকর্ড JSON আকারে দেখাবে |
| 🟢 **২. নির্দিষ্ট ID এর ডেটা দেখা** | `GET`       | `http://127.0.0.1:8000/api/dashboard/crud-1/1` | ❌ কোনো body নেই                                                                                                                              | ID=1 এর ডেটা দেখাবে         |
| 🟠 **৩. নতুন ডেটা যোগ করা**         | `POST`      | `http://127.0.0.1:8000/api/dashboard/crud-1`   | ✅ Body → JSON (raw) `json { "topic_name": "Laravel", "title": "API CRUD", "description": "Learning API CRUD", "status": "active" }`          | নতুন রেকর্ড তৈরি করবে       |
| 🔵 **৪. ডেটা আপডেট করা**            | `PUT`       | `http://127.0.0.1:8000/api/dashboard/crud-1/1` | ✅ Body → JSON (raw) `json { "topic_name": "Updated Laravel", "title": "Updated CRUD", "description": "Updated info", "status": "inactive" }` | ID=1 রেকর্ড আপডেট করবে      |
| 🔴 **৫. ডেটা মুছে ফেলা**            | `DELETE`    | `http://127.0.0.1:8000/api/dashboard/crud-1/1` | ❌ কোনো body নেই                                                                                                                              | ID=1 রেকর্ড ডিলিট করবে      |

---

## 🧭 Postman-এ কিভাবে সেট করবে

### **Step-1: (Create New Request )**

1. “+ New Tab” ক্লিক করতে হবে।
2. Method সেট করতে হবে (GET / POST / PUT / DELETE)
3. URL বসাও যেমন:
    
    ```
    http://127.0.0.1:8000/api/dashboard/crud-1
    ```
    

---

### **Step-2: (POST / PUT)**

1. “Body” Tab এর  -> “raw” সিলেক্ট করে ডান পাশে dropdown থেকে **JSON** সিলেক্ট করতে হবে।
2. তারপর নিচের মতো JSON লিখতে হবে:
  
    ```json
    {
      "topic_name": "Laravel",
      "title": "CRUD API",
      "description": "Test Create",
      "status": "active"
    }
    ```
    

---

### **Step-3: (Press Send button)**

1. Postman request পাঠাবে
2. নিচে Response section-এ JSON data দেখাবে  
    ✅ Success হলে নিচের মতো দেখাবে:
    
```json
    {
      "message": "Created successfully",
      "data": {
        "id": 1,
        "topic_name": "Laravel",
        "title": "CRUD API",
        "description": "Test Create",
        "status": "active"
      }
    }
```
---

## 🧩 Bonus: দ্রুত Debug এর টিপস

1. যদি `404 Not Found` আসে ➜ তোমার route বা prefix ঠিক আছে কিনা দেখো
2. যদি `419 Page Expired` আসে ➜ CSRF লাগবে না, কারণ `api.php` route CSRF check করে না
3. যদি `500 Server Error` আসে ➜ Controller method এর মধ্যে কোনো error আছে কিনা দেখো
---

## ✅ সারাংশ:

|Action|Method|URL|
|---|---|---|
|সব দেখা|GET|`/api/dashboard/crud-1`|
|একটার বিস্তারিত|GET|`/api/dashboard/crud-1/{id}`|
|নতুন যোগ|POST|`/api/dashboard/crud-1`|
|আপডেট|PUT|`/api/dashboard/crud-1/{id}`|
|ডিলিট|DELETE|`/api/dashboard/crud-1/{id}`|

---
