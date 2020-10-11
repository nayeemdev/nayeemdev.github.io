---
layout: post
title:  "How to create multi authentication with multiple table in Laravel"
date:   2019-12-30
excerpt: "Laravel, auth"
tag:
- laravel 
- auth
- multi
- guard
comments: true
---

![Laravel Multi Auth](https://miro.medium.com/max/640/1*32ssDgqEHx3JJM28nf1-0w.png)    

* GoTo this [Link](https://medium.com/@nayeemdev/how-to-create-multi-authentication-with-multiple-table-in-laravel-dbc43e229f9f) for read it at medimum

* Dont forget to clap and response to this.
 


How to Create Multi Authentication with Multiple Table in Laravel
=================================================================

[![Nayeem Hossain](https://miro.medium.com/fit/c/96/96/1*JGSU3lXhMgW3RE0lKKu9LQ.jpeg)](https://medium.com/@nayeemdev?source=post_page-----dbc43e229f9f--------------------------------)[Nayeem Hossain](https://medium.com/@nayeemdev?source=post_page-----dbc43e229f9f--------------------------------)Follow[Dec 30, 2019](https://medium.com/@nayeemdev/how-to-create-multi-authentication-with-multiple-table-in-laravel-dbc43e229f9f?source=post_page-----dbc43e229f9f--------------------------------) · 3 min read

We are working with laravel version 5.8. By default laravel authentication comes up with users table. Today we are going to authenticate by admins table rather than users table. Migrations and Creating admins table let’s start with create the migration file for our authentication table. Run the following command from terminal for creating the migration and model.

```
php artisan make:model Admin -m
```

Now change the migration file’s up method with necessary fields like the following.

```
public function up()  
{  
    Schema::create('admins', function (Blueprint $table) {  
       $table->increments('id');  
       $table->string('email')->unique();  
       $table->string('password');  
       $table->rememberToken();              
       $table->timestamps();  
    });  
}
```

Then migrate it by running the following command.

```
php artisan migrate
```

Take a look at the Admin model. Make sure the model class(Admin) must extends from Authenticatable(Auth) class and use Notifiable trait class.

```
use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class Admin extends Authenticatable
{
    use Notifiable;

    protected $table = 'admins';

    protected $fillable = ['email',  'password'];

    protected $hidden = ['password',  'remember_token'];


}
```

Now we are going to add custom Guard setup in the config/auth.php file. By default laravel use “web” guard. We are going to add our custom guard named “admin”. Take a look following code.

```
'guards' => [

        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'admin'  => [
          'driver'  => 'session',
          'provider' => 'admins',
        ],

    ],


 'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\User::class,
        ],

        'admins' => [
            'driver' => 'eloquent',
            'model'  => App\Admin::class,
        ],
]
```

Now time to add route for admin login.

```
Route::get('/admin/login', 'Auth\AdminLoginController@showLoginForm')->name('admin.login');
Route::post('/admin/login', 'Auth\AdminLoginController@login')->name('admin.login.post');
Route::post('/admin/logout', 'Auth\AdminLoginController@logout')->name('admin.logout');
//Admin Home page after login
Route::group(['middleware'=>'admin'], function() {
    Route::get('/admin/home', 'Admin\HomeController@index');
})
```

Now we are going to add middleware for protect our admin route. By the run following command create a middleware.

```
php artisan make:middleware RedirectIfNotAdmin
```

Add the following code to this middleware.

```
<?php
namespace App\Http\Middleware;
use Closure;
class RedirectIfNotAdmin
{
    public function handle($request, Closure $next, $guard="admin")
    {
        if(!auth()->guard($guard)->check()) {
            return redirect(route('admin.login'));
        }
        return $next($request);
    }
}
```

Then add the middleware class in kernel.php for resister the middleware.

```
protected $routeMiddleware = [
 'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
 'can' => \Illuminate\Auth\Middleware\Authorize::class,
 'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
 ...............
 // your custom middleware
 'admin' => \App\Http\Middleware\RedirectIfNotAdmin::class,
];
```

For admin login make the admin login blade file and the following codes are-

```
@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
<form class="form-horizontal" method="POST" action="{{ route('admin.login.post') }}">
    @csrf

    <div class="form-group">
        <label for="email" class="col-md-4">E-Mail</label>

        <div class="col-md-6">
            <input id="email" type="email" class="form-control" name="email" value="{{ old('email') }}" required autofocus>
        </div>
    </div>

    <div class="form-group">
        <label for="password" class="col-md-4">Password</label>

        <div class="col-md-6">
            <input id="password" type="password" class="form-control" name="password" required>
        </div>
    </div>

    <div class="form-group">
        <div class="col-md-6 col-md-offset-4">
            <div class="checkbox">
                <label>
                    <input type="checkbox" name="remember" {{ old('remember') ? 'checked' : '' }}> Remember Me
                </label>
            </div>
        </div>
    </div>

    <div class="form-group">
        <div class="col-md-8 col-md-offset-4">
            <button type="submit" class="btn btn-primary">
                Login
            </button>
        </div>
    </div>
</form>
```

Then make the AdminLoginController in Auth folder by the run following command.

```
php artisan make:controller Auth/AdminLoginController
```

Now setup our AdminLoginController we need to specify which guard we want to use.

```
use App\Http\Controllers\Controller;
use Illuminate\Foundation\Auth\AuthenticatesUsers;
use Illuminate\Support\Facades\Auth;

class AdminLoginController extends Controller
{

    use AuthenticatesUsers;

    protected $redirectTo = '/admin/home';

    public function __construct()
    {
      $this->middleware('guest')->except('logout');
    }

    public function guard()
    {
     return Auth::guard('admin');
    }
    public function showLoginForm()
    {
        return view('auth.admin_login');
    }
 }
```

And Finally, we have done our multi-level authentication with multiple table and custom guard.
