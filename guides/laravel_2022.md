# Routes

```web.php``` for basic routing.

```api.php``` for creating APIs.

# Blade

You can use ```{{}}``` for calling data.

You can also use directives like ```@foreach``` and ```@unless```.

# Model Creation
## Basic (Foundational)
Inside ```app/Models/```, you can create a model php file.

The syntax looks something like this:

```php
// app/Models/Listing.php
namespace App\Models;

class Listing {
    public static function all() {
        return <INSERT_ARRAY_DATA_HERE>
    }
    public static function find($id) {
        $listings = self::all();

        forreach($listings as $listing) {
            if($listing[$id] == $id) {
                return $listing;
            }
        }
    }
}
```

## Eloquent ORM
```terminal
php artisan make:model Listing
```

Laravel automatically generates a listing model for you with ```all()``` and ```find($id)``` inherited from ```Model```.

## Using the Listing Model
```php
// routes/web.php

...
use App\Models\Listing;

Route::get('/', function () {
    return view('listings',[
        'listings' => Listing::all()
    ]);
});

Route::get('/listings/id', function($id) {
    return view('listing', [
        'listing' => Listing::find($id);
    ]);
});
```

# Database
1. Update ```config/database.php``` based on the databse you will use.
2. Update ```.env``` based on the settings of your database.

## Updating Data in the Database
This command will update the database with the changes made in ```database/seeders/DatabaseSeeder.php```.

**Add**

```php
Listing::create([<INSERT_DATA_HERE>])
```

**Command**

```terminal
php artisan migrate:refresh --seed
```

## Factory Creation
```terminal
php artisan make:factory ListingFactory
```

This creates ```database/factories/ListingFactory.php```. It allows you to define default factory values.

**Using Factory in ```DatabaseSeeder.php```**

```php
Listing::factory(6)->create();
```

Run the migrate seed command after.

> [!NOTE]
> Read the documentation to view the different functions for creating dummy data.

# Creating Layouts
Inside ```resources/views/```, create ```layout.blade.php``` file.

```html
<!-- remember that this is layout.blade.php -->
...
<body>
    <!-- Blade files that extends this layout will appear here. -->
    @yield('content')
</body
...
```

**Child Blade File**

```php
@extends('layout');

@section('content')
...
@endsection
```

# Adding Themes

In the ```public``` folder, you can add an ```image``` folder.

**Using Images**
```php
<img src="{{asset('images/no-image.png')}}" />
```

> [!NOTE]
> For collections, you can use ```$listing->title``` instead of ```$listing['title']```.

# Template Partials
Create a ```resources/views/partials``` folder to trim down your code.

Inside the ```paritals``` folder, create a ```_*.blade.php``` file. Paste the partial code.

**Using Template Partials**
```php
@include('partials._hero')
```

# Route Model Building
```php
Route::get('/listings/id{id}', function($id){
    $listing = Listing::find($id);

    if($listing) {
        return view('listing', [
            'listing' => $listing
        ]);
    } else {
        abort('404');
    }
})
```

# Blade Components
Create a ```resources/views/components``` folder.

Inside, you can create components with props inside.

```php
// resources/views/components/listing-card.blade.php
@props(['listing'])

<x-card>
    {{ $listing->location }}
</x-card>
```

```php
// resources/views/listings.blade.php
...

@foreach($listings as $listing)
    <x-listing-card :listing="$listing" />
@endforeach
```

```php
// resources/views/components/card.blade.php
<div {{$atttributes->merge(['class' => 'bg-gray-50 border border-gray-200 rounded p-6'])}}>
    {{ $slot }}
</div>
```

# Controllers

You can add controllers in ```app/Http/Controllers```.

**Example**

```php
<?php

namespace App\Http\Controllers;

use App\Models\Listing;
use Illuminate\Http\Request;

class ListingController extends Controller
{
    // Show all listings
    public function index(){
        return view('listings', [
            'listings' => Listing:all()
        ])
    }

    // Show single listing
    public function show(Listing $listing) {
        return view('listings', [
            'listing' => $listing 
        ])
    }
}
```

```php
// routes/web.php

...

// All Listings
Route::get('/', [ListingController::class, 'index']);

// Single Listing
Route::get('/listings/{listing}', [ListingController::class, 'show']);

// Common Resource Routes:
// index  - Show all listings
// show - Show single listing
// create - Show form to create new listing
// store - Store new listing
// edit - Show form to edit listing
// update - Update listing
// destroy - Delete listing
```

# Resource Method Naming
Create a ```resources/views/listings``` folder.

Then, you can implement this:

| Old Name | New Name |
| --- | --- |
| ```listings.blade.php``` | ```index.blade.php``` |
| ```listing.blade.php``` | ```show.blade.php``` |

Then, you can update ```ListingController.php```.

```php
class ListingController extends Controller
{
    // Show all listings
    public function index(){
        return view('listings.index', [
            'listings' => Listing:all()
        ])
    }

    // Show single listing
    public function show(Listing $listing) {
        return view('listings.show', [
            'listing' => $listing 
        ])
    }
}
```

# Layout Component
```php
// resources/views/components/layout.blade.php
...
<main>
    {{$slot}}
</main>
...
```

```php
// resources/show.blade.php
<x-layout>
...
</x-layout>
```

# Filters
```php
// Http/Models/Listings.php

...

class Listing extends Model {
    use HasFactory;

    public function scopeFilter($query, array $filters) {
        if($filters['tag'] ?? false) {
            $query->where('tags', 'like', '%', . request('tag' . '%'))
        }
    }
}
```

```php
// Http/Controllers/ListingController.php

class ListingController extends Controller
{
    public function index() {
        return voew('listings.index', [
            'listings' => Listing::latest()->filter(request(['tag'])))->get()
        ])
    }
    ...
}
```

# Create Listing Form
```php
// Http/Controllers/ListingController.php

public function create() {
    return view('listings.create');
}
```

```php
// routes/web.php

Route::get('listings/create',
[ListingController::class, 'create']);
```

# Validation and Store New Listing
Use ```POST``` in forms.

Add ```@csrf``` inside your ```<form>``` tag to prevent cross site scripting.

```php
// resources/views/listings/create.blade.php
<form method="POST" actions="/listngs">
@csrf

...
</form>
```

```php
// routes/web.php

Route::post('listings',
[ListingController::class, 'store']);
```

```php
// Http/Controllers/ListingController.php

public function store(Request $request) {
    // view available validation attributes in documentation
    $formFields = $request->validate([
        'title' => 'required',
        'company' => ['required', Rule::unique('listings', 'company')]
        'location' => 'required',
        'website' => 'required',
        'email' => ['required', 'email']
        'tags' => 'required',
        'description' => 'required'
    ]);

    Listing::create($formFields);

    return redirect('/');
}
```


```php
// resources/views/listings/create.blade.php
<form method="POST" actions="/listngs">
@error('company')
    <p class="text-red-500 text-xs mt-1">{{ $message }}</p>
@enderror
// add this to every field
</form>
```

# Mass Assignment Rule
```php
// Models/Listings.php
// $fillable = properties that can be mass assignable. Anything not in the array cannot be mass assigned. $guarded = properties that cannot be mass assignable, allowing all others through.

class Listing extends Model
{
    use HasFactory;

    protected $fillable = ['title', 'company', 'location', 'website', 'email', 'description', 'tags'];

    ...
}
```

# Flash Messages
```php
// Http/Controllers/ListingController.php

class ListingController extends Controller
{
    ...

    // Method 1 - need to import session
    // Session:flash('message', 'Listing Created');

    // Method 2
    return redirect('/')->with('message', 'Listing created succesfully!');

    // message can be success, error, etc.
    ...
}
```

```php
// resources/views/components/flash-message.blade.php
@if(session()->has('message'))
<div class="...">
    <p>
        {{session('message')}}
    </p>
</div>
@endif

// you can use apline.js to make the message disappear.
```

# Keeping old data in form
Inside ```<input />```, add the attribute ```value``` with the value ```{{old('company')}}```.

Modify other fields as needed.

# Pagination
```php
// Https/Controllers/ListingController.php
class ListingController extends Controller
{
    public function index() {
        return view('listings.index', [
            'listings' => Listing::latest()->filter(request(['tag', 'search']))->paginate(2)
        ])
        // you can also use 'simplePaginate' to remove numbers
    }

    ...
}
```

```php
// resources/views/listings/index.blade.php
...
<div>
    {{ $listings->links() }}
</div>
```

# File Upload

# Edit Listing
```php
// route/web.php
Route::get('listings/{listing}/edit', [ListingControler::class, 'edit']);
```

```php
// Http/Contollers/ListingController.php
public class ListingController extends Controller {
    ...

    public function edit(Listing $listing) {
        return view('listings.edit', ['listing' => $listing]);
    }
}
```

```php
// resources/views/listings/edit.blade.php
// copy the create form
<form method="POST"...>
@csrf
@method('PUT')

<input ... value="{{$listing->location}}" />
</form>
```

```php
// Http/Contollers/ListingController.php
public class ListingController extends Controller {
    ...

    public function update(Listing $listing) {
        // view available validation attributes in documentation
        $formFields = $request->validate([
            'title' => 'required',
            'company' => ['required']
            'location' => 'required',
            'website' => 'required',
            'email' => ['required', 'email']
            'tags' => 'required',
            'description' => 'required'
        ]);

        $listing->create($formFields);

        return back('')->with('message', 'Listing updated succesfully!');
    }
}
```
```php
// route/web.php
Route::put('listings/{listing}/edit', [ListingControler::class, 'update']);
```

# Delete Listing
```php
// resources/views/listings/show.blade.php
// copy the create form
<form method="POST" action="listings/{{$listing->id}}">
@csrf
@method('DELETE')

</form>
```

```php
// route/web.php
Route::delete('listings/{listing}/', [ListingControler::class, 'delete']);
```

```php
// Http/Contollers/ListingController.php
public class ListingController extends Controller {
    ...

    public function delete(Listing $listing) {
        $listing->delete();

        return back('')->with('message', 'Listing deleted succesfully!');
    }
}
```
# User Registration
```php
// route/web.php
 
// import controller

Route::get('/register', [UserController::class, 'create']);
```

```terminal
php artisan make:controller UserController
```

Follow the standard process for creating users.


```php
// Http/Controllrs/UserController.php

public class UserController extends Controller
{
    // Create user code

    @auth()->login($user);

    return redirect('/')->with('message', 'User Created and logged in');
}
```

# Auth Links
Use the **auth** directive to show element only if you're logged in.

**Sample Use Cases**
```php
@auth
Welcome {{ auth()->user()->name }}
@else
Login
@endauth
```

# User Logout
Create a logout button in the templates.

```php
// route/web.php
 
// import controller

Route::post('/logout', [UserController::class, 'logout']);
```

```php
// Http/Controllrs/UserController.php

public class UserController extends Controller
{
    public function logout(Request $request) {
        auth()->logout();

        $request->session()->invalidate();
        $request->session()->regenerateToken();

        return redirect('/')->with('message', 'You have been logged out!')
    }
}
```

# User Login
```php
// route/web.php
 
// import controller

Route::get('/login', [UserController::class, 'login']);

Route::post('/users/authenticate');
```

```php
<form method="POST" action="users/authetnicate">
@csrf
...
</form>
```

```php
// Http/Controllrs/UserController.php

public class UserController extends Controller
{
    public function login() {
        return view('users.login');
    }

    public function authenticate(Request $request) {
        $formFields = $request->validate([
            'email' => ['required', 'email'],
            'password' => 'required'
        ])

        if(auth()->attempt($formFields)) {
            $request->session()->regenerate();

            return redirect('/')->with('message', 'You are now logged in!');
        })

        return back()->withErrors(['email' -> 'Invalid Credentials']->onlyInput('email');
    }
}
```

# Auth and Guest Middleware
```php
// route/web.php

Route...->middlewate('auth');

Route::get('/login', [UserController::class, 'login'])->name('login');
```

```php
// Http/Middleware/Authenticate.php
// Update this file when you want to update the Authentication Middleware
```

# Tinkering
Interact with the database

```terminal
php artisan tinker
```
# User Authorization