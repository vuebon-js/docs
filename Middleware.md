# Ara Katman

- [Ara Katman Nedir?](#what-is-middleware)
- [Tanımlama](#defining-middleware)
	- [before](#before)
	- [after](#after)
- [Nasıl Kullanılır](#how-to-use-middleware)
	- [Global Kullanım](#global-registration)
	- [Rota Kullanımı](#route-registration)

## Ara Katman Nedir? 

Genel tanımıyla yapılacak işlemlerin öncesinde veya sonrasında yapılmak istenen, filtrasyon işlemlerinin yürütüldüğü, terminolojik olarak `middleware` adıyla sıkça karşılaşılan katmandır. 

Vuebon ara katmanları sayfa yönlendirmeleri işlemlerinde benzeri filtre işlemleri için kullanılır. Oturum kontrol sistemi buna iyi bir örnek olarak gösterilebilir. Vuebon hem çatının merkezi sisteminde bu yapıyı kullanırken hem de geliştiricilerine kendi işlerine göre özelleştirilmiş ara katmanlar oluşturabilmeleri için sahip olduğu alt yapıyı sunar. 


## Tanımlama

Ara katman dosyaları `src/middleware` dizininde bulunur.  Ara katmanlar yapılan tanımlama ve kayıt işlemlerine göre sırasıyla çalışırlar. Dolayısıyla hiç bir ara katman işlemi tamamlanmadan bir diğerine geçilemez.

Bir ara katman dosyasında `before` ve `after` olmak üzere iki temel alan mevcuttur. Yapılacak işlem türüne göre `before` yada `after` fonksiyonundan en az biri tercih edilmelidir.

###  Before

İlgili fonksiyon sayfa yönlendirmesi yapılmadan önce çalışan alandır. Sayfa yönlendirmesi olmadan yapılmak istenilen tüm işlemler bu fonksiyon içerisinde gerçekleştirilmelidir. İlgili fonksiyon senkron yada asenkron şekilde kullanılabilir. Örnek vermek gerekirse `vue-router` kütüphanesindeki [beforeEach](https://router.vuejs.org/guide/advanced/navigation-guards.html#global-before-guards) fonksiyonu ile benzer özellikler gösterir. 

Bu fonksiyon içerisinde yapılan işlemlerde `return` edilme zorunluluğu yoktur. Dönen değerin `undefined` yada `true` olması yönlendirme işlemine devam edilebilir anlamına gelmektedir.

Ancak yönlendirmenin durdurulması istenildiği durumlarda `false` değeri döndürülmesi yeterli olacaktır. Bunun yanında yönlendirme işleminin durdurulup başka bir sayfaya yönlendirilmesi de mümkündür. Yönlendirme işlemi iki şekilde gerçekleştirilebilir. Bir sayfa linkine ait `object` tipinde yada bir  `string` tipinde belirtilebilir.

> Rota  yönlendirmeleri için [vue-router:programatic-navigation](https://router.vuejs.org/guide/essentials/navigation.html) sayfasının incelenmesi faydalı olabilir.

```javascript
export default class ExampleMiddleware() {
	before(route) {
		if (route.from.name === "foo") {
			return {name: "bar"};
		} else if (route.from.name === "baz") {
			return '/foo';
		}
	}
}
```

### After

Sayfa yönlendirmesi tamamlandıktan sonra çalışan kısımdır. Örnek vermek gerekirse `vue-router` kütüphanesindeki [afterEach](https://router.vuejs.org/guide/advanced/navigation-guards.html#global-after-hooks) fonksiyonu ile benzer bir yapıya sahiptir. 

```javascript
export default class ExampleMiddleware() {
	after({ to }) {
	    localstorage.setItem('last_seen_page', to.name);
	}
}
```

## Nasıl Kayıt Edilir?

Ara katmanlar kullanılabilmesi için kayıt edilmesi gerekmektedir. Global ve rota kullanımı olmak üzere iki şekilde kayıt edilebilir. Yapısı gereği ara katmanlar sırasıyla yani ardışık olarak çalışırlar. Aşağıdaki kullanım şekillerinde belirtilen öncelik yönlendirmelerine bu sebeple dikkat edilmelidir.

### Global Kayıt

Global kullanım için ara katman örnekte görüldüğü şekilde `globals` alanına kayıt edilmesi yeterlidir. Bu şekilde her sayfa yönlendirmesinde ara katman yazıldığı sıraya göre sırasıyla çalışacaktır.  
```javascript
import {FirstMiddleware}  from "@middleware/FirstMiddleware";  
import {SecondMiddleware} from "@middleware/SecondMiddleware";  

export const globals = [  
	FirstMiddleware,
	SecondMiddleware
];
```
### Rota Kayıt

Rota kullanımı için ara katmanlar `routes` alanına kayıt edilir. Bu bakımdan global kullanım ile benzer bir kayıt mantığında ilerler. Ancak global kullanımdan farklı olarak adından anlaşılacağı gibi her sayfa yönlendirmesinde değil belirtilen rota tanımı için yalnızca çalışır. Kayıt esnasında yazım önceliği mevcut olmamasına karşın kullanım anında çalışma önceliği mevcuttur. 

```javascript
import {FooMiddleware} from "@middleware/Foo";  

export const routes = {  
	'foo': FooMiddleware
};
```
Kayıt işleminin ardından oluşturulan ara katmanın rotada kullanım şekli aşağıdaki gibidir. 

```javascript
Route.link('/bar', 'Bar').middleware('bar')
```

> Ayrıca ara katmanların rota içerisindeki detaylı kullanımı  [Rota Ara Katmanları](Route.md#middleware) sayfasında anlatılmaktadır.

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU3MDkzMTk2Niw1MjEwNjM0NDUsNjY2MT
QxNTE3LDE0NzE1Mzk1ODEsLTE5MTM4MDMwMTEsLTE1NTI5MDkx
NiwtNTc1MDY3MjY0LDEyOTczMDU3MSwxNjYzMzk4NzUzLDEyMj
MzNTM4NjQsOTU4MjM4ODgzLC02ODU0MzMyNCwyMDU2MjgxODAw
LDM0MjEwNzcsLTc4MTQxMTY1NCwxNDMzNzAzODQwLC0xNTE3Mj
A2NTAzLC0yMDUwMTI0MjkwLDMxNzc4Nzg1MSw2MDQwMTA0MTZd
fQ==
-->