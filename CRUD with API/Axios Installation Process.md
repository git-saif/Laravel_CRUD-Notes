**Laravel + Vite প্রজেক্টে Axios install ও ব্যবহার করার process:**

---

### 📝 Axios Install & Use (Short Notes)

1. **Install Axios via NPM**

```bash
npm install axios
```

2. **Import Axios in JS**  
    `resources/js/app.js` (বা অন্য main JS file) এ লিখতে হবে:
    

```js
import axios from 'axios';
window.axios = axios; // Optional, globally useable
```

3. **Use Axios**
    

```js
axios.get('/api/crud1')
  .then(res => console.log(res.data))
  .catch(err => console.error(err));
```

4. **Build JS (Vite)**
    

```bash
npm run dev    # Development
npm run build  # Production
```

5. **Include JS in Blade**
    

```blade
@vite('resources/js/app.js')
```

✅ Done! এখন Blade বা frontend এ Axios ব্যবহার করে API থেকে JSON fetch করা যাবে।

---
