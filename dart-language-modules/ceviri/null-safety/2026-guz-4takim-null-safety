# Sağlam null güvenliği

Dart'ta null güvenliğine giriş.

## Genel bakış

Dart dili sağlam null güvenliğini zorunlu kılar; bu da bir `null` değer üzerindeki bir üyeye istemeden erişmeyi imkânsız hale getirir.

Dart'ta türler varsayılan olarak null olamaz. Null olamayan türdeki değişkenlerin başlatılması gerekir ve bunlara yalnızca null olmayan değerler atanabilir.

Dart analizcisi ve derleyicileri, potansiyel olarak `null` olabilecek değerlerin güvensiz kullanımını düzenleme zamanında yakalar; böylece diğer dillerde çalışma zamanı hatası olacak durumlar, yayına almadan önce düzeltebileceğiniz analiz hatalarına dönüşür.

## Örnekler

Aşağıdaki koddaki değişkenlerin hiçbiri `null` olamaz:

```dart
// Bunların hiçbiri asla null olamaz.
var i = 42; // int olarak çıkarsanır.
String name = getFileName();
final b = Foo();
```

Bir değişkenin `null` değerine sahip olabileceğini belirtmek için, tür bildirimine `?` ekleyin:

```dart
int? aNullableInt = null;
```

**Kaynaklar:**

- Etkileşimli örnekler için [Dart cheatsheet](https://dart.dev/resources/dart-cheatsheet) sayfasını deneyin.

- Null güvenliği hakkında daha fazla bilgi edinmek için [Null güvenliğini anlamak](https://dart.dev/null-safety/understanding-null-safety) sayfasına göz atın.

## Null güvenliği ilkeleri

Dart'ta null güvenliği iki temel tasarım ilkesi üzerine kuruludur:

**Varsayılan olarak null olamaz**

Dart'a bir değişkenin null olabileceğini açıkça söylemediğiniz sürece, o değişken null olamaz kabul edilir. Bu varsayılan, araştırmalarda API'lerde null olmamanın açık farkla en yaygın tercih olduğu görüldükten sonra seçilmiştir.

**Tamamen sağlam**

Tür sistemi bir değişkenin veya ifadenin null olamayan bir türe sahip olduğunu belirlerse, çalışma zamanında asla `null` olarak değerlendirilmeyeceği garanti edilir.

Bu ilkeler birlikte daha az hata, daha küçük ikili dosyalar ve daha hızlı çalışma sağlar.

## Geçmiş geçiş kaynakları

Dart, Mayıs 2023'te yayımlanan Dart 3'ten beri sağlam null güvenliğini zorunlu kılmaktadır. Uygulamalarınızı veya paketlerinizi hâlâ null güvenliğine geçirmeniz gerekiyorsa, [dart-community/migrate-to-null-safety](https://github.com/dart-community/migrate-to-null-safety#migrate-to-dart-null-safety) deposundaki arşivlenmiş belgelere göz atın.

# Null safety'yi anlamak

Dart dilindeki ve kütüphanelerindeki null safety ile ilgili değişikliklere derinlemesine bir bakış.

Null safety, Dart 2.0'da orijinal sağlam olmayan (unsound) opsiyonel tip sistemini sağlam (sound) bir statik tip sistemiyle değiştirdiğimizden beri Dart'ta yaptığımız en büyük değişiklik. Dart ilk çıktığında, derleme zamanında null safety nadir görülen ve uzun bir girişle anlatılması gereken bir özellikti. Bugün Kotlin, Swift, Rust ve diğer diller, artık çok tanıdık hale gelen bu soruna kendi cevaplarını sunuyor. İşte bir örnek:

```dart
// Without null safety:
bool isEmpty(String string) => string.length == 0;
 
void main() {
  isEmpty(null);
}
```

Bu Dart programını null safety olmadan çalıştırırsan, `.length` çağrısında `NoSuchMethodError` istisnası fırlatır. `null` değeri `Null` sınıfının bir örneğidir ve `Null` sınıfında "length" adlı bir getter yoktur. Çalışma zamanı hataları berbat bir şey. Bu, Dart gibi son kullanıcının cihazında çalışmak üzere tasarlanmış bir dilde özellikle geçerli. Bir sunucu uygulaması çökerse, çoğu zaman kimse fark etmeden yeniden başlatabilirsin. Ama bir Flutter uygulaması kullanıcının telefonunda çöktüğünde, kullanıcı memnun olmaz. Kullanıcıların memnun değilse, sen de memnun olmazsın.

Geliştiriciler Dart gibi statik tipli dilleri sever, çünkü tip denetleyicisi koddaki hataları derleme zamanında, çoğu zaman doğrudan IDE içinde bulabilir. Bir hatayı ne kadar erken bulursan, o kadar erken düzeltirsin. Dil tasarımcıları "null referans hatalarını düzeltmekten" bahsettiğinde, statik tip denetleyicisini zenginleştirmeyi kastederler; böylece dil, `null` olabilecek bir değer üzerinde `.length` çağırmaya çalışmak gibi hataları yakalayabilir.

Bu soruna tek bir doğru çözüm yok. Rust ve Kotlin'in her birinin kendi dilleri bağlamında anlamlı olan kendi yaklaşımı var. Bu doküman, Dart için bulduğumuz cevabın tüm ayrıntılarını anlatıyor. Statik tip sistemindeki değişiklikleri ve null-safe kod yazmanı, hatta umarım bundan keyif almanı sağlayacak bir dizi başka değişikliği ve yeni dil özelliğini içeriyor.

Bu doküman uzun. Çalışmaya başlamak için bilmen gerekenleri kapsayan daha kısa bir şey istiyorsan, genel bakış (overview) ile başla. Daha derin bir anlayışa hazır olduğunda ve vaktin olduğunda buraya dön; böylece dilin `null` ile nasıl başa çıktığını, neden bu şekilde tasarladığımızı ve idiomatik, modern, null-safe Dart'ın nasıl yazılacağını anlayabilirsin. (Spoiler: sonuç, bugün Dart yazma şeklinle şaşırtıcı derecede benzer.)

Bir dilin null referans hatalarını ele almasının çeşitli yollarının her birinin artıları ve eksileri var. Aldığımız kararlara şu ilkeler yön verdi:

Kod varsayılan olarak güvenli olmalı. Yeni Dart kodu yazarsan ve açıkça güvensiz hiçbir özellik kullanmazsan, çalışma zamanında asla null referans hatası fırlatmaz. Olası tüm null referans hataları statik olarak yakalanır. Daha fazla esneklik için bu denetimin bir kısmını çalışma zamanına bırakmak istersen bunu yapabilirsin, ama bunu kodda metin olarak görünen bir özellik kullanarak bilinçli şekilde seçmen gerekir.

Başka bir deyişle, sana bir can yeleği verip suya her çıktığında onu giymeyi hatırlamayı sana bırakmıyoruz. Bunun yerine sana batmayan bir tekne veriyoruz. Tekneden kendin atlamadıkça kuru kalırsın.

Null-safe kod yazması kolay olmalı. Mevcut Dart kodunun çoğu dinamik olarak doğrudur ve null referans hatası fırlatmaz. Dart programının şu anki görünümünü seviyorsun ve biz de kodu bu şekilde yazmaya devam edebilmeni istiyoruz. Güvenlik; kullanım kolaylığından fedakârlık etmeyi, tip denetleyicisine kefaret ödemeyi ya da düşünme şeklini ciddi biçimde değiştirmeyi gerektirmemeli.

Ortaya çıkan null-safe kod tamamen sağlam (sound) olmalı. Statik denetim bağlamında "soundness" farklı insanlar için farklı anlamlara gelir. Bizim için, null safety bağlamında bunun anlamı şu: bir ifadenin `null` değerine izin vermeyen bir statik tipi varsa, o ifadenin hiçbir olası çalışması asla `null` olarak değerlendirilemez. Dil bu garantiyi çoğunlukla statik denetimlerle sağlar, ama işin içinde bazı çalışma zamanı denetimleri de olabilir. (Ancak ilk ilkeyi unutma: bu çalışma zamanı denetimlerinin yapıldığı her yer senin seçimin olacak.)

Soundness kullanıcı güveni için önemlidir. Çoğunlukla yüzeyde kalan bir tekneyle açık denizlere çıkmak seni heyecanlandırmaz. Ama cesur derleyici hacker'larımız için de önemlidir. Dil bir programın anlamsal özellikleri hakkında kesin garantiler verdiğinde, derleyici bu özelliklerin doğru olduğunu varsayan optimizasyonlar yapabilir. `null` söz konusu olduğunda bu, gereksiz `null` denetimlerini ortadan kaldıran daha küçük kod ve üzerinde metot çağırmadan önce alıcının `null` olmadığını doğrulamasına gerek olmayan daha hızlı kod üretebileceğimiz anlamına gelir.

Bir uyarı: Soundness garantisini yalnızca tamamen null safe olan Dart programlarında veriyoruz. Dart, daha yeni null-safe kod ile eski (legacy) kodun karışımını içeren programları destekler. Bu karma sürümlü programlarda null referans hataları yine de oluşabilir. Karma sürümlü bir programda null safe olan kısımlarda statik güvenliğin tüm faydalarını elde edersin, ama uygulamanın tamamı null safe olana kadar tam çalışma zamanı soundness'ına sahip olmazsın.

`null`'ı ortadan kaldırmanın bir hedef olmadığını unutma. `null`'da yanlış bir şey yok. Aksine, bir değerin yokluğunu temsil edebilmek gerçekten çok faydalı. Dilin içine özel bir "yok" değeri desteğini doğrudan yerleştirmek, yokluğla çalışmayı esnek ve kullanışlı kılar. Bu; opsiyonel parametrelerin, kullanışlı `?.` null-aware operatörünün ve varsayılan başlatmanın temelini oluşturur. Kötü olan `null` değil, `null`'ın beklemediğin yerlere gitmesidir.

Dolayısıyla null safety ile hedefimiz, sana `null`'ın programında nereden akabileceği konusunda kontrol ve içgörü vermek ve bir çökmeye yol açacak bir yere akamayacağından emin olmanı sağlamak.

## Tip sisteminde nullability

Null safety statik tip sisteminde başlar, çünkü geri kalan her şey buna dayanır. Dart programında koca bir tip evreni var: `int` ve `String` gibi ilkel tipler, `List` gibi koleksiyon tipleri ve senin ve kullandığın paketlerin tanımladığı tüm sınıflar ve tipler. Null safety'den önce, statik tip sistemi `null` değerinin bu tiplerin herhangi birindeki ifadelere akmasına izin veriyordu.

Tip teorisi jargonuyla, `Null` tipi tüm tiplerin bir alt tipi (subtype) olarak ele alınıyordu:

![Null Safety Hierarchy Before](2026-guz-4takim-gorseller/image10.png)

Bir ifade üzerinde izin verilen işlemler kümesi (getter'lar, setter'lar, metotlar ve operatörler) o ifadenin tipi tarafından tanımlanır. Tip `List` ise üzerinde `.add()` veya `[]` çağırabilirsin. `int` ise `+` kullanabilirsin. Ama `null` değeri bu metotların hiçbirini tanımlamaz. `null`'ın başka bir tipteki ifadeye akmasına izin vermek, bu işlemlerin herhangi birinin başarısız olabileceği anlamına gelir. Null referans hatalarının özü gerçekten de budur: her hata, `null` üzerinde onda olmayan bir metodu veya özelliği aramaya çalışmaktan kaynaklanır.

### Non-nullable ve nullable tipler

Null safety, tip hiyerarşisini değiştirerek bu sorunu kökünden ortadan kaldırır. `Null` tipi hâlâ var, ama artık tüm tiplerin alt tipi değil. Bunun yerine tip hiyerarşisi şöyle görünüyor:

![Null Safety Hierarchy After](2026-guz-4takim-gorseller/image11.png)

`Null` artık bir alt tip olmadığından, özel `Null` sınıfı dışında hiçbir tip `null` değerine izin vermez. Tüm tipleri varsayılan olarak non-nullable yaptık. `String` tipinde bir değişkenin varsa, her zaman bir string içerir. İşte, tüm null referans hatalarını düzelttik.

`null`'ın hiç faydalı olmadığını düşünseydik burada durabilirdik. Ama `null` faydalı, bu yüzden onu ele almanın yine de bir yoluna ihtiyacımız var. Opsiyonel parametreler iyi bir açıklayıcı örnek. Şu null-safe Dart koduna bak:

```dart
// Using null safety:
void makeCoffee(String coffee, [String? dairy]) {
  if (dairy != null) {
    print('$coffee with $dairy');
  } else {
    print('Black $coffee');
  }
}
```

Burada `dairy` parametresinin herhangi bir string'i ya da `null` değerini, başka hiçbir şeyi kabul etmesini istiyoruz. Bunu ifade etmek için, temel tip `String`'in sonuna `?` ekleyerek `dairy`'ye nullable bir tip veriyoruz. Arka planda bu, esasen temel tip ile `Null` tipinin bir birleşimini (union) tanımlamak demek. Yani Dart'ta tam özellikli union tipler olsaydı, `String?` aslında `String|Null` için bir kısaltma olurdu.

### Nullable tipleri kullanmak

Nullable tipte bir ifaden varsa, sonucuyla ne yapabilirsin? İlkemiz varsayılan olarak güvenli olmak olduğundan, cevap pek bir şey. Temel tipin metotlarını çağırmana izin veremeyiz, çünkü değer `null` ise bunlar başarısız olabilir:

```dart
// Hypothetical unsound null safety:
void bad(String? maybeString) {
  print(maybeString.length);
}
 
void main() {
  bad(null);
}
```

Bunu çalıştırmana izin verseydik çökerdi. Güvenle erişmene izin verebileceğimiz tek metotlar ve özellikler, hem temel tip hem de `Null` sınıfı tarafından tanımlananlar. Bunlar sadece `toString()`, `==` ve `hashCode`. Yani nullable tipleri map anahtarı olarak kullanabilir, set'lerde saklayabilir, başka değerlerle karşılaştırabilir ve string interpolasyonunda kullanabilirsin, ama o kadar.

Peki non-nullable tiplerle nasıl etkileşirler? Nullable tip bekleyen bir şeye non-nullable tip geçirmek her zaman güvenlidir. Bir fonksiyon `String?` kabul ediyorsa ona `String` geçirmek serbesttir, çünkü hiçbir soruna yol açmaz. Bunu, her nullable tipi temel tipinin bir üst tipi (supertype) yaparak modelliyoruz. Nullable tip bekleyen bir şeye güvenle `null` da geçirebilirsin, dolayısıyla `Null` da her nullable tipin bir alt tipidir:

![Nullable](2026-guz-4takim-gorseller/image12.png)

Ama diğer yöne gidip, temel non-nullable tipi bekleyen bir şeye nullable bir tip geçirmek güvensizdir. `String` bekleyen kod, değer üzerinde `String` metotlarını çağırabilir. Ona bir `String?` geçirirsen, `null` içeri akabilir ve bu başarısız olabilir:

```dart
// Hypothetical unsound null safety:
void requireStringNotNull(String definitelyString) {
  print(definitelyString.length);
}
 
void main() {
  String? maybeString = null; // Or not!
  requireStringNotNull(maybeString);
}
```

Bu program güvenli değil ve buna izin vermemeliyiz. Ancak Dart'ta her zaman *implicit downcast* (örtük aşağı tip dönüşümü) denen bir şey oldu. Örneğin `String` bekleyen bir fonksiyona `Object` tipinde bir değer geçirirsen, tip denetleyicisi buna izin verir:

```dart
// Without null safety:
void requireStringNotObject(String definitelyString) {
  print(definitelyString.length);
}
 
void main() {
  Object maybeString = 'it is';
  requireStringNotObject(maybeString);
}
```

Soundness'ı korumak için derleyici, `requireStringNotObject()` fonksiyonuna verilen argümana sessizce bir `as String` cast'i ekler. Bu cast çalışma zamanında başarısız olup istisna fırlatabilir, ama derleme zamanında Dart bunun sorun olmadığını söyler. Non-nullable tipler nullable tiplerin alt tipleri olarak modellendiğinden, implicit downcast'ler `String` bekleyen bir şeye `String?` geçirmene izin verirdi. Buna izin vermek, varsayılan olarak güvenli olma hedefimizi ihlal ederdi. Bu yüzden null safety ile implicit downcast'leri tamamen kaldırıyoruz.

Bu, `requireStringNotNull()` çağrısının derleme hatası vermesini sağlıyor, ki istediğin de bu. Ama aynı zamanda tüm implicit downcast'lerin derleme hatası olacağı anlamına geliyor; `requireStringNotObject()` çağrısı da dahil. Açık (explicit) downcast'i kendin eklemen gerekecek:

```dart
// Using null safety:
void requireStringNotObject(String definitelyString) {
  print(definitelyString.length);
}
 
void main() {
  Object maybeString = 'it is';
  requireStringNotObject(maybeString as String);
}
```

Bunun genel olarak iyi bir değişiklik olduğunu düşünüyoruz. İzlenimimiz, çoğu kullanıcının implicit downcast'leri hiç sevmediği yönünde. Özellikle daha önce şuna yanmış olabilirsin:

```dart
// Without null safety:
List<int> filterEvens(List<int> ints) {
  return ints.where((n) => n.isEven);
}
```

Hatayı gördün mü? `.where()` metodu tembeldir (lazy), bu yüzden `List` değil `Iterable` döndürür. Bu program derlenir, ama sonra `Iterable`'ı `filterEvens()` fonksiyonunun döndüreceğini bildirdiği `List` tipine cast etmeye çalışırken çalışma zamanında istisna fırlatır. Implicit downcast'lerin kaldırılmasıyla bu bir derleme hatası olur.

Nerede kalmıştık? Evet, tamam, yani programındaki tip evrenini alıp iki yarıya böldük gibi:

![Nullable and Non-Nullable types](2026-guz-4takim-gorseller/image13.png)

Non-nullable tiplerden oluşan bir bölge var. Bu tipler tüm ilginç metotlara erişmene izin verir, ama asla `null` içeremez. Sonra bunların karşılığı olan tüm nullable tiplerden oluşan paralel bir aile var. Bunlar `null`'a izin verir, ama onlarla pek bir şey yapamazsın. Değerlerin non-nullable taraftan nullable tarafa akmasına izin veriyoruz çünkü bunu yapmak güvenli, ama diğer yöne değil.

Bu, nullable tiplerin temelde işe yaramaz olduğu izlenimini veriyor. Hiçbir metotları yok ve onlardan kurtulamıyorsun. Merak etme, değerleri nullable yarıdan diğer tarafa taşımana yardım edecek bir dizi özelliğimiz var; yakında onlara geleceğiz.

### Üst ve alt tipler (top ve bottom)

Bu bölüm biraz egzotik. Tip sistemi işlerine meraklı değilsen, en sondaki iki madde dışında büyük ölçüde atlayabilirsin. Programındaki tüm tipleri, birbirinin alt tipi ve üst tipi olanlar arasında kenarlar olacak şekilde hayal et. Bunu, bu dokümandaki diyagramlar gibi çizersen, `Object` gibi üst tiplerin tepeye, kendi tiplerin gibi yaprak sınıfların dibe yakın olduğu kocaman bir yönlü graf oluşur.

Bu yönlü graf tepede, (doğrudan ya da dolaylı olarak) tüm tiplerin üst tipi olan tek bir tipte birleşiyorsa, bu tipe *top type* (üst tip) denir. Aynı şekilde, dipte her tipin alt tipi olan tuhaf bir tip varsa, bir *bottom type*'ın (alt tip) vardır. (Bu durumda yönlü grafın bir lattice'tir.)

Tip sisteminin bir top ve bottom tipinin olması kullanışlıdır, çünkü *least upper bound* (en küçük üst sınır) gibi tip düzeyindeki işlemlerin (tip çıkarımı bunu, iki dalının tiplerine göre koşullu bir ifadenin tipini bulmak için kullanır) her zaman bir tip üretebilmesi anlamına gelir. Null safety'den önce `Object` Dart'ın top tipi, `Null` ise bottom tipiydi.

`Object` artık non-nullable olduğundan top tip değil. `Null` onun alt tipi değil. Dart'ta adı konmuş bir top tip yok. Bir top tipe ihtiyacın varsa, istediğin `Object?`. Aynı şekilde `Null` da artık bottom tip değil. Öyle olsaydı her şey hâlâ nullable olurdu. Bunun yerine `Never` adında yeni bir bottom tip ekledik:

![Top and Bottom](2026-guz-4takim-gorseller/image14.png)

Pratikte bu şu anlama geliyor:

- Herhangi bir tipte bir değere izin verdiğini belirtmek istiyorsan `Object` yerine `Object?` kullan. Aslında `Object` kullanmak oldukça nadir hale gelir, çünkü bu tip "bu tuhaf şekilde yasaklanmış `null` değeri dışında herhangi bir olası değer olabilir" anlamına gelir.

- Bir bottom tipe ihtiyaç duyduğun nadir durumlarda `Null` yerine `Never` kullan. Bu, özellikle bir fonksiyonun asla geri dönmediğini belirtip erişilebilirlik (reachability) analizine yardımcı olmak için kullanışlıdır. Bir bottom tipe ihtiyacın olup olmadığını bilmiyorsan, muhtemelen ihtiyacın yoktur.

## Doğruluğu sağlamak

Tip evrenini nullable ve non-nullable yarılara böldük. Soundness'ı ve çalışma zamanında sen istemedikçe asla null referans hatası alamayacağın ilkemizi korumak için, `null`'ın non-nullable taraftaki hiçbir tipte asla görünmemesini garanti etmemiz gerekiyor.

Implicit downcast'lerden kurtulmak ve `Null`'ı bottom tip olmaktan çıkarmak, tiplerin bir program içinde atamalar boyunca ve argümanlardan fonksiyon çağrılarındaki parametrelere akışının ana yerlerinin hepsini kapsıyor. `null`'ın içeri sızabileceği geriye kalan ana yerler, bir değişkenin ilk var olduğu an ve bir fonksiyondan çıktığın an. Bu yüzden bazı ek derleme hataları var:

### Geçersiz return'ler

Bir fonksiyonun non-nullable bir dönüş tipi varsa, fonksiyon içindeki her yol, bir değer döndüren bir `return` ifadesine ulaşmalıdır. Null safety'den önce Dart eksik return'ler konusunda oldukça gevşekti. Örneğin:

```dart
// Without null safety:
String missingReturn() {
  // No return.
}
```

Bunu analiz edersen, belki bir return'ü unuttuğuna dair nazik bir ipucu alırdın, ama almasan da büyük bir sorun değildi. Çünkü çalışma bir fonksiyon gövdesinin sonuna ulaşırsa Dart örtük olarak `null` döndürür. Her tip nullable olduğundan, bu fonksiyon teknik olarak güvenlidir, muhtemelen istediğin şey olmasa da.

Sound non-nullable tiplerle bu program tamamen yanlış ve güvensizdir. Null safety altında, non-nullable dönüş tipine sahip bir fonksiyon güvenilir şekilde bir değer döndürmüyorsa derleme hatası alırsın. "Güvenilir şekilde" derken, dilin fonksiyon içindeki tüm kontrol akış yollarını analiz ettiğini kastediyorum. Hepsi bir şey döndürdüğü sürece memnun olur. Analiz oldukça akıllı, bu yüzden şu fonksiyon bile sorun değil:

```dart
// Using null safety:
String alwaysReturns(int n) {
  if (n == 0) {
    return 'zero';
  } else if (n < 0) {
    throw ArgumentError('Negative values not allowed.');
  } else {
    if (n > 1000) {
      return 'big';
    } else {
      return n.toString();
    }
  }
}
```

Yeni akış analizine (flow analysis) bir sonraki bölümde daha derinlemesine gireceğiz.

### Başlatılmamış değişkenler

Bir değişken tanımladığında, ona açık bir başlatıcı (initializer) vermezsen Dart değişkeni varsayılan olarak `null` ile başlatır. Bu kullanışlı, ama değişkenin tipi non-nullable ise tamamen güvensiz. Bu yüzden non-nullable değişkenler için işleri sıkılaştırmamız gerekiyor:

Üst düzey (top level) değişken ve static alan tanımlarının bir başlatıcısı olmalıdır. Bunlara programın herhangi bir yerinden erişilip atama yapılabildiğinden, derleyicinin değişkene kullanılmadan önce bir değer verildiğini garanti etmesi imkânsızdır. Tek güvenli seçenek, tanımın kendisinin doğru tipte bir değer üreten bir başlatıcı ifadeye sahip olmasını zorunlu kılmaktır:

```dart
// Using null safety:
int topLevel = 0;
 
class SomeClass {
  static int staticField = 0;
}
```

Örnek (instance) alanlarının ya tanımda bir başlatıcısı olmalı, ya bir initializing formal kullanmalı, ya da yapıcının (constructor) başlatma listesinde başlatılmalıdır. Bu çok jargon oldu. İşte örnekler:

```dart
// Using null safety:
class SomeClass {
  int atDeclaration = 0;
  int initializingFormal;
  int initializationList;
 
  SomeClass(this.initializingFormal)
      : initializationList = 0;
}
```

Başka bir deyişle, alan yapıcı gövdesine ulaşmadan önce bir değere sahip olduğu sürece sorun yok.

Yerel değişkenler en esnek durumdur. Non-nullable bir yerel değişkenin başlatıcısı olması gerekmez. Bu gayet sorunsuz:

```dart
// Using null safety:
int tracingFibonacci(int n) {
  int result;
  if (n < 2) {
    result = n;
  } else {
    result = tracingFibonacci(n - 2) + tracingFibonacci(n - 1);
  }
 
  print(result);
  return result;
}
```

Kural sadece, bir yerel değişkenin kullanılmadan önce *kesin olarak atanmış* (definitely assigned) olması gerektiği. Bunun için de değindiğim yeni akış analizine güveniyoruz. Bir değişkenin kullanımına giden her yol önce onu başlattığı sürece, kullanım sorunsuzdur.

Opsiyonel parametrelerin varsayılan bir değeri olmalıdır. Opsiyonel bir konumsal ya da isimli parametre için argüman geçirmezsen, dil onu varsayılan değerle doldurur. Varsayılan bir değer belirtmezsen, *varsayılanın varsayılanı* `null`'dır ve bu, parametrenin tipi non-nullable ise kabul edilmez.

Yani bir parametrenin opsiyonel olmasını istiyorsan, ya onu nullable yapmalı ya da geçerli, `null` olmayan bir varsayılan değer belirtmelisin.

Bu kısıtlamalar bunaltıcı gelebilir, ama pratikte o kadar kötü değiller. `final` değişkenlerle ilgili mevcut kısıtlamalara çok benziyorlar ve muhtemelen yıllardır bunlarla, fark etmeden bile çalışıyorsun. Ayrıca bunların yalnızca *non-nullable* değişkenler için geçerli olduğunu unutma. Tipi her zaman nullable yapıp varsayılan olarak `null` ile başlatılmasını sağlayabilirsin.

Yine de bu kurallar sürtünmeye yol açıyor. Neyse ki, bu yeni sınırlamaların seni yavaşlattığı en yaygın kalıpları yağlayacak bir dizi yeni dil özelliğimiz var. Ama önce, akış analizinden (flow analysis) bahsetmenin zamanı geldi.

Flow analizi

Kontrol akışı analizi, derleyicilerde yıllardır mevcut. Çoğunlukla kullanıcılardan gizlidir ve derleyici optimizasyonu sırasında kullanılır, ancak bazı yeni diller aynı teknikleri görünür dil özellikleri için kullanmaya başladı. Dart'ta zaten tür yükseltmesi (type promotion) şeklinde bir miktar akış analizi bulunmaktadır:

```dart
// With (or without) null safety:
bool isEmptyList(Object object) {
  if (object is List) {
    return object.isEmpty; // <-- OK!
  } else {
    return false;
  }
}
```

İşaretli satırda `object` üzerinde `isEmpty` çağırabildiğimize dikkat edin. Bu metot `Object` üzerinde değil, `List` üzerinde tanımlanmıştır. Bunun nedeni, tür denetleyicisinin programdaki tüm `is` ifadelerini ve kontrol akışı yollarını incelemesidir. Bir kontrol akışı yapısının gövdesi yalnızca bir değişken üzerindeki belirli bir `is` ifadesi doğru olduğunda yürütülüyorsa, o gövde içinde değişkenin türü test edilen türe "yükseltilir".

Buradaki örnekte, `if` ifadesinin `then` dalı yalnızca `object` gerçekten bir liste içerdiğinde çalışır. Bu nedenle Dart, `object`'i bildirilen `Object` türü yerine `List` türüne yükseltir. Bu kullanışlı bir özelliktir, ancak oldukça sınırlıdır. Null güvenliğinden önce, işlevsel olarak aynı olan aşağıdaki program çalışmazdı:

```dart
// Without null safety:
bool isEmptyList(Object object) {
  if (object is! List) return false;
  return object.isEmpty; // <-- Error!
}
```

Yine, `.isEmpty` çağrısına yalnızca `object` bir liste içerdiğinde ulaşabilirsiniz, bu nedenle bu program dinamik olarak doğrudur. Ancak tür yükseltme kuralları, `return` ifadesinden sonraki ifadeye yalnızca `object` bir liste olduğunda ulaşılabileceğini anlayacak kadar akıllı değildi.

Null güvenliği için, bu sınırlı analizi alıp çeşitli şekillerde çok daha güçlü hale getirdik.

Reachability analizi

Öncelikle, tür yükseltmesinin erken dönüşler ve diğer erişilemeyen kod yolları konusunda akıllı davranmadığına dair uzun süredir devam eden şikayetleri giderdik. Bir fonksiyonu analiz ederken, artık `return`, `break`, `throw` ve bir fonksiyonda yürütmenin erken sonlanabileceği diğer tüm yolları dikkate alıyor. Null güvenliği altında, bu fonksiyon:

```dart
// Using null safety:
bool isEmptyList(Object object) {
  if (object is! List) return false;
  return object.isEmpty;
}
```

Artık tamamen geçerli. `if` ifadesi, `object` bir `List` olmadığında fonksiyondan çıkacağı için, Dart ikinci ifadede `object`'i `List`'e yükseltiyor. Bu, null değerlerle ilgisi olmayan şeyler de dahil olmak üzere birçok Dart koduna yardımcı olan gerçekten güzel bir iyileştirme.

Unreachable kod için Never

Bu erişilebilirlik analizini siz de programlayabilirsiniz. Yeni alt tür `Never`'ın hiçbir değeri yoktur. (Aynı anda `String`, `bool` ve `int` olan bir değer ne tür bir değerdir?) Peki bir ifadenin `Never` türüne sahip olması ne anlama gelir? Bu, ifadenin hiçbir zaman başarıyla değerlendirilmesini tamamlayamayacağı anlamına gelir. Bir istisna fırlatmalı, işlemi durdurmalı veya ifadenin sonucunu bekleyen çevreleyen kodun asla çalışmamasını sağlamalıdır.

Aslında dile göre bir `throw` ifadesinin statik türü `Never`'dır. Bu tür temel kütüphanelerde tanımlanır ve bir tür açıklaması olarak kullanabilirsiniz. Belki belirli bir tür istisnayı fırlatmayı kolaylaştırmak için bir yardımcı fonksiyonunuz vardır:

```dart
// Using null safety:
Never wrongType(String type, Object value) {
  throw ArgumentError('Expected $type, but was ${value.runtimeType}.');
}
```

Şöyle kullanabilirsiniz:

```dart
// Using null safety:
class Point {
  final int x, y;

  Point(this.x, this.y);

  Point operator +(Object other) {
    if (other is int) return Point(x + other, y + other);
    if (other is! Point) wrongType('int | Point', other);

    print('Adding two Point instances together: $this + $other');
    return Point(x + other.x, y + other.y);
  }

  // toString, hashCode, and other implementations...
}
```

Bu program hatasız analiz ediliyor. Metodun son satırının `other.x` ve `other.y`'ye eriştiğine dikkat edin. Fonksiyonun herhangi bir `return` veya `throw` ifadesi olmamasına rağmen, `other` `Point`'e yükseltildi. Kontrol akışı analizi, `wrongType()`'ın bildirilen dönüş türünün `Never` olduğunu biliyor; bu da `if` ifadesinin `then` dalının bir şekilde sonlandırılması gerektiği anlamına geliyor. Son ifadeye yalnızca `other` bir `Point` olduğunda ulaşılabildiğinden, Dart onu yükseltiyor.

Başka bir deyişle, `Never`'ı kendi API'lerinizde kullanmak, Dart'ın erişilebilirlik analizini genişletmenizi sağlar.

Definite assignment analizi

Yerel değişkenlerle ilgili olarak bundan kısaca bahsetmiştim. Dart, null değer alamayan yerel bir değişkenin okunmadan önce her zaman başlatılmasını sağlamalıdır. Bunu olabildiğince esnek bir şekilde yapabilmek için *kesin atama analizini* (definite assignment analysis) kullanıyoruz. Dil, her fonksiyon gövdesini analiz eder ve tüm kontrol akışı yolları boyunca yerel değişkenlere ve parametrelere yapılan atamaları izler. Değişken, kullanıldığı her yolda atandığı sürece başlatılmış kabul edilir. Bu, değişkeni başlatıcı olmadan bildirmenize ve daha sonra karmaşık kontrol akışı kullanarak başlatmanıza olanak tanır, hatta değişken null değer alamayan bir türe sahip olsa bile.

Ayrıca `final` değişkenleri daha esnek hale getirmek için de kesin atama analizini kullanıyoruz. Null güvenliği öncesinde, `final` yerel değişkenleri ilginç bir şekilde başlatmanız gerekiyorsa bunu yapmak zor olabilirdi:

```dart
// Using null safety:
int tracingFibonacci(int n) {
  final int result;
  if (n < 2) {
    result = n;
  } else {
    result = tracingFibonacci(n - 2) + tracingFibonacci(n - 1);
  }

  print(result);
  return result;
}
```

Bu bir hata olurdu çünkü `result` değişkeni `final` ancak başlatıcısı yok. Null güvenliği altındaki daha akıllı akış analiziyle bu program sorunsuz çalışır. Analiz, `result` değişkeninin her kontrol akışı yolunda kesinlikle bir kez başlatıldığını söyleyebilir, bu nedenle bir değişkeni `final` olarak işaretlemek için gereken kısıtlamalar karşılanmıştır.

Null check'lerde type promotion

Daha akıllı akış analizi, null değerlerle ilgisi olmayan kodlar da dahil olmak üzere birçok Dart koduna yardımcı oluyor. Ancak bu değişiklikleri şimdi yapmamız tesadüf değil. Türleri null değer alabilen ve alamayan kümeler olarak ayırdık. Null değer alabilen bir türün değerine sahipseniz, onunla gerçekten faydalı bir şey yapamazsınız. Değerin `null` olduğu durumlarda bu kısıtlama iyidir. Çökmenizi önler.

Ancak değer `null` değilse, üzerinde metotlar çağırabilmek için onu null değer alamayan tarafa taşımak iyi olurdu. Akış analizi, yerel değişkenler ve parametreler (ve Dart 3.2'den itibaren özel final alanlar) için bunu yapmanın başlıca yollarından biridir. Tür yükseltmesini, `== null` ve `!= null` ifadelerini de inceleyecek şekilde genişlettik.

Yerel bir değişkenin `null` olup olmadığını kontrol ederseniz, Dart bu değişkeni temelindeki null değer alamayan türe yükseltir:

```dart
// Using null safety:
String makeCommand(String executable, [List<String>? arguments]) {
  var result = executable;
  if (arguments != null) {
    result += ' ' + arguments.join(' ');
  }
  return result;
}
```

Burada `arguments` null değer alabilen bir türe sahip. Normalde bu, üzerinde `.join()` metodunu çağırmanızı engeller. Ancak değerin `null` olmadığından emin olmak için kontrol eden bir `if` ifadesiyle bu çağrıyı koruduğumuz için, Dart onu `List<String>?` türünden `List<String>` türüne yükseltir ve üzerinde metot çağırmanıza veya null değer alamayan listeler bekleyen fonksiyonlara geçirmenize olanak tanır.

Bu oldukça önemsiz bir şey gibi görünebilir, ancak null kontrollerine dayalı bu akış tabanlı yaklaşım, mevcut Dart kodlarının çoğunun null güvenliği altında çalışmasını sağlar. Çoğu Dart kodu dinamik olarak doğrudur ve metotları çağırmadan önce `null` kontrolü yaparak null referans hatalarını önler. Null kontrolleri için yeni akış analizi, bu dinamik doğruluğu kanıtlanabilir statik doğruluğa dönüştürür.

Elbette, erişilebilirlik için yaptığımız daha akıllı analizlerle de uyumlu çalışıyor. Yukarıdaki fonksiyon şu şekilde de yazılabilir:

```dart
// Using null safety:
String makeCommand(String executable, [List<String>? arguments]) {
  var result = executable;
  if (arguments == null) return result;
  return result + ' ' + arguments.join(' ');
}
```

Dil, hangi tür ifadelerin tür yükseltmesine neden olacağı konusunda da daha akıllıca davranıyor. Açık bir `== null` veya `!= null` ifadesi elbette işe yarıyor. Ancak `as` ile yapılan açık tür dönüştürmeleri, atamalar veya sonek `!` operatörü (ki bunu daha sonra ele alacağız) de tür yükseltmesine neden oluyor. Genel amaç, kod dinamik olarak doğruysa ve bunu statik olarak anlamak mantıklıysa, analizin bunu yapacak kadar akıllı olmasıdır.

Unutmayın ki, tür yükseltme özelliği başlangıçta yalnızca yerel değişkenlerde çalışıyordu ve Dart 3.2'den itibaren özel final alanlarında da çalışmaktadır. Yerel olmayan değişkenlerle çalışma hakkında daha fazla bilgi için, "Working with nullable fields" bölümüne bakın.

Gereksiz kod uyarıları

Daha akıllı erişilebilirlik analizi yapmak ve `null` değerinin programınızda hangi noktalardan akabileceğini bilmek, bu durumu ele almak için kod eklemenizi sağlar. Ancak aynı analizi, ihtiyacınız olmayan `null` kodunu tespit etmek için de kullanabiliriz. Null güvenliği öncesinde, şöyle bir şey yazsaydınız:

```dart
// Using null safety:
String checkList(List<Object> list) {
  if (list?.isEmpty ?? false) {
    return 'Got nothing';
  }
  return 'Got something';
}
```

Dart, bu null-aware `?.` operatörünün yararlı olup olmadığını bilmenin bir yoluna sahip değildi. Bildiği kadarıyla, fonksiyona `null` değerini geçirebilirdiniz. Ancak null-safe Dart'ta, bu fonksiyonda `list`'i null değer alamayan bir `List` türüyle işaretlediyseniz, `list`'in asla `null` olmayacağını bilir. Bu, `?.` operatörünün asla yararlı bir şey yapmayacağı ve sadece `.` kullanmanız gerektiği anlamına gelir.

Kodunuzu basitleştirmenize yardımcı olmak için, statik analiz artık bunu tespit edebilecek kadar hassas olduğundan, bu gibi gereksiz kodlar için uyarılar ekledik. Null-aware bir operatörü veya `== null` ya da `!= null` gibi bir kontrolü null değer alamayan bir tür üzerinde kullanmak uyarı olarak raporlanır.

Ve elbette, bu durum aynı zamanda null değer alamayan tür yükseltmesiyle de ilgilidir. Bir değişken null değer alamayan bir türe yükseltildikten sonra, onu gereksiz yere tekrar `null` için kontrol ederseniz bir uyarı alırsınız:

```dart
// Using null safety:
String checkList(List<Object>? list) {
  if (list == null) return 'No list';
  if (list?.isEmpty ?? false) {
    return 'Empty list';
  }
  return 'Got something';
}
```

Burada `?.` için bir uyarı alıyorsunuz çünkü kod çalıştırıldığı anda `list`'in `null` olmasının mümkün olmadığını zaten biliyoruz. Bu uyarıların amacı sadece gereksiz kodları temizlemek değil. Gereksiz `null` kontrollerini kaldırarak, geriye kalan anlamlı kontrollerin öne çıkmasını sağlıyoruz. Kodunuza baktığınızda `null`'ın nerede oluşabileceğini görebilmenizi istiyoruz.

Nullable türlerle çalışma

Artık `null` değeri null değer alabilen türler kümesine dahil. Akış analizi ile, `null` olmadığını kanıtlayabildiğimiz bazı değerlerin güvenli bir şekilde null değer alamayan tarafa geçmesine ve orada kullanılmasına izin verebiliriz. Bu büyük bir adım, ancak burada durursak, ortaya çıkan sistem hala oldukça kısıtlayıcı olur. Akış analizi yalnızca yerel değişkenler, parametreler ve özel final alanlar için yardımcı olur.

Dart'ın null güvenliği öncesindeki esnekliğinin mümkün olduğunca çoğunu geri kazanmak ve bazı noktalarda onu aşmak için, bir avuç yeni özellik daha ekledik.

Daha akıllı null-aware metotlar

Dart'ın null-aware operatörü `?.`, null güvenliğinden çok daha eskidir. Çalışma zamanı semantiği, alıcı `null` ise sağ taraftaki özellik erişiminin atlandığını ve ifadenin `null` olarak değerlendirildiğini belirtir:

```dart
// Without null safety:
String notAString = null;
print(notAString?.length);
```

Bu, bir istisna fırlatmak yerine "null" yazdırır. Null-aware operatörü, Dart'ta null değer alabilen türleri kullanılabilir hale getirmek için güzel bir araçtır. Null değer alabilen türler üzerinde metot çağırmanıza izin veremesek de, bunlar üzerinde null-aware operatörlerini kullanmanıza izin verebiliriz ve veriyoruz da. Programın null güvenliği sonrası sürümü şöyledir:

```dart
// Using null safety:
String? notAString = null;
print(notAString?.length);
```

Öncekiyle tamamen aynı şekilde çalışıyor.

Ancak Dart'ta null-aware operatörleri kullandıysanız, metot zincirlerinde kullanırken muhtemelen can sıkıcı bir durumla karşılaşmışsınızdır. Diyelim ki, potansiyel olarak eksik bir dizenin uzunluğunun çift sayı olup olmadığını görmek istiyorsunuz (pek gerçekçi bir sorun değil, biliyorum, ama lütfen beni dinleyin):

```dart
// Using null safety:
String? notAString = null;
print(notAString?.length.isEven);
```

Bu program `?.` kullanmasına rağmen çalışma zamanında yine de bir istisna fırlatıyor. Sorun şu ki, `.isEven` ifadesinin alıcısı, solundaki tüm ifadenin, yani `notAString?.length` ifadesinin sonucudur. Bu ifade `null` olarak değerlendirilir, bu nedenle `null` üzerinde `.isEven` çağırmaya çalışırken bir null referans hatası alıyoruz. Dart'ta null-aware operatörünü kullandıysanız, muhtemelen bir kez kullandıktan sonra zincirdeki her özellik veya metoda `?.` uygulamanız gerektiğini zor yoldan öğrenmişsinizdir:

```dart
String? notAString = null;
print(notAString?.length?.isEven);
```

Bu can sıkıcı, ama daha da kötüsü, önemli bilgileri gizliyor. Şunu düşünün:

```dart
// Using null safety:
showGizmo(Thing? thing) {
  print(thing?.doohickey?.gizmo);
}
```

İşte size bir soru: `Thing` üzerindeki `doohickey` getter'ı `null` döndürebilir mi? Sonuç üzerinde `?.` kullandığınız için döndürebilirmiş gibi görünüyor. Ancak ikinci `?.` operatörünün yalnızca `doohickey` sonucunu değil, `thing`'in `null` olduğu durumları ele almak için orada olması da mümkün. Bunu bilemeyiz.

Bu sorunu çözmek için, C#'ın aynı özelliğin tasarımından akıllı bir fikir ödünç aldık. Bir metot zincirinde null-aware bir operatör kullandığınızda, alıcı `null` olarak değerlendirilirse, metot zincirinin geri kalanının tamamı kısa devre edilir ve atlanır. Bu, `doohickey` null değer alamayan bir dönüş türüne sahipse, şunu yazabileceğiniz ve yazmanız gerektiği anlamına gelir:

```dart
// Using null safety:
void showGizmo(Thing? thing) {
  print(thing?.doohickey.gizmo);
}
```

Aslında, bunu yapmazsanız ikinci `?.` için gereksiz kod uyarısı alırsınız. Eğer şöyle bir kod görürseniz:

```dart
// Using null safety:
void showGizmo(Thing? thing) {
  print(thing?.doohickey?.gizmo);
}
```

O zaman `doohickey`'nin kendisinin de null değer alabilen bir dönüş tipine sahip olduğunu kesin olarak bilirsiniz. Her `?.`, metot zincirine `null`'ın akmasına neden olabilecek benzersiz bir yola karşılık gelir. Bu, metot zincirlerindeki null-aware operatörleri hem daha özlü hem de daha hassas hale getirir.

Bu arada, birkaç tane daha null-aware operatör ekledik:

```dart
// Using null safety:

// Null-aware cascade:
receiver?..method();

// Null-aware index operator:
receiver?[index];
```

Null-aware bir fonksiyon çağrı operatörü yok, ancak şöyle yazabilirsiniz:

```dart
// Allowed with or without null safety:
function?.call(arg1, arg2);
```

Non-null assertion operatörü

Akış analizini kullanarak null değer alabilen bir değişkeni null değer alamayan bir değişkene taşımanın en güzel yanı, bunun kanıtlanabilir şekilde güvenli olmasıdır. Null değer alamayan türlerin güvenliğinden veya performansından ödün vermeden, daha önce null değer alabilen değişken üzerinde metotlar çağırabilirsiniz.

Ancak null değer alabilen türlerin birçok geçerli kullanımının, statik analizi tatmin edecek şekilde güvenli olduğu kanıtlanamaz. Örneğin:

```dart
// Using null safety, incorrectly:
class HttpResponse {
  final int code;
  final String? error;

  HttpResponse.ok()
      : code = 200,
        error = null;
  HttpResponse.notFound()
      : code = 404,
        error = 'Not found';

  @override
  String toString() {
    if (code == 200) return 'OK';
    return 'ERROR $code ${error.toUpperCase()}';
  }
}
```

Bunu çalıştırmayı denerseniz, `toUpperCase()` çağrısında derleme hatası alırsınız. `error` alanı null değer alabilir çünkü başarılı bir yanıtta bir değeri olmayacaktır. Sınıfı inceleyerek, `error` mesajı `null` olduğunda ona asla erişmediğimizi görebiliriz. Ancak bu, `code` değeri ile `error`'ın null değer alabilirliği arasındaki ilişkiyi anlamayı gerektirir. Tür denetleyicisi bu bağlantıyı göremez.

Başka bir deyişle, kodun insan yöneticileri olarak, `error`'ın kodu kullandığımız noktada `null` olmayacağını biliyoruz ve bunu doğrulamak için bir yönteme ihtiyacımız var. Normalde, türleri `as` ile bir tür dönüştürme (cast) kullanarak doğrularsınız ve burada da aynı şeyi yapabilirsiniz:

```dart
// Using null safety:
String toString() {
  if (code == 200) return 'OK';
  return 'ERROR $code ${(error as String).toUpperCase()}';
}
```

`error`'ı null değer alamayan `String` türüne dönüştürme işlemi başarısız olursa çalışma zamanı hatası verir. Aksi takdirde, üzerinde metotlar çağırabileceğimiz null değer alamayan bir dize elde ederiz.

"Null değer alabilme özelliğini ortadan kaldırma" işlemi o kadar sık karşımıza çıkıyor ki, yeni bir kısaltılmış sözdizimimiz var. Sondaki ünlem işareti (`!`) soldaki ifadeyi alır ve onu temelindeki null değer alamayan türe dönüştürür. Dolayısıyla yukarıdaki fonksiyon şuna eşdeğerdir:

```dart
// Using null safety:
String toString() {
  if (code == 200) return 'OK';
  return 'ERROR $code ${error!.toUpperCase()}';
}
```

Bu tek karakterlik "ünlem işareti operatörü", özellikle temel tür uzun ve ayrıntılı olduğunda çok kullanışlıdır. Null değer alabilme özelliğini kaldırmak için `as Map<TransactionProviderFactory, List<Set<ResponseFilter>>>` yazmak gerçekten can sıkıcı olurdu.

Elbette, herhangi bir tür dönüştürme işleminde olduğu gibi, `!` kullanmak statik güvenliğin kaybına yol açar. Doğruluğu korumak için dönüştürme işlemi çalışma zamanında kontrol edilmelidir ve başarısız olup bir istisna fırlatabilir. Ancak bu dönüştürme işlemlerinin nereye eklendiğini kontrol edebilirsiniz ve kodunuzu inceleyerek bunları her zaman görebilirsiniz.

Late değişkenler

Tür denetleyicisinin kodun güvenliğini kanıtlayamadığı en yaygın yer, üst düzey değişkenler ve alanlardır. İşte bir örnek:

```dart
// Using null safety, incorrectly:
class Coffee {
  String _temperature;

  void heat() { _temperature = 'hot'; }
  void chill() { _temperature = 'iced'; }

  String serve() => _temperature + ' coffee';
}

void main() {
  var coffee = Coffee();
  coffee.heat();
  coffee.serve();
}
```

Burada, `heat()` metodu `serve()` metodundan önce çağrılıyor. Bu, `_temperature` alanının kullanılmadan önce null olmayan bir değere başlatılacağı anlamına gelir. Ancak statik analizle bunu belirlemek mümkün değildir. (Bunun gibi basit bir örnek için mümkün olabilir, ancak bir sınıfın her örneğinin durumunu izlemeye çalışmanın genel durumu imkansızdır.)

Tür denetleyicisi alanların ve üst düzey değişkenlerin kullanımını analiz edemediği için, null değer alamayan alanların ya bildirim sırasında (ya da örnek alanlar için kurucu başlatma listesinde) başlatılması gerektiği konusunda muhafazakar bir kurala sahiptir. Bu nedenle Dart, bu sınıf için derleme hatası bildirir.

Alanı null değer alabilir hale getirerek ve ardından kullanımlarda null olmayan doğrulama operatörünü kullanarak hatayı düzeltebilirsiniz:

```dart
// Using null safety:
class Coffee {
  String? _temperature;

  void heat() { _temperature = 'hot'; }
  void chill() { _temperature = 'iced'; }

  String serve() => _temperature! + ' coffee';
}
```

Bu yöntem sorunsuz çalışıyor. Ancak sınıfın yöneticisine kafa karıştırıcı bir sinyal gönderiyor. `_temperature` alanını null değer alabilir olarak işaretleyerek, o alan için kullanışlı ve anlamlı bir değer olarak `null`'ın var olduğunu ima ediyorsunuz. Ama amaç bu değil. `_temperature` alanı, mevcut durumunda asla `null` olarak gözlemlenmemelidir.

Gecikmeli başlatma içeren durumların yaygın kullanım şeklini ele almak için yeni bir değiştirici ekledik: `late`. Bunu şu şekilde kullanabilirsiniz:

```dart
// Using null safety:
class Coffee {
  late String _temperature;

  void heat() { _temperature = 'hot'; }
  void chill() { _temperature = 'iced'; }

  String serve() => _temperature + ' coffee';
}
```

Dikkat edin, `_temperature` alanı null değer alamayan bir türe sahip ancak başlatılmamış. Ayrıca, kullanıldığında açık bir null olmayan doğrulama da yok. `late` değiştiricisinin semantiğine uygulayabileceğiniz birkaç model var, ancak ben bunu şöyle düşünüyorum: `late` değiştiricisi, "bu değişkenin kısıtlamalarını derleme zamanında değil, çalışma zamanında uygula" anlamına gelir. Bu, "late" (geç) kelimesinin değişkenin garantilerini ne zaman uyguladığını açıklamasına neredeyse benziyor.

Bu durumda, alan kesin olarak başlatılmadığı için, alan her okunduğunda, ona bir değer atanıp atanmadığını kontrol etmek için çalışma zamanında bir kontrol eklenir. Atanmamışsa, bir istisna fırlatılır. Değişkene `String` türünü vermek "beni asla bir dizeden başka bir değerle görmemelisiniz" anlamına gelir ve `late` değiştiricisi "bunu çalışma zamanında doğrula" anlamına gelir.

Bir bakıma, `late` değiştiricisi `?` kullanmaktan daha "sihirli"dir, çünkü alanın herhangi bir kullanımı başarısız olabilir ve kullanım yerinde metinsel olarak görünür hiçbir şey yoktur. Ancak bu davranışı elde etmek için bildirimde `late` yazmanız gerekir ve bizce değiştiriciyi orada görmek, bunun sürdürülebilir olması için yeterince açık bir ifadedir.

Bunun karşılığında, null değer alabilen bir tür kullanmaktan daha iyi statik güvenlik elde edersiniz. Alanın türü artık null değer alamaz olduğundan, alana `null` atamaya çalışmak derleme hatasına neden olur. `late` değiştiricisi başlatmayı ertelemenizi sağlar, ancak yine de `String` türünü null değer alabilen bir değişken gibi ele almanızı engeller.

Lazy initialization

`late` değiştiricisi başka özel yeteneklere de sahip. Paradoksal gibi görünse de, başlatıcısı olan bir alanda da `late` kullanabilirsiniz:

```dart
// Using null safety:
class Weather {
  late int _temperature = _readThermometer();
}
```

Bunu yaptığınızda, başlatıcı *tembel* hale gelir. Örnek oluşturulur oluşturulmaz çalıştırılmak yerine, ertelenir ve alana ilk erişildiğinde tembelce çalıştırılır. Başka bir deyişle, üst düzey bir değişken veya statik alan üzerindeki bir başlatıcı gibi çalışır. Bu, başlatma ifadesinin maliyetli olduğu ve gerekli olmayabileceği durumlarda kullanışlı olabilir.

Başlatıcıyı tembel bir şekilde çalıştırmak, `late` değiştiricisini bir örnek alanında kullandığınızda size ekstra bir avantaj sağlar. Genellikle örnek alan başlatıcıları `this` üzerinden erişemez, çünkü tüm alan başlatıcıları tamamlanana kadar yeni nesneye erişemezsiniz. Ancak bir `late` alan söz konusu olduğunda bu artık geçerli değildir, bu nedenle `this` üzerinden örneğe erişebilir, metotları çağırabilir veya alanlara erişebilirsiniz.

Late final değişkenler

`late` değiştiricisini `final` ile de birleştirebilirsiniz:

```dart
// Using null safety:
class Coffee {
  late final String _temperature;

  void heat() { _temperature = 'hot'; }
  void chill() { _temperature = 'iced'; }

  String serve() => _temperature + ' coffee';
}
```

Normal `final` alanların aksine, alanı bildiriminde veya kurucu başlatma listesinde başlatmanız gerekmez. Daha sonra çalışma zamanında ona değer atayabilirsiniz. Ancak yalnızca bir kez değer atayabilirsiniz ve bu durum çalışma zamanında kontrol edilir. Birden fazla kez değer atamaya çalışırsanız (burada olduğu gibi hem `heat()` hem de `chill()` çağırarak), ikinci atama bir istisna fırlatır. Bu, sonunda başlatılan ve sonrasında değiştirilemez olan durumu modellemenin harika bir yoludur.

Başka bir deyişle, yeni `late` değiştiricisi, Dart'ın diğer değişken değiştiricileriyle birlikte Kotlin'deki `lateinit` ve Swift'teki `lazy` özelliğinin büyük bir bölümünü kapsıyor. Hatta biraz yerel tembel değerlendirme istiyorsanız, yerel değişkenlerde bile kullanabilirsiniz.

Required named parametreler

Bir parametrenin hiçbir zaman `null` olarak görülmemesini garanti etmek için, tür denetleyicisi tüm isteğe bağlı parametrelerin ya null değer alabilen bir türe ya da varsayılan bir değere sahip olmasını gerektirir. Peki ya null değer alamayan bir türe sahip ve varsayılan değeri olmayan adlandırılmış bir parametre istiyorsanız? Bu, çağıranın onu her zaman geçirmesini istediğiniz anlamına gelir. Başka bir deyişle, adlandırılmış ancak isteğe bağlı olmayan bir parametre istiyorsunuz.

Dart parametrelerinin çeşitli türlerini bu tabloyla görselleştiriyorum:

![Table of Dart parameter kinds](2026-guz-4takim-gorseller/image15.png)

Anlaşılmaz nedenlerden dolayı, Dart uzun zamandır bu tablonun üç köşesini desteklemiş ancak adlandırılmış+zorunlu kombinasyonunu boş bırakmıştır. Null güvenliği ile bunu doldurduk. Gerekli bir adlandırılmış parametreyi, parametrenin önüne `required` koyarak belirtirsiniz:

```dart
// Using null safety:
function({int? a, required int? b, int? c, required int? d}) {}
```

Burada tüm parametreler isimleriyle geçirilmelidir. `a` ve `c` parametreleri isteğe bağlıdır ve atlanabilir. `b` ve `d` parametreleri zorunludur ve geçirilmelidir. Zorunluluğun, null değer alabilme özelliğinden bağımsız olduğunu unutmayın. Null değer alabilen türlerde zorunlu adlandırılmış parametrelere ve (varsayılan değerleri varsa) null değer alamayan türlerde isteğe bağlı adlandırılmış parametrelere sahip olabilirsiniz.

Bence bu da, null güvenliğinden bağımsız olarak Dart'ı daha iyi yapan özelliklerden biri. Bana göre bu, dili daha eksiksiz hissettiriyor.

Abstract alanlar

Dart'ın güzel özelliklerinden biri de *tekdüze erişim ilkesini* (uniform access principle) desteklemesidir. İnsan dilinde bu, alanların getter ve setter'lardan ayırt edilemez olduğu anlamına gelir. Bir Dart sınıfındaki bir "özelliğin" hesaplanıp hesaplanmadığı veya depolanıp depolanmadığı bir uygulama detayıdır. Bu nedenle, soyut bir sınıf kullanarak bir arayüz tanımlarken, alan bildirimi kullanmak tipiktir:

```dart
abstract class Cup {
  Beverage contents;
}
```

Amaç, kullanıcıların yalnızca bu sınıfı uygulamaları ve onu genişletmemeleridir. Alan sözdizimi, basitçe bir getter/setter çifti yazmanın daha kısa bir yoludur:

```dart
abstract class Cup {
  Beverage get contents;
  set contents(Beverage);
}
```

Ancak Dart, bu sınıfın hiçbir zaman somut bir tür olarak kullanılmayacağını bilmiyor. Bu bildirimi gerçek bir `contents` alanı olarak görüyor. Ve ne yazık ki, bu alan null değer alamaz ve başlatıcısı yoktur, bu nedenle derleme hatası alırsınız.

Bir çözüm, ikinci örnekteki gibi açık soyut getter/setter bildirimleri kullanmaktır. Ancak bu biraz uzun olduğundan, null güvenliği ile birlikte açık soyut alan bildirimleri için de destek ekledik:

```dart
abstract class Cup {
  abstract Beverage contents;
}
```

Bu, ikinci örnekle tamamen aynı şekilde davranır. Verilen isim ve türle soyut bir getter ve setter tanımlar.

Nullable alanlarla çalışma

Bu yeni özellikler birçok yaygın kalıbı kapsıyor ve çoğu zaman null değer alamayan türlerle çalışmayı oldukça kolaylaştırıyor. Ancak yine de, deneyimlerimize göre, null değer alabilen alanlar hala zor olabiliyor. Alanı null değer alamaz hale getirebildiğiniz durumlarda sorun yok. Ancak birçok durumda, alanın bir değeri olup olmadığını kontrol etmeniz gerekiyor ve bu da onu `late` yerine null değer alabilir hale getirmeyi gerektiriyor, böylece `null` değerini gözlemleyebilirsiniz.

Hem private hem de final olarak tanımlanmış, null değer alabilen alanlar (bazı özel nedenler hariç) tür yükseltmesi yapabilir. Herhangi bir nedenle bir alanı private ve final yapamıyorsanız, yine de bir çözüm bulmanız gerekecektir.

Örneğin, bunun işe yarayacağını düşünebilirsiniz:

```dart
// Using null safety, incorrectly:
class Coffee {
  String? _temperature;

  void heat() { _temperature = 'hot'; }
  void chill() { _temperature = 'iced'; }

  void checkTemp() {
    if (_temperature != null) {
      print('Ready to serve ' + _temperature + '!');
    }
  }

  String serve() => _temperature! + ' coffee';
}
```

`checkTemp()` içinde, `_temperature` alanının `null` olup olmadığını kontrol ediyoruz. Değilse, ona erişiyoruz ve üzerinde `+` operatörünü çağırıyoruz. Maalesef, buna izin verilmiyor.

Akış tabanlı tür yükseltmesi yalnızca hem private hem de final olan alanlar için geçerlidir. Aksi takdirde, statik analiz, alanın değerinin kontrol ettiğiniz nokta ile kullandığınız nokta arasında değişmediğini kanıtlayamaz. (Patolojik durumlarda, alanın kendisinin, ikinci kez çağrıldığında `null` değeri döndüren bir alt sınıftaki bir getter tarafından geçersiz kılınabileceğini göz önünde bulundurun.)

Dolayısıyla, sağlamlığı önemsediğimiz için, public ve/veya final olmayan alanlar yükseltilmiyor ve yukarıdaki metot derlenmiyor. Bu can sıkıcı. Buradaki gibi basit durumlarda, en iyi çözüm alanın kullanımına bir `!` eklemektir. Gereksiz gibi görünse de, Dart bugün aşağı yukarı böyle davranıyor.

Yardımcı olabilecek bir diğer yöntem ise önce alanı yerel bir değişkene kopyalamak ve ardından bu değişkeni kullanmaktır:

```dart
// Using null safety:
void checkTemp() {
  var temperature = _temperature;
  if (temperature != null) {
    print('Ready to serve ' + temperature + '!');
  }
}
```

Tür yükseltmesi yerel değişkenler için de geçerli olduğundan, bu artık sorunsuz çalışıyor. Değeri değiştirmeniz gerekirse, yalnızca yerel değişkene değil, alana da geri kaydetmeyi unutmayın.

Bu ve diğer tür yükseltme sorunlarının çözümü hakkında daha fazla bilgi için, "Fixing type promotion failures" bölümüne bakın.

Nullability ve generics

Çoğu modern statik tipli dil gibi, Dart'ın da genel (generic) sınıfları ve genel metotları vardır. Bunlar, sezgisel görünmeyen ancak sonuçlarını düşündüğünüzde anlam kazanan birkaç şekilde null değer alabilirlik ile etkileşime girer. Birincisi, "bu tür null değer alabilir mi?" sorusu artık basit bir evet veya hayır sorusu değildir. Şunu düşünün:

```dart
// Using null safety:
class Box<T> {
  final T object;
  Box(this.object);
}

void main() {
  Box<String>('a string');
  Box<int?>(null);
}
```

`Box` tanımında, `T` null değer alabilen bir tür mü yoksa null değer alamayan bir tür mü? Gördüğünüz gibi, her iki türle de örneklendirilebilir. Cevap, `T`'nin *potansiyel olarak null değer alabilen* bir tür olduğudur. Genel bir sınıfın veya metodun gövdesi içinde, potansiyel olarak null değer alabilen bir tür, hem null değer alabilen türlerin hem de null değer alamayan türlerin tüm kısıtlamalarına sahiptir.

Birincisi, `Object` üzerinde tanımlanmış birkaç metot dışında hiçbir metodu çağıramayacağınız anlamına gelir. İkincisi ise, bu türdeki tüm alanları veya değişkenleri kullanmadan önce başlatmanız gerektiği anlamına gelir. Bu da tür parametreleriyle çalışmayı oldukça zorlaştırabilir.

Pratikte birkaç kalıp ortaya çıkar. Tür parametresinin herhangi bir tür ile örneklendirilebildiği koleksiyon benzeri sınıflarda, kısıtlamalarla başa çıkmanız gerekir. Çoğu durumda, buradaki örnekte olduğu gibi, bu, bir tür bağımsız değişkeniyle çalışmanız gerektiğinde, o türden bir değere erişebildiğinizden emin olmak anlamına gelir. Neyse ki, koleksiyon benzeri sınıflar nadiren elemanları üzerinde metot çağırır.

Değere erişiminizin olmadığı yerlerde, tür parametresinin kullanımını null değer alabilir hale getirebilirsiniz:

```dart
// Using null safety:
class Box<T> {
  T? object;
  Box.empty();
  Box.full(this.object);
}
```

`object` alanının bildirimindeki `?` işaretine dikkat edin. Artık alanın açıkça null değer alabilen bir türü var, bu nedenle başlatılmamış olarak bırakmak sorun değil.

Burada olduğu gibi bir tür parametresini `T?` ile null değer alabilir hale getirdiğinizde, null değer alabilme özelliğini ortadan kaldırmak için tür dönüştürme işlemi yapmanız gerekebilir. Bunu yapmanın doğru yolu, `!` operatörü yerine açık bir `as T` tür dönüştürmesi kullanmaktır:

```dart
// Using null safety:
class Box<T> {
  T? object;
  Box.empty();
  Box.full(this.object);

  T unbox() => object as T;
}
```

`!` operatörü, değer `null` ise her zaman hata fırlatır. Ancak tür parametresi null değer alabilen bir türle örneklendirilmişse, `null` değeri `T` için tamamen geçerli bir değerdir:

```dart
// Using null safety:
void main() {
  var box = Box<int?>.full(null);
  print(box.unbox());
}
```

Bu program hatasız çalışmalıdır. `as T` bunu sağlar; `!` ise bir istisna fırlatırdı.

Diğer genel türlerin, uygulanabilecek tür bağımsız değişkenlerinin türlerini kısıtlayan bazı sınırları vardır:

```dart
// Using null safety:
class Interval<T extends num> {
  T min, max;

  Interval(this.min, this.max);

  bool get isEmpty => max <= min;
}
```

Eğer sınır null değer alamazsa, tür parametresi de null değer alamaz. Bu, null değer alamayan türlerin kısıtlamalarına tabi olduğunuz anlamına gelir; alanları ve değişkenleri başlatılmamış bırakamazsınız. Buradaki örnek sınıfın, alanları başlatan bir kurucuya sahip olması gerekir.

Bu kısıtlamanın karşılığında, tür parametresinin sınırında tanımlanan değerler üzerinde herhangi bir metodu çağırabilirsiniz. Ancak null değer alamayan bir sınıra sahip olmak, genel sınıfınızın kullanıcılarının onu null değer alabilen bir tür bağımsız değişkeniyle örneklemesini engeller. Bu, çoğu sınıf için muhtemelen makul bir sınırlamadır.

Ayrıca null değer alabilen bir sınır da kullanabilirsiniz:

```dart
// Using null safety:
class Interval<T extends num?> {
  T min, max;

  Interval(this.min, this.max);

  bool get isEmpty {
    var localMin = min;
    var localMax = max;

    // No min or max means an open-ended interval.
    if (localMin == null || localMax == null) return false;
    return localMax <= localMin;
  }
}
```

Bu, sınıfın gövdesinde tür parametresini null değer alabilir olarak ele alma esnekliğine sahip olduğunuz, ancak aynı zamanda null değer alabilirliğin kısıtlamalarına da tabi olduğunuz anlamına gelir. Null değer alabilme sorununu önce çözmeden bu türdeki bir değişken üzerinde hiçbir şey çağıramazsınız. Buradaki örnekte, alanları yerel değişkenlere kopyalıyoruz ve `<=` işleminden önce null akış analizinin bunları null değer alamayan türlere yükseltmesi için bu yerel değişkenleri kontrol ediyoruz.

Null değer alabilen sınırın, kullanıcıların sınıfı null değer alamayan türlerle örneklemesini engellemediğini unutmayın. Null değer alabilen sınır, tür bağımsız değişkeninin null değer alabileceği anlamına gelir, olması gerektiği anlamına gelmez. (Aslında, bir `extends` yan tümcesi yazmazsanız, tür parametrelerindeki varsayılan sınır null değer alabilen `Object?` sınırıdır.) Null değer alabilen bir tür bağımsız değişkeni *gerektirmenin* bir yolu yoktur. Tür parametresinin kullanımlarının güvenilir bir şekilde null değer alabilir olmasını ve örtük olarak `null` ile başlatılmasını istiyorsanız, sınıfın gövdesi içinde `T?` kullanabilirsiniz.

Core library değişiklikleri

Dilde birkaç ufak değişiklik daha var, ancak bunlar önemsiz. Örneğin, `on` yan tümcesi içermeyen bir `catch` yan tümcesinin varsayılan türü artık `dynamic` yerine `Object`. `switch` ifadelerindeki fall-through (geçiş) analizi yeni akış analizini kullanıyor.

Sizin için gerçekten önemli olan kalan değişiklikler temel kütüphanelerde. Büyük Null Güvenliği Macerası'na başlamadan önce, temel kütüphanelerimizi dünyayı büyük ölçüde bozmadan null güvenli hale getirmenin bir yolu olmadığı konusunda endişelenmiştik. Ancak durum o kadar da vahim çıkmadı. Birkaç önemli değişiklik var, ancak çoğunlukla geçiş sorunsuz gerçekleşti. Çoğu temel kütüphane ya doğal olarak null değer alamayan türlere geçti ya da `null`'ı kabul edip null değer alabilen bir türle sorunsuz bir şekilde geçiş yaptı.

Ancak birkaç önemli nokta var:

Map index operatörü nullable'dır

Bu aslında bir değişiklik değil, daha çok bilinmesi gereken bir şey. `Map` sınıfındaki `[]` indeks operatörü, anahtar mevcut değilse `null` döndürür. Bu, operatörün dönüş türünün null değer alabileceği anlamına gelir: `V` yerine `V?`.

Anahtar mevcut olmadığında bir istisna fırlatacak şekilde metodu değiştirebilir ve daha kolay kullanımlı, null değer alamayan bir dönüş türü verebilirdik. Ancak indeks operatörünü kullanan ve `null` sonucunu kontrol ederek anahtarın mevcut olup olmadığını anlayan kodlar çok yaygın, analizimize göre tüm kullanımların yaklaşık yarısı bu şekilde. Bu kodun tamamını bozmak Dart ekosistemini alevlendirirdi.

Bunun yerine, çalışma zamanı davranışı aynıdır ve bu nedenle dönüş türü null değer alabilir olmak zorundadır. Bu, genellikle bir harita aramasının sonucunu hemen kullanamayacağınız anlamına gelir:

```dart
// Using null safety, incorrectly:
var map = {'key': 'value'};
print(map['key'].length); // Error.
```

Bu, null değer alabilen bir dize üzerinde `.length` çağrısı yapmaya çalışırken derleme hatasına neden olur. Anahtarın mevcut olduğunu bildiğiniz durumlarda, tür denetleyicisine `!` ile bilgi verebilirsiniz:

```dart
// Using null safety:
var map = {'key': 'value'};
print(map['key']!.length); // OK.
```

`Map`'e bunu sizin için yapacak başka bir metot eklemeyi düşündük: anahtarı arayın, bulunamazsa hata fırlatın, aksi takdirde null değer alamayan bir değer döndürün. Ama ona ne ad vereceğiz? `!` işaretinden daha kısa bir isim olamazdı ve metot adı, çağrı yerinde yerleşik anlamıyla birlikte bir `!` görmekten daha açık olmazdı. Bu nedenle, bir haritada bilinen, mevcut bir öğeye erişmenin en yaygın yolu `[]!` kullanmaktır. Buna alışırsınız.

Unnamed List constructor yok

`List` sınıfının isimsiz kurucusu, verilen boyutta yeni bir liste oluşturur ancak hiçbir elemanı başlatmaz. Eğer null değer alamayan bir türde bir liste oluşturup daha sonra bir elemana erişirseniz, bu durum sağlamlık garantilerinde çok büyük bir açık oluşturur.

Bunu önlemek için, kurucuyu tamamen kaldırdık. Null-safe kodda `List()` çağırmak, null değer alabilen bir türle bile hatadır. Bu kulağa korkutucu geliyor, ancak pratikte çoğu kod, liste değişmezleri, `List.filled()`, `List.generate()` kullanarak veya başka bir koleksiyonu dönüştürmenin sonucu olarak listeler oluşturur. Belirli bir türün boş bir listesini oluşturmak istediğiniz uç durum için yeni bir `List.empty()` kurucusu ekledik.

Tamamen başlatılmamış bir liste oluşturma yöntemi Dart'ta her zaman yersiz görünmüştür ve şimdi daha da öyle. Eğer bu yöntem yüzünden kodunuz bozulduysa, liste oluşturmanın diğer birçok yolundan birini kullanarak her zaman düzeltebilirsiniz.

Non-nullable listelerde daha büyük length ayarlanamaz

Bu pek bilinmiyor, ancak `List`'in `length` getter'ının karşılık gelen bir setter'ı da var. Listeyi kısaltmak için uzunluğu daha kısa bir değere ayarlayabilirsiniz. Ayrıca, listeyi başlatılmamış öğelerle doldurmak için daha uzun bir uzunluğa da ayarlayabilirsiniz.

Eğer bunu null değer alamayan bir türdeki listeyle yaparsanız, daha sonra yazılmamış öğelere eriştiğinizde sağlamlığı ihlal edersiniz. Bunu önlemek için, liste null değer alamayan bir öğe türüne sahipse ve siz onu daha uzun bir uzunluğa ayarlarsanız (ve yalnızca bu durumda) `length` setter'ı bir çalışma zamanı istisnası fırlatacaktır. Her türden listeyi kısaltmak hala sorunsuzdur ve null değer alabilen türlerdeki listeleri uzatabilirsiniz.

`ListBase` veya `ListMixin`'i genişleten ya da uygulayan kendi liste türlerinizi tanımlarsanız bunun önemli bir sonucu vardır. Bu türlerin her ikisi de, uzunluğu ayarlayarak eklenen öğe için yer açan bir `insert()` uygulaması sağlar. Bu, null güvenliğiyle başarısız olurdu, bu nedenle bunun yerine `ListMixin`'deki (`ListBase`'in paylaştığı) `insert()` uygulamasını `add()` çağıracak şekilde değiştirdik. Bu miras alınan `insert()` metodunu kullanabilmek istiyorsanız, özel liste sınıfınız bir `add()` tanımı sağlamalıdır.

Iterator.current'a iterasyondan önce veya sonra erişilemez

`Iterator` sınıfı, `Iterable` arayüzünü uygulayan bir türün öğelerini dolaşmak için kullanılan değiştirilebilir bir "imleç"tir. Herhangi bir öğeye erişmeden önce ilk öğeye ilerlemek için `moveNext()` metodunu çağırmanız beklenir. Bu metot `false` döndürdüğünde, sona ulaşmışsınız demektir ve daha fazla öğe yoktur.

Eskiden, ilk `moveNext()` çağrısından önce veya yineleme bittikten sonra çağrıldığında `current`, `null` değeri döndürürdü. Null güvenliği ile bu, `current` dönüş türünün `E` değil `E?` olması gerektiği anlamına gelir. Bu da her öğe erişiminin çalışma zamanı `null` kontrolü gerektireceği anlamına gelir.

Bu kontroller, neredeyse hiç kimsenin mevcut öğeye bu hatalı şekilde erişmediği göz önüne alındığında işe yaramaz olurdu. Bunun yerine, `current` değerinin türünü `E` olarak belirledik. Yineleme öncesinde veya sonrasında bu türde bir değer mevcut olmayabileceğinden, yineleyiciyi çağırmamanız gereken bir durumda çağırdığınızda davranışını tanımsız bıraktık. Çoğu `Iterator` uygulaması bir `StateError` fırlatır.

Özet

Bu, null güvenliğiyle ilgili tüm dil ve kütüphane değişikliklerine dair çok detaylı bir tur. Bir sürü şey var, ama bu oldukça büyük bir dil değişikliği. Daha da önemlisi, Dart'ın hala tutarlı ve kullanılabilir hissettirdiği bir noktaya ulaşmak istedik. Bu, sadece tip sistemini değil, etrafındaki bir dizi diğer kullanılabilirlik özelliğini de değiştirmeyi gerektiriyor. Null güvenliğinin sonradan eklenmiş gibi hissettirmesini istemedik.

Özetle, şu noktalara dikkat etmek gerekiyor:

- Türler varsayılan olarak null değer almaz ve sonlarına `?` eklenerek null değer alabilir hale getirilir.

- İsteğe bağlı parametreler null değer alabilir olmalı veya varsayılan bir değere sahip olmalıdır. Adlandırılmış parametreleri isteğe bağlı olmaktan çıkarmak için `required` kullanabilirsiniz. Null değer alamayan üst düzey değişkenler ve statik alanlar başlatıcılara sahip olmalıdır. Null değer alamayan örnek alanlar, kurucu gövdesi başlamadan önce başlatılmalıdır.

- Null-aware operatörlerden sonraki metot zincirleri, alıcı `null` ise kısa devre yapar. Yeni null-aware cascade (`?..`) ve indeks (`?[]`) operatörleri vardır. Sonek null olmayan doğrulama "bang" operatörü (`!`), null değer alabilen işlenenini temel null değer alamayan türe dönüştürür.

- Akış analizi, null değer alabilen yerel değişkenleri ve parametreleri (ve Dart 3.2'den itibaren özel final alanlarını) güvenli bir şekilde kullanılabilir, null değer alamayan değişkenlere yükseltmenizi sağlar. Yeni akış analizi ayrıca tür yükseltme, eksik dönüş değerleri, erişilemeyen kod ve değişken başlatma için daha akıllı kurallara sahiptir.

- `late` değiştiricisi, çalışma zamanı kontrolü pahasına, normalde kullanamayacağınız yerlerde null değer alamayan türleri ve `final` kullanmanıza olanak tanır. Ayrıca, tembel başlatmalı alanlar da sağlar.

- Başlatılmamış öğeleri önlemek için `List` sınıfı değiştirildi.

Son olarak, tüm bunları özümseyip kodunuzu null güvenliği dünyasına taşıdığınızda, derleyicilerin optimize edebileceği ve çalışma zamanı hatasının oluşabileceği her yerin kodunuzda görünür olduğu sağlam bir program elde edersiniz. Bunun için harcanan çabaya değeceğini umuyoruz.
