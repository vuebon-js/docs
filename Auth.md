# Oturum Yönetimi

- [Başlangıç](#link)
-  [Bileşenler ve Dosya Yolu](#link)

## Başlangıç
Vuebon oturum yönetimi için bir servisi de içerisinde barındırır. Bilindiği üzere SPA uygulamalarda oturum bilgilerinin bir kısmı da istemci bölümünde saklanmalıdır. Tabi ki bilgilerin saklanması kadar güvenliğinin sağlanması da oldukça önemlidir. Bu sebeple Vuebon [JWT](https://jwt.io/introduction) standardını ve Bearer şemasını servisi içinde kullanır. 
Sunduğu güvenliğin yanı sıra tüm bileşenlerden oturum bilgilerine basit bir şekilde erişilebilmesi bir diğer avantajıdır.

## Bileşenler ve Dosya Yolu
Vuebon oturum yönetimi için sunduğu servisin yanı sıra ihtiyaç duyulacak bazı  özelleştirilebilir bileşenleri de beraberinde sunmaktadır. Oturum açma, oturum oluşturma işlemleri için geliştirilen bu başlangıç bileşenleri `src/view/pages/authentication` dosya yolunda bulunmaktadır. 

## Yönlendirmeler
Sunulan tüm bileşenler rota servisinin `auth` isimli makrosu aracılığı ile servis edilirler. 
```javascript
// src/routes.js
Route.auth();
```
Bu makro ile sunulan yönlendirmelerin içeriği şöyledir.

|link| bileşen adı |rota adı | ara katman adı
|--|--|--|-- |
| /login| Login.vue | login | guest |
| /register | Register.vue |register | guest |

Yukarıda ki içerik gösterimi makronun sadece ön tanımlı olarak sunduğu özellikleri yansıtmaktadır. Tabi ki tüm özellikler, ihtiyaca göre dinamik olarak değiştirilebilmektedir.

```javascript
// src/routes.js

Route.auth({
	login: {
		namespace: 'custom-directory',
		middleware: ['custom', 'guest'],
		filename: 'MyLoginComponent',
		name: 'auth.login'
	}
	register: {
		namespace: 'custom-directory',
		name: 'auth.register'
	}
})
```

## Konfigürasyon

Başlangıçta bahsedildiği üzere Oturum Servisi JWT standardı kullanır. Bu sebeple bazı düzenlemelerin yapılmasına ihtiyaç duymaktadır. 

Tüm ayarlamalar için proje ana dizininde bulunan `.env` dosyasını açarak içerisinde bulunan API ve JWT sabitlerini  girilmesi gerekmektedir. Bu bilgileri servis sağlayıcı sisteminden elde edilebilir. Örnek bir uygulama dokümanına [buradan](https://laravel.com/docs/7.x/passport#password-grant-tokens) ulaşılabilir.

```env 
# API Const  
API_BASE_URL=http://my-api-service.com
API_LOGIN_PATH=/token  
API_USER_PATH=/user
# JWT Const  
JWT_GRANT_TYPE=password  
JWT_CLIENT_ID=2  
JWT_CLIENT_SECRET=your_secret_key  
JWT_SCOPE=*
```
## Ara Katmanlar

Servisin sundukları bileşen ve yönlendirmeler ile sınırlı değildir. Sunduklarının yanı sıra yine `auth` ve `guest` adında iki ara katman dosyası sunulmaktadır. `auth` ile oturum açmış kullanıcıyı, `guest` ile konuk oturumun denetlenmesi sağlanabilir.

```javascript
// src/routes.js
Route.link('private-page').middleware('auth');
Route.link('public-page').middleware('guest');
```
## Servis Kullanımı

Servisin sunduğu bir çok taslak özelliğiyle beraber ayrıca bir servis arayüzü de geliştiricilere sunulmaktadır. Oturum kullanıcı bilgileri, oturum kontrolleri, oturum açma ve kapama gibi işlemler bu arayüz ile gerçekleştirilebilir. 

Bu işlemler için `Vue` içerisinde `$auth` prototipi yer almaktadır. Bununla beraber oturum servisi, kapsayıcı içerisinde de yer almaktadır. Kapsayıcı içinden erişim sağlanabilmesi için `auth` anahtarının parametre olarak verilmesi gerekmektedir.

```javascript
import { make } from "vuebon/utils/helpers"

export default {
	mounted() {
		// Prototip(Vue global config) kullanımı
		console.log(this.$auth);
			
		// Kapsayıcı(Container) kullanımı
		console.log(make('auth'));
	}
}
```

### Kullanıcı Bilgisi

Oturum açan kullanıcıya ait bilgiler, sunulan prototip içerisinde `user()` fonksiyonu ile erişilebilir.

```javascript
getUser() {
	return this.$auth.user();
}
```

Alternatif olarak oturum kullanıcısına erişim için kısa bir yola da servis içinde yer verilmiştir.

```javascript
getUser() {
	return this.$user;
}
```
### Oturum Kontrolü
Kullanıcı oturum kontrolü `check()` fonksiyonu ile yapılabilir. Fonksiyon cevap olarak `bool` tipinde bir sonuç dönecektir.

```html
<template>
	<div v-if="$auth.check()">{{$user.full_name}}</div>
	<route-link :to="{name: 'login'}" v-else>Giriş Yap</route-link>
</template>
```

Konuk kullanıcıların kontrolü için `guest()` fonksiyonu tercih edilebilir. Bir önceki `check` fonksiyonun tam tersi gibi düşünülebilir.

```html
<template>
	<div v-if="$auth.guest()">Konuk Oturum</div>
	<div v-else>{{$user.full_name}}</div>
</template>
```
### Oturum Açma ve Kapatma

Giriş işlemleri için `login` fonksiyonu ve fonksiyona verilmek üzere kullanıcı adı ve şifresinden oluşan bir nesne parametresi yeterli olacaktır. Ancak giriş işleminin doğru şekilde gerçekleştirilebilmesi için ilk olarak yukarıda anlatılan konfigürasyon bölümündeki  `JWT` ve `API` ayarlarının yönlendirmelere uygun yapılmış olması gerekir. 

```javascript
export default {
	data() {
		credentials: {
			username: '',
			password: '',
		}
	},
	methods: {
		login() {
			this.$auth.login(this.credentials);
		}
	}
 }
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjcwNjI1MzM1LDU5OTczMDI0OSwzNTk2MT
Q0MzYsLTEwMTYwMzg5MywtODM2MTYxNDQ0LDE0OTkzNzc0MjUs
LTY1MTIwMTM5Nyw4MDQ5MjM4NywxNDAyMDk1ODEwLC0xODMwOD
Y1ODkyLDczMTg0ODg3LC02NTk0NjUxMzEsLTYyMTUyMDY4MCwx
MDAwNjk4MzA1LDE5MzAyMDI1MSwtODc3NDA4OTY1LDE4NDgwMz
UzNzEsODk1MzU5MDU3LC0xMTA3NDkyMjIyLC0xMDI4NTgwNDkw
XX0=
-->