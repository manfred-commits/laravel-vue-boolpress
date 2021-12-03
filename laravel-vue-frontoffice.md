# Laravel-vue-frontoffice


### Step 1 Inizio
1. inizializiamo il progetto 
```composer create-project --prefer-dist laravel/laravel:^7.0 .```

2. creiamo il db e colleghiamolo nel file .env 

```php
// APP_NAME=Laravel
// APP_ENV=local
// APP_KEY=base64:ReKoSVdVWhYNHas9cyi57YiYAkAb7ak2xMy87pecv70=
// APP_DEBUG=true
// APP_URL=http://localhost

// LOG_CHANNEL=stack

// DB_CONNECTION=mysql
// DB_HOST=127.0.0.1
// DB_PORT=3306
// DB_DATABASE=laravel
// DB_USERNAME=root
// DB_PASSWORD=
```

3. lanciare il comando ```php artisan config:clear``` dopo aver collegato il db
4. lanciare il comando ```php artisan serve``` per vedere se funziona tutto

### Step 2 Authentication

1. installiamo la libreria ```composer require laravel/ui:^2.4```
2. installiamo lo scaffolding fornito da laravel (in laravel/ui),
 utilizzando il comando ```php artisan ui vue --auth```
3. adesso lanciando il comando ```php artisan migrate``` e avremmo l'autenticazione aggiunta alla nostra pagina


## Step 3 Separazione Backoffice e Frontoffice
Laravel crea già un **HomeController**, possiamo eliminarlo o spostarlo in maniera appropriata.

- Se vogliamo elimarlo e crearlo nuovamente possiamo:

creare un **HomeController** sotto il namespace Admin che gestirà la pagina di atterraggio dopo il login 
```php artisan make:controller Admin/HomeController ```

- se vogliamo invece spostartlo mantenendolo:
possiamo creare una cartella apposita nella cartella ```controller``` denominata ```Admin``` in cui andremmo a posizionare il nostro file **HomeController** 

**Nota bene: il path del file Home Controller, dovra essere cambiato in maniera opportuna** 

**```use App\Http\Controllers\Controller```** 

1. construito l'home controller, creiamo un **PageController** per gestire l'homepage e le pagine pubbliche


```php artisan make:controller PageController ```

## Definizione rotte
definiamo ora le rotte per la parte di backoffice, raggruppandole con namespace, prefisso, middleware auth e name.
```php
// routes/web.php

// Rotte pubbliche
Route::get('/', 'PageController@index');

// Rotte Autenticazione
Auth::routes();

// Rotte area Admin
// con questo tipo di rotta, si sta impostando un prefisso admin
Route::middleware('auth')->namespace('Admin')->name('admin.')->prefix('admin')->group(function() {
    Route::get('/', 'HomeController@index')->name('home');
});

// Se si vuole dunque modificare redirect di base della pagina bisogna andare in RouteServiceProvider,
// (App/Providers/RouteServiceProvider.php) 
```

### Modifica path della costante HOME
1. Cambiamo l'url di redirect dopo il login (al posto di `/home`, scriviamo `/admin`)

```php
// nel file app/Providers/RouteServiceProvider.php

class RouteServiceProvider extends ServiceProvider
{
    // ...

		/**
     * The path to the "home" route for your application.
     *
     * @var string
     */
    // cambiamo questa costante da "/home" a "/admin"
    public const HOME = '/admin';

		// ...
}
```
### Organizzazione views

1. Creiamo **2 layout**: uno per la parte di backoffice e uno per la parte di frontoffice:

- Creiamo una sottocartella `resources/views/admin/` per gestire tutte le views lato **backoffice**.

In questa cartella potremmmo spostare il nostro file `home.blade.php`. Una volta fatto ciò sarà necessario andare a modificare la route restituita nel nostro `HomeController`

```php
public function index()
    {
        return view('admin.home');
    }
```

- e creiamo una sottocartella `resources/views/guest/` per gestire tutte le views lato **frontoffice**

In questa cartella potremmo inizializzare un file `front.blade.php`, in cui andremmo a svolgere le nostre operazioni di front office tramite vue.



## Divisione cartelle scss per backoffice e frontoffice 

1. All'interno della nostra cartella sass/js possiamo dedicare dei file, che andranno ad essere unicamente utilizzati per il front office e per il backoffice. 

`resources/sass/front.scss`

`resources/js/front.js`
per questo file sarà necessario importare il seguente codice per inizializzarlo in maniera appropriata

```js

// questo codice ci consente di utilizzare axios nel nostro file
window.axios = require('axios');
window.axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';

// questo sezione del codice e necessaria per utilizzare vue
window.Vue = require('vue');

const app = new Vue({
    el: '#app',
});
```

Una volta fatto ciò si dovrà andare nel nostro file `webpack.mix.js`
e aggiungere i relativi path dei nuovi file creati, nella seguente maniera

```js
const mix = require('laravel-mix');

/*
 |--------------------------------------------------------------------------
 | Mix Asset Management
 |--------------------------------------------------------------------------
 |
 | Mix provides a clean, fluent API for defining some Webpack build steps
 | for your Laravel application. By default, we are compiling the Sass
 | file for the application as well as bundling up all the JS files.
 |
 */


mix.js('resources/js/app.js', 'public/js')
    .js('resources/js/front.js', 'public/js')
    .sass('resources/sass/app.scss', 'public/css')
    .sass('resources/sass/front.scss', 'public/css');
    

```

Una volta completato questo passaggio, lanciare il comando `npm run watch`, per far compilare i file css e js

## Inizializiamo le nostre view per frontoffice e backoffice

Avendo suddiviso le nostre view in `resources/views/admin/home.blade.php` e `resources/views/guest/front.blade.php`, dovremmo inizializzare i nostri file.

- Per `resources/views/admin/home.blade.php` laravel fornirà già un layout html con gli opportuni collegamenti al file css

- Per `resources/views/guest/front.blade.php` dovremmo andare a compilare il file aggiungendo un normale layout html,

ma prestando cura di apporre gli opportuni collegamenti al file css

`<link rel="stylesheet" href="{{asset("css/front.css")}}">`

ed al file js, alla fine del body

`<script src="{{asset("js/front.js")}}"></script>`

Inoltre, sarà necessario aggiungere, per creare un collegamento con il nostro file vue

```html

<div id="app"></div>
```


## Crud time

Arrivati a questo punto possiamo dedicarci alla creazione delle nostre **CRUD** per ogni singola entità, ricordando di separare parte pubblica da parte privata!