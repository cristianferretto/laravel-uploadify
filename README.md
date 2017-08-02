# Laravel Uploadify

Uploadify is a library for Laravel that handles image uploading with automatic renaming, showing thumbnail image with custom routes and more. All that is available over Eloquent models.

[![Build For Laravel](https://img.shields.io/badge/Built_for-Laravel-orange.svg)](https://styleci.io/repos/79834672)
[![Latest Stable Version](https://poser.pugx.org/fsasvari/laravel-uploadify/v/stable)](https://packagist.org/packages/fsasvari/laravel-uploadify)
[![Latest Unstable Version](https://poser.pugx.org/fsasvari/laravel-uploadify/v/unstable)](https://packagist.org/packages/fsasvari/laravel-uploadify)
[![Total Downloads](https://poser.pugx.org/fsasvari/laravel-uploadify/downloads)](https://packagist.org/packages/fsasvari/laravel-uploadify)
[![License](https://poser.pugx.org/fsasvari/laravel-uploadify/license)](https://packagist.org/packages/fsasvari/laravel-uploadify)

## Installation

### Step 1: Install package

To get started with Laravel Uploadify, execute Composer command to add the package to your composer.json project's dependencies:

```
composer require fsasvari/laravel-uploadify
```

Or add it directly by copying next line into composer.json:

```
"fsasvari/laravel-uploadify": "0.1.*"
```

And then run composer update:

```
composer update
```

### Step 2: Service Provider and Facade

After installing the Laravel Uploadify library, register the `Uploadify\Providers\UploadifyServiceProvider` in your `config/app.php` configuration file:

```php
'providers' => [
    // Application Service Providers...
    // ...

    // Other Service Providers...
    Uploadify\Providers\UploadifyServiceProvider::class,
    // ...
],
```

Optionally, you can add alias to `Uploadify` facade:

```php
'aliases' => [
    'Uploadify' => Uploadify\Facades\Uploadify::class,
];
```

### Step 3: Configuration

We need copy the configuration file to our project.

```
php artisan vendor:publish --tag=uploadify
```

### Step 4: Symbolic link

If you have not yet created symbolic link in project, we need to create link between `public` and `storage` directories. We can use built in Laravel function `storage:link` which will create link between `public/storage` and `storage/app/storage` directories.

```
php artisan storage:link
```

Or use Windows function for custom storage link:

```
mklink /d "c:\path-to-project\project-name\public\custom-directory-name" "c:\path-to-project\project-name\storage\app\custom-directory-name" // custom-directory-name could be "storage", "upload"...
```

Or use Unix function for custom storage link:

```
ln -s /path-to-project/project-name/storage/app/custom-directory-name /path-to-project/project-name/public/custom-directory-name // custom-directory-name could be "storage", "upload"...
```

### Step 5: Models

You need to include `UploadifyTrait` trait in your Eloquent models.

#### Files

If you need to show simple files (pdf, doc, zip...) in Eloquent model, you need to define `$files` property with database field name as key and `path` as array value which is required. Also, `disk` value is optional and it will be taken from default disk value from configuration.

```php
<?php

namespace App;

use Uploadify\Traits\UploadifyTrait;

class Car extends Eloquent
{
    use UploadifyTrait;

    /**
     * List of uploadify files
     *
     * @var array
     */
    protected $files = [
        'upload_information' => ['path' => 'documents/information/'],
        'upload_specification' => ['path' => 'documents/specification/', 'disk' => 's3'],
    ];
}
```

#### Images

If you need to show images (jpg, png, gif...) in Eloquent model, you need to define `$images` property with database field name as key and paths as array values (`path` and `path_thumb`). `path` value is required, but `path_thumb` is not. Use `path_thumb` only if path to thumb images is different then default one (we always use `thumb/` prefix on defined `path` value). Also, `disk` value is optional and it will be taken from default disk value from configuration.

```php
<?php

namespace App;

use Uploadify\Traits\UploadifyTrait;

class User extends Eloquent
{
    use UploadifyTrait;

    /**
     * List of uploadify images
     *
     * @var array
     */
    protected $images = [
        'upload_cover' => ['path' => 'images/cover/'],
        'upload_avatar' => ['path' => 'images/avatar/', 'path_thumb' => 'images/avatar-small/', 'disk' => 's3'],
    ];
}
```

#### Files and Images combined

You can also combine files and images into one Eloquent model:

```php
<?php

namespace App;

use Uploadify\Traits\UploadifyTrait;

class Car extends Eloquent
{
    use UploadifyTrait;

    /**
     * List of uploadify files
     *
     * @var array
     */
    protected $files = [
        'upload_information' => ['path' => 'documents/information/'],
        'upload_specification' => ['path' => 'documents/specification/'],
    ];

    /**
     * List of uploadify images
     *
     * @var array
     */
    protected $images = [
        'upload_cover' => ['path' => 'images/cover/'],
    ];
}
```

## Usage

### Files

```php
// To use this package, first we need an instance of our model
$car = Car::first();

// get full file name with extension
$cat->upload_specification->name(); // car-specification.pdf

// get file basename
$cat->upload_specification->basename(); // car-specification

// get file extension
$cat->upload_specification->extension(); // pdf

// get file size in bytes
$cat->upload_specification->filesize(); // 1500000

// get full url path to file
$car->upload_specification->url(); // documents/specification/car-specification.pdf
```

### Images

```php
// To use this package, first we need an instance of our model
$user = User::first();

// get full image name with extension
$cat->upload_avatar->name(); // user-avatar.jpg

// get full image thumb name with extension
$cat->upload_avatar->name(200, 200); // user-avatar-w200-h200.jpg

// get image basename
$cat->upload_avatar->basename(); // user-avatar

// get image thumb basename
$cat->upload_avatar->basename(200, 200); // user-avatar-w200-h200

// get file extension
$cat->upload_avatar->extension(); // jpg

// get image size in bytes
$cat->upload_avatar->filesize(); // 150000

// get full url path to image
$car->upload_avatar->url(); // images/avatar/user-avatar.jpg

// get full url path to image thumb
$car->upload_avatar->url(200, 200); // images/avatar/thumb/user-avatar-w200-h200.jpg
```

### Upload with UploadedFile

Upload example with usage of Laravel UploadedFile class received by Request instance.

```php
// create new eloquent model object
$car = new Car();

// get UploadedFile from request Illuminate\Http\Request
$file = $request->file('specification');

// create new uploadify instance, set file, model and field name
$uploadify = Uploadify::create($file, $car, 'upload_specification'); // or set($file, new Car, 'upload_specification')

// additional options
$uploadify->setName('custom file name'); // set custom file name
$uploadify->setPath('path-to-custom-directory/'); // set path to custom upload directory, maybe some temporary directory ?

// upload() method returns uploaded file name with extension (without path), so you can save value in database
$specificationName = $uploadify->upload(); // need to define field name;

$car->upload_specification = $specificationName;
$car->save();
```

### Upload with InterventionImage

Upload example with usage of [Intervention Image](http://image.intervention.io/) class created by user. First, you create Image instance with all image manipulations you want (resize, crop, rotate, grayscale...) and then inject that image instance in UploadManager.

```php
// create new eloquent model object
$user = new User;

$file = $request->file('avatar');

// create new uploadify instance, set file, model and field name
$uploadify = Uploadify::create($file, $user, 'upload_avatar'); // or set($image, new User, 'upload_avatar');

// if you want additional image manipulation from Intervention Image package
$image = Image::make($file)->resize(800, null, function ($constraint) {
    $constraint->aspectRatio();
    $constraint->upsize();
});

$uploadify->process($image);

// additional options
$uploadify->setName('custom image name'); // set custom file name
$uploadify->setPath('path-to-custom-directory/'); // set path to custom upload directory, maybe some temporary directory ?

// upload() method returns uploaded file name with extension (without path), so you can save value in database
$avatarName = $uploadify->upload(); // need to define field name;

$user->upload_avatar = $avatarName;
$user->save();
```

### Copy file from existing directory

If you want to copy file from existing directory using `Uploadify`, you must use out-of-the-box solution like creating custom UploadedFile instance.

```php
use Illuminate\Http\UploadedFile;

$name = 'custom-image.jpg';
$path = storage_path('path-to-directory/custom-image.jpg');

$file = new UploadedFile($path, $name, 'image/jpeg', filesize($path), null, true);

// create new eloquent model object
$user = new User;

// create new uploadify instance, set file, model and field name
$uploadify = Uploadify::create($file, $user, 'upload_avatar'); // or set($image, new User, 'upload_avatar');
```

### Delete

delete() method deletes file from filesystem

```php
$car = Car::first();

// deletes file
$car->upload_specification->delete();

// you need to manually set field value to "null" after deletion
$car->upload_specification = null;
```

## Example Usage

### Controller

```php
<?php

namespace App\Http\Controllers

use App\Car;

class CarController
{
    public function index()
    {
        $cars = Car::get();

        $data = [
            'cars' => $cars,
        ];

        return view('index', $data);
    }
}
```

### View
```html
<div class='row'>
    @foreach ($cars as $car)
        <div class='col-12 col-sm-6 col-md-4'>
            @if ($car->upload_cover)
                <p>
                    <img src='{{ $car->upload_cover->url(400, 300) }}'
                         alt='{{ $car->name }}' title='{{ $car->name }}'
                         width='400' height='300'
                         class='img-thumbnail img-fluid'>
                </p>
            @endif
            <h2><a href='{{ $car->url }}'>{{ $car->name }}</a></h2>
            <p>{{ str_limit($car->description, 200) }}</p>
            @if ($car->upload_specification)
                <p>
                    <a href='{{ $car->upload_specification->url() }}'>
                        <i class='fa fa-archive'></i>
                        {{ $car->upload_specification->name() }}
                    </a>
                    <br>
                    <span class='text-muted'>{{ $car->upload_specification->filesize() }} bytes</span>
                </p>
            @endif
        </div>
    @endforeach
</div>
```

## Licence

MIT Licence. Refer to the [LICENSE](https://github.com/fsasvari/laravel-uploadify/blob/master/LICENSE.md) file to get more info.

## Author

Frano Šašvari

Email: sasvari.frano@gmail.com
