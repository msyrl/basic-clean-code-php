# *Clean Code*

## Pengenalan

*Clean Code* merupakan suatu pedoman yang dikemukakan oleh Robert C. Martin dalam bukunya yang berjudul *Clean Code: A Handbook of Agile Software Craftsmanship*. Pedoman ini bertujuan untuk menghasilkan perangkat lunak yang *readable* (mudah dibaca), *reusable* (dapat digunakan kembali), dan *refactorable* (dapat direstrukturisasi).

Paham akan pedoman ini tidak akan langsung membuat kita menjadi *software developer* yang baik, dan menggunakan pedoman ini bertahun-tahun bukan berarti kita tidak akan membuat kesalahan. Setiap baris kode diawali dengan *draft* awal. Proses penulisan kode yang bersih dapat dianalogikan seperti pembuatan mangkuk yang prosesnya diawali dengan tanah liat basah, lalu dibentuk, dan pada akhirnya berbentuklah sebuah mangkuk. Maka dari itu, perlu proses untuk menulis kode yang bersih.

## Apa sih itu *clean code*?

Yaitu kode yang ditulis dengan format yang benar, disusun dengan baik dan rapi, dan mengikuti standar.

## Mengapa kode harus ditulis dengan format yang benar, disusun dengan baik dan rapi, dan mengikuti standar?

1. Agar kode program mudah dimengerti programmer lain.
2. Agar kode program mudah dibaca dan ditelusuri.
3. Agar memperlancar proses pengembangan perangkat lunak secara tim.
4. Agar proses *debug* lebih mudah dan cepat dilakukan.

## Bagaimana kita menulis kode yang bersih?

#### Gunakan nama yang bermakna dan dapat diucapkan.

**Buruk:**

```php
$ymd = date('Y-m-d');
```

**Baik:**

```php
$currentDate = date('Y-m-d');
```

#### Gunakan kosakata yang sama untuk data yang sama.

**Buruk:**

```php
getUserInfo();
getUserData();
getUserRecord();
getUserProfile();
```

**Baik:**

```php
getUser();
```

#### Gunakan nama yang mudah untuk dicari.

**Buruk:**

```php
// Apa itu 1?
if ($user->hasAccess(1)) {
    // ...
}
```

**Baik:**

```php
class Permissions
{
    const CREATE = 1;
    const READ = 2;
    const EDIT = 3;
    const DELETE = 4;
}

if ($user->hasAccess(Permissions::CREATE)) {
    // ...
}
```

#### Hindari percabangan yang terlalu dalam.

**Buruk:**

```php
function isShopOpen($day): bool
{
    if ($day) {
        if (is_string($day)) {
            $day = strtolower($day);
            if ($day === 'friday') {
                return true;
            } elseif ($day === 'saturday') {
                return true;
            } elseif ($day === 'sunday') {
                return true;
            }
            return false;
        }
        return false;
    }
    return false;
}
```

**Baik:**

```php
function isShopOpen(string $day): bool
{
    if (empty($day)) {
        return false;
    }

    $openingDays = ['friday', 'saturday', 'sunday'];

    return in_array(strtolower($day), $openingDays, true);
}
```

#### Hapus konteks yang tidak perlu.

**Buruk:**

```php
class User
{
    public $userName;

    public $userEmail;

    public $userPassword;

    //...
}
```

**Baik:**

```php
class User
{
    public $name;

    public $email;

    public $password;

    //...
}
```

#### Gunakan *coalescing operator*.

**Buruk:**

```php
if (isset($_GET['name'])) {
    $name = $_GET['name'];
} elseif (isset($_POST['name'])) {
    $name = $_POST['name'];
} else {
    $name = 'nobody';
}
```

**Baik:**
```php
declare(strict_types=1);

$name = $_GET['name'] ?? $_POST['name'] ?? 'nobody';
```

#### Enkapsulasi kondisional.

**Buruk:**

```php
if ($post->status === 'published') {
    // ...
}

```

**Baik:**

```php
if ($post->isPublished()) {
    // ...
}
```

#### Hapus kode yang sudah tidak digunakan.

**Buruk:**

```php
// function oldRequestModule(string $url): void
// {
//   // ...
// }

function requestModule(string $url): void
{
    // ...
}

$request = requestModule($requestUrl);
inventoryTracker('apples', $request, 'www.inventory-awesome.io');
```

**Baik:**

```php
function requestModule(string $url): void
{
    // ...
}

$request = requestModule($requestUrl);
inventoryTracker('apples', $request, 'www.inventory-awesome.io');
```

#### Ekstrak dan dekomposisi proses

**Buruk:**

```php
class DashboardController {

    public function index(Request $request)
    {
        $userTotal = User::where('type', 'customer')->count();
        $userToday = User::where('type', 'customer')->whereDate('created_at', date('Y-m-d'))->count();
        $userActive = User::where('type', 'customer')->where('is_active', true)->count();
        $userSuspended = User::where('type', 'customer')->where('is_active', false)->count();

        $merchantTotal = Merchant::published()->count();
        $merchantToday = Merchant::published()->whereDate('created_at', date('Y-m-d'))->count();
        $merchantActive = Merchant::published()->where('is_active', true)->count();
        $merchantSuspended Merchant::published()->where('is_active', false)->count();

        // other codes

      return response()->json([...]);
    }
}
```

**Baik:**

```php
class DashboardController {

    public function index(Request $request)
    {
        $user = $this->userSummary();

        // other codes

      return response()->json([...]);
    }

    private function userSummary(): array
    {
        $user['total'] = User::where('type', 'customer')->count();
        $user['today'] = User::where('type', 'customer')->whereDate('created_at', date('Y-m-d'))->count();
        $user['active'] = User::where('type', 'customer')->where('is_active', true)->count();
        $user['suspended'] = User::where('type', 'customer')->where('is_active', false)->count();

        return $user;
    }

    private function merchantSummary(): array
    {
        $merchant['total'] = Merchant::published()->count();
        $merchant['today'] = Merchant::published()->whereDate('created_at', date('Y-m-d'))->count();
        $merchant['active'] = Merchant::published()->where('is_active', true)->count();
        $merchant['suspended'] Merchant::published()->where('is_active', false)->count();

        return $merchant;
    }
}
```

#### *Don't Repeat Yourself (DRY)*

**Buruk:**

```php
class ProductController
{
    public function store(Request $request)
    {
        $data = $request->validate([
            'name' => [
                'required',
                'string',
            ],
            'description' => [
                'required',
                'string',
            ],
            'price' => [
                'required',
                'numeric',
            ],
        ]);

        $product = Product::create($data);

        return response()->json([...]);
    }

    public function update(Request $request, Product $product)
    {
        $data = $request->validate([
            'name' => [
                'required',
                'string',
            ],
            'description' => [
                'required',
                'string',
            ],
            'price' => [
                'required',
                'numeric',
            ],
        ]);

        $product->update($data);

        return response()->json([...]);
    }
}
```

**Baik:**

```php
class ProductController
{
    public function store(Request $request)
    {
        $product = Product::create($this->validatedRequest());

        return response()->json([...]);
    }

    public function update(Request $request, Product $product)
    {
        $product->update($this->validatedRequest());

        return response()->json([...]);
    }

    private function validatedRequest()
    {
        return request()->validate([
            'name' => [
                'required',
                'string',
            ],
            'description' => [
                'required',
                'string',
            ],
            'price' => [
                'required',
                'numeric',
            ],
        ]);
    }
}
```

#### Referensi

- https://github.com/jupeter/clean-code-php
- https://medium.com/dot-lab/5-cara-mudah-menerapkan-clean-code-b2e0ec1b860e
- https://medium.com/@nihon_rafy/clean-code-in-php-f5ba499049be