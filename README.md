قم بتجميع جميع الاكواد معا مع زكر ترتيب الفولدر 
ChatGPT said:
🚀 هيكلة المشروع وتجميع الأكواد الكاملة لمتجر الملابس الإلكتروني
📂 1. هيكلة مجلدات المشروع
bash
Copy
Edit
/ecommerce-project
│── /app
│   ├── /Http
│   │   ├── /Controllers
│   │   │   ├── ProductController.php
│   │   │   ├── ReviewController.php
│   │   │   ├── OrderController.php
│   │   ├── Models
│   │   │   ├── Product.php
│   │   │   ├── Review.php
│   │   │   ├── Order.php
│── /database
│   ├── /migrations
│   │   ├── 2025_xx_xx_create_products_table.php
│   │   ├── 2025_xx_xx_create_reviews_table.php
│   │   ├── 2025_xx_xx_create_orders_table.php
│── /resources
│   ├── /js
│   │   ├── /components
│   │   │   ├── ProductList.vue
│   │   │   ├── Cart.vue
│   │   │   ├── Checkout.vue
│   │   │   ├── OrderList.vue
│   │   ├── app.js
│   ├── /css
│   │   ├── app.css
│── /routes
│   ├── api.php
│── tailwind.config.js
│── package.json
│── webpack.mix.js
🔹 2. أكواد الـ Backend (Laravel)
📁 app/Models/Product.php
php
Copy
Edit
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    use HasFactory;
    protected $fillable = ['name', 'category', 'description', 'price', 'image'];
}
📁 app/Http/Controllers/ProductController.php
php
Copy
Edit
namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    public function index(Request $request)
    {
        $query = Product::query();
        if ($request->has('category')) {
            $query->where('category', $request->category);
        }
        if ($request->has('search')) {
            $query->where('name', 'LIKE', '%' . $request->search . '%');
        }
        return response()->json($query->get());
    }
}
📁 app/Models/Review.php
php
Copy
Edit
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Review extends Model
{
    use HasFactory;
    protected $fillable = ['user_id', 'product_id', 'rating', 'comment'];
}
📁 app/Http/Controllers/ReviewController.php
php
Copy
Edit
namespace App\Http\Controllers;

use App\Models\Review;
use Illuminate\Http\Request;

class ReviewController extends Controller
{
    public function store(Request $request)
    {
        $request->validate([
            'product_id' => 'required',
            'rating' => 'required|integer|min:1|max:5',
            'comment' => 'nullable|string',
        ]);

        Review::create([
            'user_id' => auth()->id(),
            'product_id' => $request->product_id,
            'rating' => $request->rating,
            'comment' => $request->comment,
        ]);

        return response()->json(['message' => 'تمت إضافة المراجعة']);
    }
}
📁 app/Http/Controllers/OrderController.php
php
Copy
Edit
namespace App\Http\Controllers;

use App\Models\Order;
use Illuminate\Http\Request;

class OrderController extends Controller
{
    public function index()
    {
        return response()->json(Order::all());
    }

    public function update(Request $request, $id)
    {
        $order = Order::findOrFail($id);
        $order->update(['status' => $request->status]);
        return response()->json(['message' => 'تم تحديث حالة الطلب']);
    }
}
🔹 3. أكواد الـ Frontend (Vue.js)
📁 resources/js/components/ProductList.vue
vue
Copy
Edit
<template>
  <div>
    <input v-model="search" placeholder="ابحث..." @input="fetchProducts" />
    <select v-model="category" @change="fetchProducts">
      <option value="">كل الفئات</option>
      <option value="ملابس رجالية">ملابس رجالية</option>
      <option value="ملابس نسائية">ملابس نسائية</option>
    </select>
    <div v-for="product in products" :key="product.id">
      <h2>{{ product.name }}</h2>
      <p>{{ product.category }}</p>
      <p>${{ product.price }}</p>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  data() {
    return { products: [], search: "", category: "" };
  },
  mounted() { this.fetchProducts(); },
  methods: {
    fetchProducts() {
      axios.get('/api/products', { params: { search: this.search, category: this.category } })
        .then(response => { this.products = response.data; });
    }
  }
};
</script>
📁 resources/js/components/Cart.vue
vue
Copy
Edit
<template>
  <div>
    <h1>عربة التسوق</h1>
    <div v-if="cart.length">
      <div v-for="item in cart" :key="item.id">
        <p>{{ item.product.name }} - الكمية: {{ item.quantity }}</p>
        <button @click="removeFromCart(item.id)">حذف</button>
      </div>
    </div>
    <p v-else>عربة التسوق فارغة</p>
  </div>
</template>
📁 resources/js/components/OrderList.vue
vue
Copy
Edit
<template>
  <div>
    <h1>إدارة الطلبات</h1>
    <table>
      <tr v-for="order in orders" :key="order.id">
        <td>رقم الطلب: {{ order.id }}</td>
        <td>المبلغ: ${{ order.total_price }}</td>
        <td>الحالة: {{ order.status }}</td>
        <td><button @click="updateStatus(order.id, 'تم الشحن')">تم الشحن</button></td>
      </tr>
    </table>
  </div>
</template>
🔹 4. API Routes
📁 routes/api.php
php
Copy
Edit
use App\Http\Controllers\ProductController;
use App\Http\Controllers\ReviewController;
use App\Http\Controllers\OrderController;

Route::get('/products', [ProductController::class, 'index']);
Route::post('/reviews', [ReviewController::class, 'store']);
Route::get('/orders', [OrderController::class, 'index']);
Route::patch('/orders/{id}', [OrderController::class, 'update']);
🔹 5. تشغيل المشروع
تثبيت الحزم
bash
Copy
Edit
npm install
composer install
php artisan migrate
تشغيل السيرفر
bash
Copy
Edit
npm run dev
php artisan serve
