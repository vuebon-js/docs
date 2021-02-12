
# Rota

- [Basit Rota İşlemleri](#basics)
- [Rota Adlandırma](#named)
- [middleware](#middleware) 
- [namespace](#namespace) 
- [group](#group)
	- [middleware](#middleware) 
	- [prefix](#prefix)
	- [namespace](#namespace) 
- [macro](#macro)

<a name="basics"></a>
## Basit Rota İşlemleri
`Vuebon` içerisinde sunulan bir çok özellikten biri de `Route` servisidir. Bir rota oluşturulması için yalnızca `link`  fonksiyonunun kullanılması yeterlidir. Bu fonksiyon kullanılırken ilk parametrede rota yolu, ikinci parametre olarak da bileşen adı tanımlanır. 
```javascript
Route.link('example-path', 'ExampleComponent');
```
#### Tanımlamalar

Tüm rota tanımlamaları  `src/routes.js` içerisinde yapılmaktadır. Oluşturulan rotalarda girilen bileşen adı ile aynı isimde `src/view/pages` dizini altında `.vue` uzantılı dosyanın mevcut olması gerekmektedir.  `Route` servisi burada yapılan tüm tanımlamaları `vue-router` paketine uygun hale getirip, ilgili pakete otomatik olarak entegre eder.  

#### Nasıl Çalışır?
Anlaşılacağı üzere `Route` servisi arka planda `vue-router` paketini kullanır ancak bu servis `vue-router` paketinde ki sıradan obje tanımlama yapısına karşın, sunduğu nesnel arayüzü ile daha basit ve anlaşılır rotalar geliştirilmesine olanak tanır. 

#####  Vuebon kullanımı;

```javascript
Route.group({middleware: 'auth', prefix: 'users', namespace: 'Users'}, function() {
	Route.link('create', 'Create').name('users.create');
	Route.link('{id}/edit', 'Edit').name('users.edit')
})
```
##### Vue-Router çıktısı;

```javascript
[{
    "name": "users.create",
    "component": () => import(`@view/pages${this.options.meta.componentDir}.vue`  /* webpackChunkName: '[request]' */
    "meta": {
        "middleware": ["auth"],
        "namespace": "Users",
        "prefix": "/users",
        "componentDir": "/Users/Create",
        "componentName": "Create"
    },
    "path": "/users/create"
}, {
    "name": "users.edit",
    "component": () => import(`@view/pages${this.options.meta.componentDir}.vue`  /* webpackChunkName: '[request]' */,
    "meta": {
        "middleware": ["auth"],
        "namespace": "Users",
        "prefix": "/users",
        "componentDir": "/Users/Edit",
        "componentName": "Edit"
    },
    "path": "/users/{id}/edit"
}]
```
Yukarıda ki kullanım çıktısından görüldüğü gibi  `vue-router` ile geliştirilmesi karmaşık olabilecek ve dahası zaman alabilecek bir çok özelliği de geliştiricilere sunmaktadır.

<a name="named"></a>
## Rota Adlandırma
Rota tanımlaması yapılırken buna bir ad verilerek daha sonrasında verilen bu ad aracılığı ile de kullanılması mümkündür. Bu tanımla için `name` fonksiyonu kullanılması ve buna `string` tipinde bir parametre eklenmesi gerekmektedir.
```javascript
Route.link('user', 'UsersComponent').name('users');
```
Yukarıda ki örnekte gösterildiği gibi yapılacak tanımlamanın ardından "vue-router" paketinin içindeki sunulan tüm yönlendirme işlemleri içinde kullanılabilir.

####  Link kullanımı;
```html
<template>
	<div id="my-navigator">
		<router-link :to="{name: 'users'}">User List</router-link>
	</div>
</template>
```
#### Programatik kullanımı;
```html
<template>
	<div id="my-navigator">
		<button @click="pushWithName('users')">User List</button>
		<button @click="pushWithName('posts')">Post List</button>
	</div>
</template>
<script>
	export default {
		methods: {
			pushWithName(name) {
				this.$router.push({ name });
			}
		}
	}
</script>
```

## Middleware
Yapılan istek ile geri bildirim arasında bir katman görevi görürler. Bu kısa tanımdan sonrasında rota içerisinde ki kullanım şekillerine geçmeden önce [Middleware](/middleware) başlığı altından nasıl oluşturulacağı ve bunun gibi diğer ayrıntılı bilgilere de göz atılması faydalı olabilir.

Bu kısımda tanımlanan bir `Middleware` özelliğinin `link` fonksiyonu ile en basit şekilde kullanımı gösterilmektedir.

```javascript
Route.link('path', 'MyComponent').middleware('my-middleware');
```
Bunun yanı sıra kullanım şeklinde herhangi bir kısıtlama bulunmamaktadır. Zincirleme yöntemiyle yada `Array` tipinde çoklu `Middleware` kullanımı da mümkün kılınmıştır. 
```javascript
Route.link('path', 'MyComponent').middleware(['auth','admin']);
```
> **Bilgi:** Kullanımda öncelik hassasiyeti mevcuttur. Dolayısıyla önceliklendirme gereken işlemlerde yazım sırasına dikkat edilmelidir.

## Namespace
Basit rota işlemlerinde değinildiği üzere bileşenler `src/view/pages` dizini altında bulunurlar. Ancak zaman zaman bileşenleri bu dizin altında ayrı klasörler altında gruplama ihtiyacı duyulabilir. Bu gibi durumlarda `namespace` fonksiyonu kullanılabilir. Bu fonksiyona girilen `string` parametre sayesinde bileşen ilgili klasörden çağrılabilir.
```javascript
Route.link('login', 'LoginComponent').namespace('Auth');
```
## Gruplama
Birden fazla rotaya benzer özellikler tanımlanmak istenirse rota gruplama işlevi kullanılabilir. Bu işlev `group` fonksiyonu sayesinde kullanılır. İçerisinde tanımlanan her bir rota, grup tarafından paylaşılan özelliklere sahip olur. Böylelikle aynı özellik tekrarından kaçınılırken, tüm rotaların düzenli bir yapıya sahip olması sağlanır.

### Middleware
```javascript
Route.group({middleware: 'auth'}, function() {
	Route.link('posts', 'PostsComponent').name('profile');
	Route.link('newsletter', 'NewsletterComponent').middleware('isAdmin').name('newsletter')
});
```
> **Bilgi:** `group` fonksiyonu içerisinde kullanılan `middleware` özelliği `link` fonksiyonu üzerinden kullanılan `middleware` özelliğine göre her zaman daha önce çalışır.

### Prefix
Aynı link yolu üzerinde bulunan rotalar `prefix` özelliği sayesinde grup içinde sabit olan link yolunu paylaşabilirler. 
 
```javascript
Route.group({prefix: 'users/{id}'}, function() {
	Route.link('profile', 'ProfileComponent');
	Route.link('favourites', 'FavouritesComponent');
});
```

### Namespace
Aynı dizinde yer alan bileşenler bu özellik sayesinde gruplanabilir. Diğer `link` kullanımında olduğu gibi başlangıç değeri `src/view/pages` dizini olacaktır. İlgili özellik ile yazılacak her yol ana dizin yoluyla birleşecek ve ardına eklenecektir. 
```javascript
Route.group({namespace: 'User'}, function() {
	Route.link('profile', 'ProfileComponent');
	Route.link('favourites', 'FavouritesComponent');
});
```
Yine dikkat edilmesi gereken bir diğer şey, tüm grup özelliklerinde olduğu gibi öncelikli olarak grup içerisinde yazılan işlevlerin çalışacağıdır.
```javascript
Route.group({namespace: 'DirA'}, function() {
	Route.link('foo', 'Foo').namespace('DirB');
	Route.link('bar', 'Bar');
});
```
Bu gibi bir örnekte `Foo` bileşeninin dosya yolu `src/view/pages/DirA/DirB/Foo.vue` şeklinde olacaktır.

## Makro
Rota servisinin sunduğu arayüze ek özellikler eklenebilir. Bu  `macro` fonksiyonu kullanılarak yapılabilir. 
```javascript
Route.macro('named', function(Route, ...params) {
	console.log(params);
})

Route.named('foo', 'bar');
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTkyNjg4MDEzMiwxNzQxOTM0MzM1LC01OT
M3MzM3MDgsMjExNTIyMzAwMSwtMTU0OTkyNzA4MiwxODg2NTMx
MDM0LC04NDkwNDM0MiwtNzkyMzMwMDIzLDMzMzE4MTM5MSwxNj
I1MjYyOTYwLDEzMTEwMTQyMTAsODgzNjc4MzgzLC0xNTEzMDgy
MDkzLDExMDk1OTI5MDQsMTAzNDAyMjY4NiwtMjE0NzQwMzk5My
wtNzMxNDg0ODI2LC0yMDI3NTExNDE1LDY4MjM5NDczMCwtMTgz
MTIzNDE1Ml19
-->