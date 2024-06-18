# Laravel Multi-Authentication with Middleware

This project demonstrates how to implement multi-authentication in a Laravel application. It uses middleware to restrict access to certain routes based on user roles, specifically preventing regular users from accessing the admin dashboard and preventing admins from accessing the user dashboard via URLs.

## Prerequisites

- PHP >= 7.3
- Composer
- Laravel >= 8

## Setup

1. **Clone the repository:**

    ```sh
    git clone https://github.com/your-repository.git
    cd your-repository
    ```

2. **Install dependencies:**

    ```sh
    composer install
    ```

3. **Set up environment variables:**

    Copy the `.env.example` file to `.env` and fill in your database and other necessary configurations.

    ```sh
    cp .env.example .env
    ```

4. **Generate application key:**

    ```sh
    php artisan key:generate
    ```

5. **Run migrations:**

    ```sh
    php artisan migrate
    ```

## Middleware Setup

### Admin Middleware

This middleware ensures that only admin users can access the admin dashboard.

1. **Create the Admin middleware:**

    ```sh
    php artisan make:middleware Admin
    ```

2. **Implement the Admin middleware:**

    ```php
    // app/Http/Middleware/Admin.php
    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;
    use Illuminate\Support\Facades\Auth;

    class Admin
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            if (Auth::user()->usertype != 'admin') {
                return redirect('dashboard');
            }

            return $next($request);
        }
    }
    ```

### User Middleware

This middleware ensures that admin users cannot access the user dashboard.

1. **Create the User middleware:**

    ```sh
    php artisan make:middleware User
    ```

2. **Implement the User middleware:**

    ```php
    // app/Http/Middleware/User.php
    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;
    use Illuminate\Support\Facades\Auth;

    class User
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            if (Auth::user() && Auth::user()->usertype == 'admin') {
                return redirect('admin/dashboard');
            }

            return $next($request);
        }
    }
    ```

### Register Middleware

Register both middlewares in `bootstrap/app.php`.

```php
->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'admin' => \App\Http\Middleware\Admin::class,
            'user' => \App\Http\Middleware\User::class,
        ]);
    })
