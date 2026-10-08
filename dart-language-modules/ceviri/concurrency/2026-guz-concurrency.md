
# Dart'ta Eşzamanlılık (Concurrency)

Birden fazla işlemci çekirdeğinde paralel kod çalıştırmayı sağlamak için isolate'ları kullanın.

Bu sayfa, Dart'ta eşzamanlı programlamanın nasıl çalıştığına dair kavramsal bir genel bakış sunar. Olay döngüsünü, asenkron dil özelliklerini ve isolate'ları üst düzeyde açıklar. Dart'ta eşzamanlılığı kullanmaya yönelik daha pratik kod örnekleri için [Asenkron programlama](https://dart.dev/language/async) sayfasını ve [Isolates](https://dart.dev/language/isolates) sayfasını okuyun.

Dart'ta eşzamanlı programlama, hem Future ve Stream gibi asenkron API'leri hem de süreçleri ayrı çekirdeklere taşımanıza olanak tanıyan *isolate*'ları ifade eder.

Tüm Dart kodu isolate'lar içinde çalışır; varsayılan ana isolate ile başlar ve isteğe bağlı olarak sonradan açıkça oluşturduğunuz isolate'lara genişleyebilir. Yeni bir isolate başlattığınızda, bu isolate'ın kendine ait yalıtılmış belleği ve kendine ait bir olay döngüsü olur. Dart'ta asenkron ve eşzamanlı programlamayı mümkün kılan şey olay döngüsüdür.

# Olay Döngüsü

Dart'ın çalışma zamanı modeli bir olay döngüsüne dayanır. Olay döngüsü; programınızın kodunu çalıştırmaktan, olayları toplayıp işlemekten ve daha fazlasından sorumludur.

Uygulamanız çalışırken tüm olaylar, *olay kuyruğu* adı verilen bir kuyruğa eklenir. Olaylar; kullanıcı arayüzünü yeniden çizme isteklerinden, kullanıcı dokunuşlarına ve tuş vuruşlarına, diskten gelen G/Ç'ye kadar her şey olabilir. Uygulamanız olayların hangi sırayla gerçekleşeceğini öngöremediği için, olay döngüsü olayları kuyruğa alındıkları sırayla, birer birer işler.

![event-loop](2026-guz-4takim-gorseller/image1.png)

Olay döngüsünün işleyişi aşağıdaki koda benzer:

```dart
while (eventQueue.waitForEvent()) {
  eventQueue.processNextEvent();
}
```

Bu örnek olay döngüsü senkrondur ve tek bir iş parçacığı üzerinde çalışır. Ancak çoğu Dart uygulamasının aynı anda birden fazla iş yapması gerekir. Örneğin bir istemci uygulaması, bir kullanıcının bir düğmeye dokunmasını dinlerken aynı zamanda bir HTTP isteği de yürütmek zorunda kalabilir. Bunu ele almak için Dart, [Future'lar, Stream'ler ve async-await](https://dart.dev/language/async) gibi birçok asenkron API sunar. Bu API'ler bu olay döngüsünün etrafında inşa edilmiştir.

Örneğin bir ağ isteği yapmayı ele alalım:

```dart
http.get('https://example.com').then((response) {
  if (response.statusCode == 200) {
    print('Success!');
  }
}
```

Bu kod olay döngüsüne ulaştığında, ilk yan tümceyi olan http.get'i hemen çağırır ve bir Future döndürür. Ayrıca olay döngüsüne, HTTP isteği sonuçlanana kadar then() yan tümcesindeki geri çağrıyı elinde tutmasını söyler. İstek sonuçlandığında, isteğin sonucunu argüman olarak ileterek bu geri çağrıyı çalıştırmalıdır.

![async-event-loop](2026-guz-4takim-gorseller/image2.png)

Olay döngüsünün Dart'taki diğer tüm asenkron olayları — örneğin [Stream](https://api.dart.dev/dart-async/Stream-class.html) nesneleri — ele alış biçimi de genel olarak bu modelle aynıdır.

# Asenkron Programlama

Bu bölüm, Dart'taki farklı asenkron programlama türlerini ve sözdizimlerini özetler. Future, Stream ve async-await konularına zaten aşinaysanız, doğrudan [isolate'lar bölümüne](#isolates) geçebilirsiniz.

## Future'lar

Bir Future, sonunda bir değerle ya da bir hatayla tamamlanacak olan asenkron bir işlemin sonucunu temsil eder.

Bu örnek kodda, Future\<String\> dönüş tipi, sonunda bir String değeri (ya da bir hata) sağlama sözünü temsil eder.

```dart
Future<String> _readFileAsync(String filename) {
  final file = File(filename);

  // .readAsString() returns a Future.
  // .then() registers a callback to be executed when `readAsString` resolves.
  return file.readAsString().then((contents) {
    return contents.trim();
  });
}
```

## async-await Sözdizimi

async ve await anahtar sözcükleri, asenkron fonksiyonları tanımlamak ve sonuçlarını kullanmak için bildirimsel bir yol sağlar.

Aşağıda, dosya G/Ç'sini beklerken bloke olan bazı senkron kodların bir örneği yer alıyor:

```dart
const String filename = 'with_keys.json';

void main() {
  // Read some data.
  final fileData = _readFileSync();
  final jsonData = jsonDecode(fileData);

  // Use that data.
  print('Number of JSON keys: ${jsonData.length}');
}

String _readFileSync() {
  final file = File(filename);
  final contents = file.readAsStringSync();
  return contents.trim();
}
```

İşte benzer bir kod; ancak onu asenkron hale getirmek için yapılan değişiklikler (vurgulanmış olarak) içeriyor:

```dart
const String filename = 'with_keys.json';

void main() async {
  // Read some data.
  final fileData = await _readFileAsync();
  final jsonData = jsonDecode(fileData);

  // Use that data.
  print('Number of JSON keys: ${jsonData.length}');
}

Future<String> _readFileAsync() async {
  final file = File(filename);
  final contents = await file.readAsString();
  return contents.trim();
}
```

main() fonksiyonu, yerel kod (dosya G/Ç'si) çalışırken diğer Dart kodlarının (örneğin olay işleyicilerinin) CPU'yu kullanabilmesi için \_readFileAsync() çağrısının önünde await anahtar sözcüğünü kullanır. await kullanmanın bir diğer etkisi de, \_readFileAsync() tarafından döndürülen Future\<String\> değerini bir String'e dönüştürmesidir. Sonuç olarak contents değişkeni, örtük olarak String tipine sahip olur.

> **Not** await anahtar sözcüğü yalnızca, fonksiyon gövdesinden önce async bulunan fonksiyonlarda çalışır.

Aşağıdaki şekilde görüldüğü gibi, readAsString() Dart çalışma zamanında ya da işletim sisteminde Dart dışı bir kod çalıştırırken Dart kodu duraklar. readAsString() bir değer döndürdüğünde, Dart kodunun yürütülmesi kaldığı yerden devam eder.

![basics-await](2026-guz-4takim-gorseller/image3.png)

## Stream'ler

Dart, asenkron kodu stream biçiminde de destekler. Stream'ler, değerleri gelecekte ve zaman içinde tekrar tekrar sağlar. Zaman içinde bir dizi int değeri sağlama sözü, Stream\<int\> tipine sahiptir.

Aşağıdaki örnekte, Stream.periodic ile oluşturulan stream, her saniye yeni bir int değeri yayar.

```dart
Stream<int> stream = Stream.periodic(const Duration(seconds: 1), (i) => i * i);
```

### await-for ve yield

Await-for, döngünün her bir sonraki yinelemesini yeni değerler sağlandıkça çalıştıran bir tür for döngüsüdür. Başka bir deyişle, stream'ler üzerinde "döngü kurmak" için kullanılır. Bu örnekte, argüman olarak verilen stream'den yeni değerler yayıldıkça sumStream fonksiyonundan da yeni bir değer yayılacaktır. Değer stream'leri döndüren fonksiyonlarda return yerine yield anahtar sözcüğü kullanılır.

```dart
Stream<int> sumStream(Stream<int> stream) async* {
  var sum = 0;
  await for (final value in stream) {
    yield sum += value;
  }
}
```

async, await, Stream'ler ve Future'lar hakkında daha fazla bilgi edinmek isterseniz [asenkron programlama eğitimine](https://dart.dev/libraries/async/async-await) göz atın.

# Isolate'lar

Dart, [asenkron API'lere](#asynchronous-programming) ek olarak isolate'lar aracılığıyla da eşzamanlılığı destekler. Çoğu modern cihaz çok çekirdekli CPU'lara sahiptir. Birden fazla çekirdekten yararlanmak için geliştiriciler bazen eşzamanlı çalışan, paylaşımlı bellekli iş parçacıkları kullanır. Ancak paylaşımlı durum eşzamanlılığı [hataya açıktır](https://en.wikipedia.org/wiki/Race_condition#In_software) ve karmaşık koda yol açabilir.

İş parçacıkları yerine, tüm Dart kodu isolate'lar içinde çalışır. Isolate'ları kullanarak Dart kodunuz, varsa ek işlemci çekirdeklerini de kullanarak aynı anda birden fazla bağımsız görev gerçekleştirebilir. Isolate'lar iş parçacıklarına veya süreçlere benzer; ancak her isolate'ın kendine ait belleği ve bir olay döngüsü çalıştıran tek bir iş parçacığı vardır.

Her isolate'ın kendi global alanları vardır; bu da bir isolate'daki hiçbir durumun başka bir isolate'dan erişilebilir olmamasını sağlar. Isolate'lar birbirleriyle yalnızca mesaj geçirerek iletişim kurabilir. Isolate'lar arasında paylaşılan durum olmaması, [mutex'ler veya kilitler](https://en.wikipedia.org/wiki/Lock_(computer_science)) ve [veri yarışları](https://en.wikipedia.org/wiki/Race_condition#Data_race) gibi eşzamanlılık karmaşıklıklarının Dart'ta yaşanmayacağı anlamına gelir. Bununla birlikte isolate'lar, yarış koşullarını tümüyle de önlemez. Bu eşzamanlılık modeli hakkında daha fazla bilgi için [Aktör modeli](https://en.wikipedia.org/wiki/Actor_model) hakkında okuyun.

> **Platform notu** Isolate'ları yalnızca [Dart Native platformu](https://dart.dev/overview#platform) uygular. Dart Web platformu hakkında daha fazla bilgi edinmek için [Web'de eşzamanlılık](#concurrency-on-the-web) bölümüne bakın.

## Ana Isolate

Çoğu durumda isolate'ları hiç düşünmenize gerek yoktur. Dart programları varsayılan olarak ana isolate'da çalışır. Bu, bir programın çalışmaya ve yürütülmeye başladığı iş parçacığıdır; aşağıdaki şekilde gösterildiği gibi:

![basics-main-isolate](2026-guz-4takim-gorseller/image4.png)

Tek isolate'lı programlar bile sorunsuz çalışabilir. Bu uygulamalar, bir sonraki kod satırına geçmeden önce asenkron işlemlerin tamamlanmasını beklemek için [async-await](https://dart.dev/libraries/async/async-await) kullanır. İyi davranışlı bir uygulama hızlı başlar ve olay döngüsüne mümkün olan en kısa sürede ulaşır. Ardından uygulama, gerektiğinde asenkron işlemlerden yararlanarak kuyruktaki her olaya hızla yanıt verir.

## Isolate Yaşam Döngüsü

Aşağıdaki şekilde görüldüğü gibi, her isolate main() fonksiyonu gibi bir miktar Dart kodu çalıştırarak başlar. Bu Dart kodu, örneğin kullanıcı girdisine veya dosya G/Ç'sine yanıt vermek için bazı olay dinleyicileri kaydedebilir. Isolate'ın başlangıç fonksiyonu döndüğünde, olayları işlemesi gerekiyorsa isolate ortada kalmaya devam eder. Olayları işledikten sonra isolate sonlanır.

![basics-isolate](2026-guz-4takim-gorseller/image5.png)

## Olay İşleme

Bir istemci uygulamasında, ana isolate'ın olay kuyruğu yeniden çizim istekleri ile dokunma ve diğer arayüz olaylarına ilişkin bildirimler içerebilir. Örneğin aşağıdaki şekil, bir yeniden çizim olayını, ardından bir dokunma olayını, ardından da iki yeniden çizim olayını göstermektedir. Olay döngüsü olayları kuyruktan ilk giren ilk çıkar sırasıyla alır.

![event-loop](2026-guz-4takim-gorseller/image1.png)

Olay işleme, main() sonlandıktan sonra ana isolate üzerinde gerçekleşir. Aşağıdaki şekilde, main() sonlandıktan sonra ana isolate önce ilk yeniden çizim olayını işler. Ardından ana isolate dokunma olayını, sonra da bir yeniden çizim olayını işler.

Bir senkron işlem çok fazla işlem süresi alırsa, uygulama yanıt vermez hale gelebilir. Aşağıdaki şekilde dokunmayı işleyen kod çok uzun sürdüğü için sonraki olaylar çok geç işlenmektedir. Uygulama donmuş gibi görünebilir ve gerçekleştirdiği her animasyon takılmalı olabilir.

![event-jank](2026-guz-4takim-gorseller/image6.png)

İstemci uygulamalarında, aşırı uzun süren bir senkron işlemin sonucu çoğunlukla [takılmalı (akıcı olmayan) arayüz animasyonlarıdır](https://docs.flutter.dev/perf/rendering-performance). Daha da kötüsü, arayüz tamamen yanıt vermez hale gelebilir.

## Arka Plan Çalışanları

Uygulamanızın arayüzü, zaman alan bir hesaplama nedeniyle (örneğin [büyük bir JSON dosyasını ayrıştırmak](https://docs.flutter.dev/cookbook/networking/background-parsing)) yanıt vermez hale geliyorsa, bu hesaplamayı genellikle *arka plan çalışanı* olarak adlandırılan bir çalışan isolate'a devretmeyi düşünün. Aşağıdaki şekilde gösterilen yaygın bir durum, bir hesaplama yapıp ardından sonlanan basit bir çalışan isolate başlatmaktır. Çalışan isolate, sonlanırken sonucunu bir mesaj içinde döndürür.

![isolate-bg-worker](2026-guz-4takim-gorseller/image7.png)

Bir çalışan isolate G/Ç gerçekleştirebilir (örneğin dosya okuyup yazabilir), zamanlayıcılar kurabilir ve daha fazlasını yapabilir. Kendine ait belleği vardır ve ana isolate ile hiçbir durumu paylaşmaz. Çalışan isolate, diğer isolate'ları etkilemeden bloke olabilir.

## Isolate'ları Kullanmak

Kullanım senaryosuna bağlı olarak, Dart'ta isolate'larla çalışmanın iki yolu vardır:

- Ayrı bir iş parçacığında tek bir hesaplama yapmak için [Isolate.run()](https://api.dart.dev/dart-isolate/Isolate/run.html) kullanın.

- Zaman içinde birden fazla mesajı işleyecek bir isolate ya da bir arka plan çalışanı oluşturmak için [Isolate.spawn()](https://api.dart.dev/dart-isolate/Isolate/spawn.html) kullanın. Uzun ömürlü isolate'larla çalışma hakkında daha fazla bilgi için [Isolates](https://dart.dev/language/isolates) sayfasını okuyun.

Çoğu durumda, süreçleri arka planda çalıştırmak için önerilen API Isolate.run'dır.

### Isolate.run()

Statik Isolate.run() metodu tek bir argüman gerektirir: yeni oluşturulan isolate üzerinde çalıştırılacak bir geri çağrı.

```dart
int slowFib(int n) => n <= 1 ? 1 : slowFib(n - 1) + slowFib(n - 2);

// Compute without blocking current isolate.
void fib40() async {
  var result = await Isolate.run(() => slowFib(40));
  print('Fib(40) = $result');
}
```

## Performans ve Isolate Grupları

Bir isolate [Isolate.spawn()](https://api.dart.dev/dart-isolate/Isolate/spawn.html) çağırdığında, iki isolate aynı çalıştırılabilir koda sahip olur ve aynı *isolate grubunda* yer alır. Isolate grupları, kod paylaşımı gibi performans optimizasyonlarını mümkün kılar; yeni bir isolate, isolate grubuna ait kodu hemen çalıştırır. Ayrıca Isolate.exit() yalnızca isolate'lar aynı isolate grubundayken çalışır.

Bazı özel durumlarda, yeni isolate'ı belirtilen URI'deki kodun bir kopyasıyla kuran [Isolate.spawnUri()](https://api.dart.dev/dart-isolate/Isolate/spawnUri.html) metodunu kullanmanız gerekebilir. Ancak spawnUri(), spawn()'dan çok daha yavaştır ve yeni isolate, kendisini başlatanın isolate grubunda yer almaz. Bir diğer performans sonucu da, isolate'lar farklı gruplardayken mesaj geçirmenin daha yavaş olmasıdır.

## Isolate'ların Sınırlamaları

### Isolate'lar iş parçacığı değildir

Dart'a çok iş parçacıklı bir dilden geliyorsanız, isolate'ların iş parçacıkları gibi davranmasını beklemek makul olurdu; ancak durum böyle değildir. Her isolate'ın kendi durumu vardır ve bu, bir isolate'daki hiçbir durumun başka bir isolate'dan erişilebilir olmamasını sağlar. Bu nedenle isolate'lar, yalnızca kendi belleklerine erişimle sınırlıdır.

Örneğin, global bir değiştirilebilir değişkene sahip bir uygulamanız varsa, bu değişken başlatılan isolate'da ayrı bir değişken olacaktır. Bu değişkeni başlatılan isolate'da değiştirirseniz, ana isolate'daki değişken olduğu gibi kalır. Isolate'lar tam da bu şekilde çalışmak üzere tasarlanmıştır ve isolate kullanmayı düşünürken bunu aklınızda tutmak önemlidir.

### Mesaj tipleri

[SendPort](https://api.dart.dev/dart-isolate/SendPort-class.html) üzerinden gönderilen mesajlar neredeyse her tür Dart nesnesi olabilir, ancak birkaç istisna vardır:

- [Socket](https://api.dart.dev/dart-io/Socket-class.html) gibi yerel kaynaklara sahip nesneler.

- [ReceivePort](https://api.dart.dev/dart-isolate/ReceivePort-class.html)

- [DynamicLibrary](https://api.dart.dev/dart-ffi/DynamicLibrary-class.html)

- [Finalizable](https://api.dart.dev/dart-ffi/Finalizable-class.html)

- [Finalizer](https://api.dart.dev/dart-core/Finalizer-class.html)

- [NativeFinalizer](https://api.dart.dev/dart-ffi/NativeFinalizer-class.html)

- [Pointer](https://api.dart.dev/dart-ffi/Pointer-class.html)

- [UserTag](https://api.dart.dev/dart-developer/UserTag-class.html)

- @pragma('vm:isolate-unsendable') ile işaretlenmiş sınıfların örnekleri.

Bu istisnalar dışında, her nesne gönderilebilir. Daha fazla bilgi için [SendPort.send](https://api.dart.dev/dart-isolate/SendPort/send.html) dokümantasyonuna göz atın.

Isolate.spawn() ve Isolate.exit() metotlarının SendPort nesneleri üzerinde bir soyutlama sağladığını, dolayısıyla aynı sınırlamalara tabi olduklarını unutmayın.

### Isolate'lar arasında senkron bloke edici iletişim

Paralel çalışabilecek isolate sayısında bir sınır vardır. Bu sınır, Dart'ta isolate'lar arasında mesajlar aracılığıyla yapılan standart *asenkron* iletişimi etkilemez. Yüzlerce isolate'ı eşzamanlı olarak çalıştırabilir ve ilerleme kaydettirebilirsiniz. Isolate'lar CPU üzerinde döngüsel biçimde zamanlanır ve birbirlerine sık sık yol verir.

Isolate'lar, saf Dart'ın dışında yalnızca *senkron* olarak iletişim kurabilir; bunu yapmak için de [FFI](https://dart.dev/interop/c-interop) aracılığıyla C kodu kullanılır. FFI çağrılarında senkron bloke ederek isolate'lar arasında senkron iletişim kurmaya çalışmak, özel bir dikkat gösterilmediği sürece, isolate sayısı sınırı aşarsa kilitlenmeye yol açabilir. Sınır belirli bir sayıya sabitlenmiş değildir; Dart uygulamasının kullanımına açık Dart VM yığın boyutuna göre hesaplanır.

Bu durumdan kaçınmak için, senkron bloke etme işlemini gerçekleştiren C kodunun, bloke edici işlemi yapmadan önce mevcut isolate'dan ayrılması ve FFI çağrısından Dart'a dönmeden önce isolate'a yeniden girmesi gerekir. Daha fazla bilgi edinmek için [Dart_EnterIsolate](https://github.com/dart-lang/sdk/blob/c9a8bbd8d6024e419b5e5f26b5131285eb19cc93/runtime/include/dart_api.h#L1254) ve [Dart_ExitIsolate](https://github.com/dart-lang/sdk/blob/c9a8bbd8d6024e419b5e5f26b5131285eb19cc93/runtime/include/dart_api.h#L1455) hakkında okuyun.

# Web'de Eşzamanlılık

Tüm Dart uygulamaları, bloke etmeyen, iç içe geçmiş hesaplamalar için async-await, Future ve Stream kullanabilir. Ancak [Dart web platformu](https://dart.dev/overview#platform) isolate'ları desteklemez. Dart web uygulamaları, betikleri isolate'lara benzer biçimde arka plan iş parçacıklarında çalıştırmak için [web çalışanlarını](https://developer.mozilla.org/docs/Web/API/Web_Workers_API/Using_web_workers) kullanabilir. Ancak web çalışanlarının işlevselliği ve yetenekleri isolate'lardan bir miktar farklıdır.

Örneğin web çalışanları iş parçacıkları arasında veri gönderirken veriyi ileri geri kopyalar. Ancak veri kopyalama, özellikle büyük mesajlarda çok yavaş olabilir. Isolate'lar da aynı şeyi yapar; ancak bunun yerine mesajı tutan belleği daha verimli biçimde *aktarabilen* API'ler de sunar.

Web çalışanları ile isolate'ları oluşturma biçimi de farklıdır. Web çalışanlarını yalnızca ayrı bir program giriş noktası tanımlayıp onu ayrı olarak derleyerek oluşturabilirsiniz. Bir web çalışanını başlatmak, bir isolate'ı başlatmak için Isolate.spawnUri kullanmaya benzer. Bir isolate'ı Isolate.spawn ile de başlatabilirsiniz; bu, kendisini başlatan isolate ile [kodun ve verinin bir kısmını yeniden kullandığı](#performance-and-isolate-groups) için daha az kaynak gerektirir. Web çalışanları için eşdeğer bir API yoktur.

# Ek Kaynaklar

- Çok sayıda isolate kullanıyorsanız, Flutter'da [IsolateNameServer](https://api.flutter.dev/flutter/dart-ui/IsolateNameServer-class.html) veya Flutter dışı Dart uygulamaları için benzer işlevsellik sağlayan [package:isolate_name_server](https://pub.dev/packages/isolate_name_server) paketini değerlendirin.

- Dart isolate'larının dayandığı [Aktör modeli](https://en.wikipedia.org/wiki/Actor_model) hakkında daha fazla bilgi edinin.

- Isolate API'lerine ilişkin ek dokümantasyon:

  - [Isolate.exit()](https://api.dart.dev/dart-isolate/Isolate/exit.html)

  - [`Isolate.spawn()`](https://api.dart.dev/dart-isolate/Isolate/spawn.html)

  - [`ReceivePort`](https://api.dart.dev/dart-isolate/ReceivePort-class.html)

  - [`SendPort`](https://api.dart.dev/dart-isolate/SendPort-class.html)

[Dil](https://dart.dev/language) › [Asenkron programlama](https://dart.dev/language/async)

# Asenkron programlama

Dart'ta asenkron kod yazma hakkında bilgiler.

Dart kütüphaneleri, [Future](https://api.dart.dev/dart-async/Future-class.html) veya [Stream](https://api.dart.dev/dart-async/Stream-class.html) nesneleri döndüren fonksiyonlarla doludur. Bu fonksiyonlar *asenkrondur*: zaman alması muhtemel bir işlemi (örneğin G/Ç) başlattıktan sonra, o işlemin tamamlanmasını beklemeden geri dönerler.

`async` ve `await` anahtar sözcükleri asenkron programlamayı destekler; böylece senkron koda benzeyen asenkron kod yazmanızı sağlarlar.

## Future'ları yönetmek

Tamamlanmış bir Future'ın sonucuna ihtiyaç duyduğunuzda iki seçeneğiniz vardır:

- `async` ve `await` kullanın; burada ve [asenkron programlama eğitiminde](https://dart.dev/libraries/async/async-await) açıklandığı gibi.

- Future API'sini kullanın; [dart:async belgelerinde](https://dart.dev/libraries/dart-async#future) açıklandığı gibi.

`async` ve `await` kullanan kod asenkrondur, ancak senkron koda çok benzer görünür. Örneğin, aşağıdaki kod bir asenkron fonksiyonun sonucunu beklemek için `await` kullanır:

```dart
await lookUpVersion();
```

``

`await` kullanmak için kodun bir `async` fonksiyonunda, yani `async` olarak işaretlenmiş bir fonksiyonda olması gerekir:

```dart
Future<void> checkVersion() async {
  var version = await lookUpVersion();
  // version ile bir şey yap
}
```

> **Not**
>
> Bir `async` fonksiyonu zaman alan işlemler gerçekleştirebilse de bu işlemleri beklemez. Bunun yerine `async` fonksiyonu yalnızca ilk `await` ifadesiyle karşılaşana kadar çalışır. Ardından bir `Future` nesnesi döndürür ve yürütmeye ancak `await` ifadesi tamamlandıktan sonra devam eder.

`await` kullanan koddaki hataları ele almak ve temizlik yapmak için `try`, `catch` ve `finally` kullanın:

```dart
try {
  version = await lookUpVersion();
} catch (e) {
  // Sürümün bulunamamasına tepki ver
}
```

Bir `async` fonksiyonunda `await` ifadesini birden çok kez kullanabilirsiniz. Örneğin, aşağıdaki kod fonksiyonların sonuçları için üç kez bekler:

```dart
var entrypoint = await findEntryPoint();
var exitCode = await runExecutable(entrypoint, args);
await flushThenExit(exitCode);
```

``

`await expression` içinde `expression` değeri genellikle bir Future'dır; değilse, değer otomatik olarak bir Future içine sarılır. Bu Future nesnesi, bir nesne döndüreceğine dair bir söz belirtir. `await expression` ifadesinin değeri, döndürülen o nesnedir. await ifadesi, bu nesne kullanılabilir olana kadar yürütmenin duraklamasına neden olur.

**await kullanırken derleme zamanı hatası alırsanız, await ifadesinin bir async fonksiyonunda olduğundan emin olun.** Örneğin, uygulamanızın `main()` fonksiyonunda `await` kullanmak için `main()` gövdesinin `async` olarak işaretlenmesi gerekir:

```dart
void main() async {
  checkVersion();
  print('In main: version is ${await lookUpVersion()}');
}
```

> **Not**

```dart
Önceki örnek, sonucunu beklemeden bir async fonksiyonunu (checkVersion()) kullanır; kod fonksiyonun çalışmasını bitirdiğini varsayıyorsa bu uygulama sorunlara yol açabilir. Bu sorunu önlemek için unawaited_futures linter kuralını kullanın.
```

Future'ların, `async` ve `await` kullanımının etkileşimli bir tanıtımı için [asenkron programlama eğitimine](https://dart.dev/libraries/async/async-await) bakın.

## async fonksiyonlarını tanımlamak

Bir `async` fonksiyonu, gövdesi `async` değiştiricisiyle işaretlenmiş bir fonksiyondur.

Bir fonksiyona `async` anahtar sözcüğünü eklemek, o fonksiyonun bir Future döndürmesini sağlar. Örneğin, bir String döndüren şu senkron fonksiyonu ele alalım:

```dart
String lookUpVersion() => '1.0.0';
```

Bunu bir `async` fonksiyonuna dönüştürürseniz (örneğin gelecekteki bir uygulama zaman alacağı için), döndürülen değer bir Future olur:

```dart
Future<String> lookUpVersion() async => '1.0.0';
```

Fonksiyonun gövdesinin Future API'sini kullanması gerekmediğine dikkat edin. Gerekirse Dart, Future nesnesini sizin için oluşturur. Fonksiyonunuz kullanışlı bir değer döndürmüyorsa, dönüş türünü `Future<void>` yapın.

Future'ların, `async` ve `await` kullanımının etkileşimli bir tanıtımı için [asenkron programlama eğitimine](https://dart.dev/libraries/async/async-await) bakın.

## Stream'leri yönetmek

Bir Stream'den değer almanız gerektiğinde iki seçeneğiniz vardır:

- `async` ve bir *asenkron for döngüsü* (`await for`) kullanın.

- Stream API'sini kullanın; [dart:async belgelerinde](https://dart.dev/libraries/dart-async#stream) açıklandığı gibi.

> **Not**
>
> `await for` kullanmadan önce, kodu daha anlaşılır hale getirdiğinden ve gerçekten stream'in tüm sonuçlarını beklemek istediğinizden emin olun. Örneğin, UI çerçeveleri sonsuz olay akışları gönderdiği için UI olay dinleyicileri için genellikle **kullanmamalısınız**.

Bir asenkron for döngüsü şu biçimdedir:

```dart
await for (varOrType identifier in expression) {
  // Stream her değer yaydığında çalıştırılır.
}
```

``

`expression` değerinin türü Stream olmalıdır. Yürütme şu şekilde ilerler:

1.  Stream bir değer yayana kadar bekle.

2.  Değişken, yayılan bu değere ayarlanmış halde for döngüsünün gövdesini çalıştır.

3.  Stream kapanana kadar 1. ve 2. adımları tekrarla.

Stream'i dinlemeyi bırakmak için `break` veya `return` deyimini kullanabilirsiniz; bu deyimler for döngüsünden çıkar ve stream aboneliğini sonlandırır.

**Bir asenkron for döngüsü uygularken derleme zamanı hatası alırsanız, await for ifadesinin bir async fonksiyonunda olduğundan emin olun.** Örneğin, uygulamanızın `main()` fonksiyonunda bir asenkron for döngüsü kullanmak için `main()` gövdesinin `async` olarak işaretlenmesi gerekir:

```dart
void main() async {
  // ...
  await for (final request in requestServer) {
    handleRequest(request);
  }
  // ...
}
```

Dart'ın asenkron programlama desteği hakkında daha fazla bilgi için [dart:async](https://dart.dev/libraries/dart-async) kütüphane belgelerine göz atın.

# İsolates

Dart dilinde izole metinlerin yazılmasına dair bilgiler.

Bu sayfada, `Isolate` izole uygulamaları hayata geçirmek için API'nin kullanımına dair bazı örnekler ele alınmaktadır.

Uygulamanız, diğer hesaplamaları geçici olarak engelleyecek kadar büyük hesaplamalar yapıyorsa, izole kodlar kullanmalısınız. En yaygın örnek, Flutter uygulamalarında, aksi takdirde kullanıcı arayüzünün yanıt vermemesine neden olabilecek büyük hesaplamalar yapmanız gerektiğinde görülür.

> **Çırpınma notası**
>
> Flutter web, birden fazla izole sınıfı desteklemiyor. Ayrıca bakınız: Web'de eşzamanlılık

İzole genleri ne zaman kullanmanız gerektiğine dair kesin kurallar yok , ancak işte bunların faydalı olabileceği bazı durumlar:

- Son derece büyük JSON veri bloklarını ayrıştırma ve kod çözme.

- Fotoğraf, ses ve video dosyalarının işlenmesi ve sıkıştırılması.

- Ses ve video dosyalarını dönüştürme.

- Büyük listelerde veya dosya sistemlerinde karmaşık arama ve filtreleme işlemleri gerçekleştirme.

- Veritabanıyla iletişim kurmak gibi giriş/çıkış işlemleri gerçekleştirmek.

- Çok sayıda ağ isteğini işlemek.

## Basit Bir İşçi İsolate Uygulamak

Bu örnekler, basit bir çalışan izole işlemi başlatan ana bir izole işlemi uygular. `Isolate.run()` Çalışan izole işlemlerinin kurulumu ve yönetimiyle ilgili adımları basitleştirir:

- İzole bir nesne oluşturur (başlatır ve yaratır).

- Oluşturulan izole ortamda bir fonksiyon çalıştırır.

- Sonucu kaydeder.

- Sonucu ana izoleye döndürür.

- İşlem tamamlandığında izolasyonu sonlandırır.

- İstisnaları ve hataları kontrol eder, yakalar ve ana izole ortama geri gönderir.

> **Çırpınma notası**
>
> Eğer Flutter kullanıyorsanız, `Isolate.run()` yerine Flutter'ın `compute` fonksiyonunu kullanabilirsiniz.

### Yeni Bir İsolate Ortamda Mevcut Bir Yöntemi Çalıştırmak

Ana izole işlem içinde, sonuç beklenirken doğrudan yeni bir izole işlem ( arka planda çalışan bir işlem ) `run()` başlatmak için çağrı yapılır : `main()`

```dart
const String filename = 'with_keys.json';
void main() async {
  // Read some data.
  final jsonData = await Isolate.run(_readAndParseJson);
  // Use that data.
  print('Number of JSON keys: ${jsonData.length}');
}
```

Çalışana, çalıştırmasını istediğiniz fonksiyonu ilk argüman olarak iletin. Bu örnekte, mevcut fonksiyondur `_readAndParseJson()`:

```dart
Future<Map<String, dynamic>> _readAndParseJson() async {
  final fileData = await File(filename).readAsString();
  final jsonData = jsonDecode(fileData) as Map<String, dynamic>;
  return jsonData;
}
```

``

`Isolate.run()` Sonucu alır `_readAndParseJson()`, değeri ana izole cihaza geri gönderir ve işçi izole cihazını kapatır.

Çalışan izole hücre, sonucu tutan belleği ana izole hücreye aktarır . Verileri kopyalamaz . Çalışan izole hücre, nesnelerin aktarılmasına izin verilip verilmediğini doğrulamak için bir doğrulama işlemi gerçekleştirir.

`_readAndParseJson()` Mevcut, eşzamansız bir fonksiyondur ve ana izole ortamda doğrudan çalıştırılabilir. `Isolate.run()` Bunun yerine kullanarak çalıştırmak eşzamanlılığı sağlar. Çalışan izole ortam, hesaplamalarını tamamen soyutlar `_readAndParseJson()`. Ana izole ortamı engellemeden tamamlanabilir.

Sonuç `Isolate.run()` her zaman bir Future'dır, çünkü ana izole ortamdaki kod çalışmaya devam eder. Çalışan izole ortamın yürüttüğü hesaplamanın senkron mu yoksa asenkron mu olduğu ana izole ortamı etkilemez, çünkü her iki durumda da eş zamanlı olarak çalışır.

Programın tamamı için send_and_receive.dart örnek dosyasına göz atabilirsiniz.

### İsolatelerle Birlikte Kapatma İşlemleri Gönderiliyor.

`run()` Ana izole ortamda doğrudan bir fonksiyon değişmezi veya kapatma kullanarak da basit bir çalışan izole ortamı oluşturabilirsiniz .

```dart
const String filename = 'with_keys.json';
void main() async {
  // Read some data.
  final jsonData = await Isolate.run(() async {
      final fileData = await File(filename).readAsString();
      final jsonData = jsonDecode(fileData) as Map<String, dynamic>;
      return jsonData;
  });
  // Use that data.
  print('Number of JSON keys: ${jsonData.length}');
}
```

Bu örnek, bir öncekiyle aynı işlevi görüyor. Yeni bir izole işlem başlatılıyor, bir şeyler hesaplanıyor ve sonuç geri gönderiliyor.

Ancak şimdi izole edilmiş nesne bir closure gönderiyor . Closure'lar, hem işleyiş biçimleri hem de koda yazılma şekilleri bakımından tipik adlandırılmış fonksiyonlardan daha az sınırlıdır. Bu örnekte, yerel kod gibi görünen bir şeyi eş zamanlı olarak yürütüyor. Bu anlamda, "paralel çalıştır" için bir kontrol akışı operatörü gibi çalıştığını `Isolate.run()` düşünebilirsiniz `.run()`

## Portlar Aracılığıyla İsolate Cihazlar Arasında Birden Fazla Mesaj Gönderme

Kısa ömürlü izole ortamlar kullanımı kolaydır, ancak yeni izole ortamlar oluşturmak ve nesneleri bir izole ortamdan diğerine kopyalamak performans açısından ek yük getirir. Kodunuz aynı hesaplamayı tekrar tekrar çalıştırmaya dayanıyorsa `Isolate.run`, bunun yerine hemen çıkmayan uzun ömürlü izole ortamlar oluşturarak performansı artırabilirsiniz.

Bunu yapmak için, soyutlama sağlayan bazı düşük seviyeli izole API'lerden yararlanabilirsiniz `Isolate.run`:

- `Isolate.spawn()` Ve `Isolate.exit()`

- `ReceivePort` Ve `SendPort`

- `SendPort.send()` yöntem

Bu bölümde, yeni oluşturulan bir izole sistem ile ana izole sistem arasında çift yönlü iletişimi kurmak için gereken adımlar ele alınmaktadır . İlk örnek olan "Temel portlar" , süreci genel bir düzeyde tanıtmaktadır. İkinci örnek olan "Sağlam portlar" ise, ilk örneğe kademeli olarak daha pratik, gerçek dünya işlevselliği eklemektedir.

### `ReceivePort` Ve `SendPort`

İzole cihazlar arasında uzun süreli iletişim kurmak için (ek olarak `Isolate`) iki sınıfa daha ihtiyaç vardır: `ReceivePort` ve `SendPort`. Bu portlar, izole cihazların birbirleriyle iletişim kurmasının tek yoludur.

A `ReceivePort`, diğer izole sistemlerden gönderilen mesajları işleyen bir nesnedir. Bu mesajlar bir aracılığıyla gönderilir `SendPort`.

> **Not**
>
> Bir `SendPort` nesne yalnızca bir ile ilişkilidir `ReceivePort`, ancak tek bir nesnenin `ReceivePort` birden fazla nesnesi olabilir `SendPorts`. Bir nesne oluşturduğunuzda `ReceivePort`, kendisi için bir nesne daha oluşturur . Mevcut bir nesneye mesaj gönderebilen `SendPort` ek nesneler oluşturabilirsiniz .

Portlar Stream nesnelere benzer şekilde davranır (aslında, alıcı portlar da uygular !). Bir ve portunu sırasıyla Stream'in ve dinleyicileri gibi `Stream` düşünebilirsiniz . Bir port, bir porta benzer çünkü bunlara `request` yöntemiyle mesajlar "eklersiniz" ve bu mesajlar bir dinleyici tarafından, bu durumda `request` tarafından, işlenir . `request` daha sonra aldığı mesajları, sağladığınız bir geri çağırma işlevine argüman olarak geçirerek işler.

### Bağlantı Noktalarının Ayarlanması

Yeni oluşturulan bir izole hücre, yalnızca çağrı yoluyla aldığı bilgilere sahiptir `Isolate.spawn`. Ana izole hücrenin, ilk oluşturulmasından sonra da yeni oluşturulan izole hücreyle iletişim kurmaya devam etmesi gerekiyorsa, yeni oluşturulan izole hücrenin ana izole hücreye mesaj gönderebileceği bir iletişim kanalı kurmalısınız. İzole hücreler yalnızca mesaj yoluyla iletişim kurabilirler. Birbirlerinin hafızasının içine "bakamazlar", bu da "izolat" adının kökenini oluşturur.

Bu çift yönlü iletişimi kurmak için öncelikle `ReceivePort` ana izole ortamda bir nesnesi oluşturun, ardından `SendPort` bu nesneyi yeni izole ortamı oluştururken argüman olarak yeni izole ortama iletin `Isolate.spawn`. Yeni izole ortam daha sonra kendi nesnesini oluşturur `ReceivePort` ve ana izole ortam tarafından kendisine iletilen nesnesi üzerinden geri gönderir . Ana izole ortam bu nesneyi alır ve artık her iki taraf da mesaj gönderip almak için açık bir kanala sahip olur. `SendPort`

> **Not**
>
> Bu bölümdeki diyagramlar genel bir bakış açısı sunmakta ve izole cihazlar için portların kullanım konseptini aktarmayı amaçlamaktadır. Gerçek uygulama biraz daha fazla kod gerektirir ve bu kodu sayfanın ilerleyen bölümlerinde bulacaksınız .

1.  ![](2026-guz-4takim-gorseller/image8.png)

2.  ``

3.  `ReceivePort` Ana izole ortamda bir tane oluşturun . Bu, `SendPort` üzerinde bir özellik olarak otomatik olarak oluşturulur `ReceivePort`.

1.  İşçi izolesini şu şekilde oluşturun: `Isolate.spawn()`

2.  `ReceivePort.sendPort` Çalışan izole cihaza ilk mesaj olarak bir referans iletin .

3.  `ReceivePort` İşçi izolasyonunda yeni bir tane daha oluşturun .

4.  Ana izole cihaza *geri gönderilen* `ReceivePort.sendPort` ilk mesajda, çalışan izole cihazın referansını iletin .

Bağlantı noktalarını oluşturmanın ve iletişimi kurmanın yanı sıra, bağlantı noktalarına mesaj aldıklarında ne yapacaklarını da belirtmeniz gerekecek. Bu, `listen` her birine ait yöntem kullanılarak yapılır `ReceivePort`.

![Olay döngüsüne tek tek beslenen olayları gösteren bir şekil.](2026-guz-4takim-gorseller/image9.png)

1.  Ana izole cihazın referansı üzerinden çalışan izole cihazına bir mesaj gönderin `SendPort`.

1.  Çalışan izole sunucudaki bir dinleyici aracılığıyla mesajı alın ve işleyin `ReceivePort`. Ana izole sunucudan taşımak istediğiniz hesaplama burada yürütülür.

2.  Çalışan izole ünitesinin ana izole ünitesine olan referansı aracılığıyla bir yanıt mesajı gönderin `SendPort`.

3.  Mesajı ana izole cihazın üzerindeki bir dinleyici aracılığıyla alın `ReceivePort`.

### Temel Bağlantı Noktaları Örneği

Bu örnek, ana izole ile arasında çift yönlü iletişim bulunan uzun ömürlü bir çalışan izole nasıl kurulabileceğini göstermektedir. Kod, JSON metninin yeni bir izoleye gönderilmesi örneğini kullanmaktadır; burada JSON ayrıştırılıp çözümlendikten sonra ana izoleye geri gönderilecektir.

> **Uyarı**
>
> Bu örnek , zaman içinde birden fazla mesaj gönderip alabilen yeni bir izole hücre oluşturmak için gereken en temel bilgileri öğretmeyi amaçlamaktadır .
>
> Üretim yazılımlarında beklenen hata yönetimi, port kapatma ve mesaj sıralaması gibi önemli işlevleri kapsamamaktadır.
>
> Sonraki bölümde yer alan Sağlam bağlantı noktaları örneği, bu işlevselliği kapsamakta ve bu işlevsellik olmadan ortaya çıkabilecek bazı sorunları ele almaktadır.

#### Adım 1: Çalışan Sınıfını Tanımlayın

Öncelikle, arka plan çalışanı izole işleminiz için bir sınıf oluşturun. Bu sınıf, aşağıdaki işlevleri yerine getirmek için ihtiyacınız olan tüm fonksiyonları içerir:

- İzole bir varlık oluşturun.

- O izole cihaza mesaj gönder.

- İzole cihazın bazı JSON verilerini çözmesini sağlayın.

- Çözümlenmiş JSON dosyasını ana izole cihaza geri gönderin.

Bu sınıf iki genel yöntem sunar: biri çalışan izole sunucuyu başlatır, diğeri ise bu çalışan izole sunucuya mesaj göndermeyi yönetir.

Bu örneğin geri kalan bölümleri, sınıf yöntemlerini tek tek nasıl dolduracağınızı gösterecektir.

```dart
class Worker {
  Future<void> spawn() async {
    // TODO: Add functionality to spawn a worker isolate.
  }
  void _handleResponsesFromIsolate(dynamic message) {
    // TODO: Handle messages sent back from the worker isolate.
  }
  static void _startRemoteIsolate(SendPort port) {
    // TODO: Define code that should be executed on the worker isolate.
  }
  Future<void> parseJson(String message) async {
    // TODO: Define a public method that can
    // be used to send messages to the worker isolate.
  }
}
```

#### Adım 2: Bir İşçi İsolate Hücresi Oluşturun

Bu `Worker.spawn` yöntem, çalışan izole işlemini oluşturmak ve mesaj alıp gönderebilmesini sağlamak için gereken kodu gruplandıracağınız yerdir.

- Öncelikle bir tane oluşturun `ReceivePort`. Bu, ana izole cihazın yeni oluşturulan işçi izole cihazından gönderilen mesajları almasını sağlar.

- Ardından, çalışan izole cihazın geri göndereceği mesajları işlemek için alma portuna bir dinleyici ekleyin. Dinleyiciye iletilen geri çağırma işlevi, 4. `_handleResponsesFromIsolate` adımda ele alınacaktır .

- Son olarak, \`worker isolate\`'ı \`spatch\` komutuyla başlatın `Isolate.spawn`. Bu komut iki argüman bekler: worker isolate üzerinde yürütülecek bir fonksiyon ( 3. adımda ele alınmıştır ) ve `sendPort` alma portunun özelliği.

```dart
Future<void> spawn() async {
  final receivePort = ReceivePort();
  receivePort.listen(_handleResponsesFromIsolate);
  await Isolate.spawn(_startRemoteIsolate, receivePort.sendPort);
}
```

Bu `receivePort.sendPort` argüman, işçi izolesinde çağrıldığında geri çağrı fonksiyonuna ( ) bir argüman olarak iletilecektir `_startRemoteIsolate`. Bu, işçi izolesinin ana izoleye mesaj gönderebilmesini sağlamanın ilk adımıdır.

#### Adım 3: Çalışan İsolate Sunucuda Kodu Çalıştırın.

`_startRemoteIsolate` Bu adımda, çalışan izole işlem başlatıldığında yürütülecek olan yöntemi tanımlarsınız . Bu yöntem, çalışan izole işlem için "ana" yöntem gibidir.

- Öncelikle, yeni bir port daha oluşturun `ReceivePort`. Bu port, ana izole cihazdan gelen gelecekteki mesajları alacaktır.

- Ardından, o portun verilerini `SendPort` ana izole cihaza geri gönderin.

- Son olarak, yeni yapıya bir dinleyici ekleyin `ReceivePort`. Bu dinleyici, ana izole cihazın çalışan izole cihaza gönderdiği mesajları işler.

```dart
static void _startRemoteIsolate(SendPort port) {
  final receivePort = ReceivePort();
  port.send(receivePort.sendPort);
  receivePort.listen((dynamic message) async {
      if (message is String) {
        final transformed = jsonDecode(message);
        port.send(transformed);
      }
  });
}
```

Çalışan cihazdaki dinleyici, `ReceivePort` ana izole cihazdan gelen JSON verisini çözümler ve ardından çözümlenmiş JSON verisini ana izole cihaza geri gönderir.

Bu dinleyici, ana izole sistemden çalışan izole sisteme gönderilen mesajlar için giriş noktasıdır. Çalışan izole sisteme gelecekte hangi kodu çalıştıracağını söyleyebileceğiniz tek fırsat budur.

#### Adım 4: Ana İsolate Cihazda Mesajları İşleyin

Son olarak, ana izole cihazın, işçi izole cihazından ana izole cihaza gönderilen mesajları nasıl işleyeceğini belirtmeniz gerekiyor. Bunu yapmak için, ilgili metodu doldurmanız gerekir `_handleResponsesFromIsolate`. Hatırlayın ki, bu metot 2. adımda açıklandığı gibi `receivePort.listen` metoduna iletilir :

```dart
Future<void> spawn() async {
  final receivePort = ReceivePort();
  receivePort.listen(_handleResponsesFromIsolate);
  await Isolate.spawn(_startRemoteIsolate, receivePort.sendPort);
}
```

Ayrıca 3. adımda `SendPort` ana izoleye bir mesaj gönderdiğinizi de hatırlayın. Bu yöntem, bu mesajın alınmasının yanı sıra gelecekteki mesajları (çözümlenmiş JSON olacak) da işler.

- Öncelikle, mesajın bir port olup olmadığını kontrol edin `SendPort`. Eğer öyleyse, daha sonra mesaj göndermek için kullanılabilmesi için bu portu sınıfın `_sendPort` özelliğine atayın.

- Ardından, mesajın `Map<String, dynamic>` beklenen çözümlenmiş JSON türü olan türünde olup olmadığını kontrol edin. Eğer öyleyse, bu mesajı uygulamanıza özgü mantıkla işleyin. Bu örnekte, mesaj yazdırılıyor.

```dart
void _handleResponsesFromIsolate(dynamic message) {
  if (message is SendPort) {
    _sendPort = message;
    _isolateReady.complete();
  } else if (message is Map<String, dynamic>) {
    print(message);
  }
}
```

#### Adım 5: İsolate Kurulduğundan Emin Olmak İçin Bir Tamamlayıcı Ekleyin.

Sınıfı tamamlamak için, `parseJson` çalışan izole sunucuya mesaj göndermekten sorumlu olan adlı bir public metot tanımlayın. Ayrıca, izole sunucu tamamen kurulmadan önce mesajların gönderilebilmesini de sağlaması gerekir. Bunu ele almak için bir kullanın `Completer`.

- Öncelikle, sınıf düzeyinde "a" adında bir özellik ekleyin `Completer` ve ona "a" adını verin `_isolateReady`.

- Ardından, mesaj bir `SendPort` ise, ( 4. adımda oluşturulan) `_handleResponsesFromIsolate` yöntemde tamamlayıcıya bir `complete()` çağrısı ekleyin .

- Son olarak, yöntemde `parseJson`, `_sendPort.send` eklemeden önce `await _isolateReady.future` ekleyin. Bu, işçi izole hücresi oluşturulup ana izole hücreye `SendPort` geri gönderilmeden önce hiçbir mesajın işçi izole hücresine gönderilememesini sağlar .

```dart
Future<void> parseJson(String message) async {
  await _isolateReady.future;
  _sendPort.send(message);
}
```

#### Tam Örnek

### Sağlam Bağlantı Noktaları Örneği

Önceki örnek, çift yönlü iletişime sahip uzun ömürlü bir izole ortam kurmak için gereken temel yapı taşlarını açıklamıştı. Bahsedildiği gibi, bu örnekte hata yönetimi, artık kullanılmadığında portları kapatma yeteneği ve bazı durumlarda mesaj sıralamasındaki tutarsızlıklar gibi bazı önemli özellikler eksiktir.

Bu örnek, ilk örnekteki bilgileri genişleterek, bu ek özelliklere ve daha fazlasına sahip, daha iyi tasarım kalıplarını izleyen uzun ömürlü bir çalışan izole süreci oluşturur. Bu kod ilk örneğe benzerlik gösterse de, o örneğin bir uzantısı değildir.

> **Not**
>
> Bu örnek, önceki örnekte `Isolate.spawn` ele alınan, izole cihazlar arasında ve portlar aracılığıyla iletişim kurma konusuna zaten aşina olduğunuzu varsaymaktadır .

#### Adım 1: Çalışan Sınıfını Tanımlayın

Öncelikle, arka plan çalışanı izole işleminiz için bir sınıf oluşturun. Bu sınıf, aşağıdaki işlevleri yerine getirmek için ihtiyacınız olan tüm fonksiyonları içerir:

- İzole bir varlık oluşturun.

- O izole cihaza mesaj gönder.

- İzole cihazın bazı JSON verilerini çözmesini sağlayın.

- Çözümlenmiş JSON dosyasını ana izole cihaza geri gönderin.

Bu sınıf üç genel yöntem sunar: biri çalışan izole sunucuyu oluşturur, diğeri bu çalışan izole sunucuya mesaj göndermeyi yönetir ve üçüncüsü de artık kullanılmadığında portları kapatabilir.

```dart
class Worker {
  final SendPort _commands;
  final ReceivePort _responses;
  Future<Object?> parseJson(String message) async {
    // TODO: Ensure the port is still open.
    _commands.send(message);
  }
  static Future<Worker> spawn() async {
    // TODO: Add functionality to create a new Worker object with a
    // connection to a spawned isolate.
    throw UnimplementedError();
  }
  Worker._(this._responses, this._commands) {
    // TODO: Initialize main isolate receive port listener.
  }
  void _handleResponsesFromIsolate(dynamic message) {
    // TODO: Handle messages sent back from the worker isolate.
  }
  static void _handleCommandsToIsolate(ReceivePort rp, SendPort sp) async {
    // TODO: Handle messages sent back from the worker isolate.
  }
  static void _startRemoteIsolate(SendPort sp) {
    // TODO: Initialize worker isolate's ports.
  }
}
```

> **Not**
>
> Bu örnekte, `SendPort` örnekler `ReceivePort` en iyi uygulama adlandırma kuralına uyarak ana izoleye göre adlandırılır. `SendPort` Ana izoleden çalışan izoleye gönderilen mesajlara komut , ana izoleye geri gönderilen mesajlara ise yanıt denir .

#### Adım 2: Yöntem `Worker.spawn` İçinde Bir `RawReceivePort` Oluşturun

`RawReceivePort` Bir izole ortam oluşturmadan önce, daha alt seviye bir yapı olan bir \`\<iostream\>\` oluşturmanız gerekir `ReceivePort`. \`\<iostream\>\` kullanmak `RawReceivePort` tercih edilen bir yöntemdir çünkü izole ortamın başlatma mantığını, izole ortamda mesaj iletimini yöneten mantıktan ayırmanıza olanak tanır.

Yöntemde `Worker.spawn`:

- Öncelikle, `RawReceivePort`. Bu, `ReceivePort` yalnızca çalışan izole sunucusundan gelen ilk mesajı almakla sorumludur; bu mesaj bir . olacaktır `SendPort`.

- Ardından, izole cihazın mesaj almaya hazır olduğunu gösteren bir `Completer` kaydı oluşturun . Bu işlem tamamlandığında, bir `ReceivePort` ve bir `SendPort` içeren bir kayıt döndürecektir .

- Ardından, özelliği tanımlayın `RawReceivePort.handler`. Bu özellik, `Function?` şu şekilde davranan bir özelliktir `ReceivePort.listener`: . Bu porttan bir mesaj alındığında fonksiyon çağrılır.

- İşleyici fonksiyonu içinde, `connection.complete()` yöntemini çağırın. Bu yöntem, argüman olarak bir `ReceivePort` ve bir `SendPort` içeren bir kayıt bekler. `SendPort`, bir sonraki adımda `_commands` adlı sınıf düzeyine atanacak olan, çalışan izole işleminden gönderilen ilk mesajdır .

- `ReceivePort` Ardından, yapıcıyı kullanarak yeni bir nesne oluşturun `ReceivePort.fromRawReceivePort` ve ona `initPort` değerini iletin.

```dart
class Worker {
  final SendPort _commands;
  final ReceivePort _responses;
  static Future<Worker> spawn() async {
    // Create a receive port and add its initial message handler.
    final initPort = RawReceivePort();
    final connection = Completer<(ReceivePort, SendPort)>.sync();
    initPort.handler = (initialMessage) {
      final commandPort = initialMessage as SendPort;
      connection.complete((
          ReceivePort.fromRawReceivePort(initPort),
          commandPort,
      ));
    };
  }
}
```

``

`RawReceivePort` Önce bir tane, sonra da bir tane oluşturarak `ReceivePort`, daha sonra yeni bir geri çağırma işlevi ekleyebileceksiniz `ReceivePort.listen`. Tersine, doğrudan bir tane oluşturursanız `ReceivePort`, yalnızca bir tane ekleyebilirsiniz `listener`, çünkü yerine `ReceivePort` arayüzünü uygular . `Stream` `BroadcastStream`

Esasen bu, izole başlatma mantığınızı, iletişim kurulumu tamamlandıktan sonra mesaj alma mantığını ayırmanıza olanak tanır. Diğer yöntemlerdeki mantık geliştikçe bu avantaj daha belirgin hale gelecektir.

#### Adım 3: Aşağıdaki Komutla Bir İşçi İzole Hücresi Oluşturun: `Isolate.spawn`

Bu adımda metodu doldurmaya devam edersiniz `Worker.spawn`. Bir izole ortam oluşturmak için gereken kodu ekleyecek ve `Worker` bu sınıftan bir örnek döndüreceksiniz. Bu örnekte, çağrısı bir `try` / `catch` bloğu `Isolate.spawn` içine alınmıştır ; bu, izole ortamın başlatılamaması durumunda `initPort` kapatılacağını ve `Worker` nesnenin oluşturulmayacağını garanti eder.

- Öncelikle, bir `try` / `catch` bloğunda bir işçi izole hücresi oluşturmayı deneyin. İşçi izole hücresi oluşturma başarısız olursa, önceki adımda oluşturulan alma portunu kapatın. iletilecek yöntem `Isolate.spawn` daha sonraki bir adımda ele alınacaktır.

- Ardından, yanıtı bekleyin `connection.future` ve döndürdüğü kayıttan gönderme portunu ve alma portunu ayrıştırın.

- Son olarak, `Worker` özel kurucusunu çağırarak ve bu tamamlayıcıdan gelen portları parametre olarak geçirerek bir örnek döndürün.

```dart
class Worker {
  final SendPort _commands;
  final ReceivePort _responses;
  static Future<Worker> spawn() async {
    // Create a receive port and add its initial message handler.
    final initPort = RawReceivePort();
    final connection = Completer<(ReceivePort, SendPort)>.sync();
    initPort.handler = (initialMessage) {
      final commandPort = initialMessage as SendPort;
      connection.complete((
          ReceivePort.fromRawReceivePort(initPort),
          commandPort,
      ));
    };
    // Spawn the isolate.
    try {
      await Isolate.spawn(_startRemoteIsolate, (initPort.sendPort));
    } on Object {
      initPort.close();
      rethrow;
    }
    final (ReceivePort receivePort, SendPort sendPort) =
    await connection.future;
    return Worker._(receivePort, sendPort);
  }
}
```

Bu örnekte ( önceki örneğe kıyasla ), `Worker.spawn` bu sınıf için eşzamansız statik bir kurucu görevi gördüğünü ve bir örneğini oluşturmanın tek yolu olduğunu unutmayın `Worker`. Bu, API'yi basitleştirerek bir örneğini oluşturan kodu `Worker` daha temiz hale getirir.

#### Adım 4: İsolate Kurulum İşlemini Tamamlayın

Bu adımda, temel izole kurulum sürecini tamamlayacaksınız. Bu, önceki örnekle neredeyse tamamen örtüşüyor ve yeni bir kavram yok. Kodun daha fazla metoda bölünmesiyle küçük bir değişiklik var; bu, bu örneğin geri kalanında daha fazla işlevsellik eklemenizi kolaylaştıran bir tasarım uygulamasıdır. İzole kurulumunun temel sürecinin ayrıntılı bir açıklaması için temel portlar örneğine bakın .

Öncelikle, `Worker.spawn` metottan döndürülen özel kurucu metodu oluşturun. Kurucu metodun gövdesinde, ana izole ortam tarafından kullanılan alma portuna bir dinleyici ekleyin ve bu dinleyiciye henüz tanımlanmamış bir metot iletin `_handleResponsesFromIsolate`.

```dart
class Worker {
  final SendPort _commands;
  final ReceivePort _responses;
  Worker._(this._responses, this._commands) {
    _responses.listen(_handleResponsesFromIsolate);
  }
}
```

``

`_startRemoteIsolate` Ardından, işçi izolesindeki portları başlatmaktan sorumlu kodu ekleyin. Bu yöntemin, `Isolate.spawn` yönteminde geçirildiğini ve `Worker.spawn` ana izolenin `SendPort` argümanı olarak kendisine iletileceğini hatırlayın.

1.  Yeni bir tane oluşturun `ReceivePort`.

1.  O portları `SendPort` ana izole cihaza geri gönderin.

2.  Yeni bir metot çağırın `_handleCommandsToIsolate` ve hem yeni metodu `ReceivePort` hem de `SendPort` ana izole ortamdan gelen metodu argüman olarak geçirin.

```dart
static void _startRemoteIsolate(SendPort sendPort) {
  final receivePort = ReceivePort();
  sendPort.send(receivePort.sendPort);
  _handleCommandsToIsolate(receivePort, sendPort);
}
```

``

`_handleCommandsToIsolate` Ardından, ana izole sunucudan gelen mesajları almaktan, çalışan izole sunucuda JSON'u çözmekten ve çözülmüş JSON'u yanıt olarak geri göndermekten sorumlu olan yöntemi ekleyin .

1.  Öncelikle, işçi izole cihazında bir dinleyici tanımlayın `ReceivePort`.

1.  Dinleyiciye eklenen geri çağırma işlevi içinde, ana izole cihazdan gelen JSON verisini bir `try` / `catch` bloğu içinde çözmeye çalışın. Çözme işlemi başarılı olursa, çözülmüş JSON verisini ana izole cihaza geri gönderin.

2.  Bir hata oluşursa, bir `RemoteError` ile geri dönün.

```dart
static void _handleCommandsToIsolate(
  ReceivePort receivePort,
  SendPort sendPort,
) {
  receivePort.listen((message) {
      try {
        final jsonData = jsonDecode(message as String);
        sendPort.send(jsonData);
      } catch (e) {
        sendPort.send(RemoteError(e.toString(), ''));
      }
  });
}
```

Ardından, metodun kodunu ekleyin `_handleResponsesFromIsolate`.

1.  Öncelikle, mesajın bir hata mesajı olup olmadığını kontrol edin `RemoteError`; eğer öyleyse, `throw` o hatayı düzeltmelisiniz.

1.  Aksi takdirde, mesajı yazdırın. İlerleyen adımlarda, bu kodu mesajları yazdırmak yerine döndürecek şekilde güncelleyeceksiniz.

```dart
void _handleResponsesFromIsolate(dynamic message) {
  if (message is RemoteError) {
    throw message;
  } else {
    print(message);
  }
}
```

``

`parseJson` Son olarak, dışarıdan gelen kodun JSON verilerini çözümlenmek üzere çalışan izole sunucuya göndermesine olanak tanıyan, herkese açık bir yöntem olan metodu ekleyin .

```dart
Future<Object?> parseJson(String message) async {
  _commands.send(message);
}
```

Bu yöntemi bir sonraki adımda güncelleyeceksiniz.

#### Adım 5: Aynı Anda Birden Fazla Mesajı İşleme

Şu anda, eğer çalışan izole sunucuya hızlı bir şekilde mesaj gönderirseniz, izole sunucu çözümlenmiş JSON yanıtını gönderilme sırasına göre değil, tamamlanma sırasına göre gönderir. Hangi yanıtın hangi mesaja karşılık geldiğini belirlemenin bir yolu yoktur.

Bu adımda, her mesaja bir kimlik (id) vererek ve dış kod `parseJson` çağırdığında çağırana döndürülen yanıtın doğru yanıt olmasını sağlamak için `Completer` nesneler kullanarak bu sorunu çözeceksiniz .

Öncelikle, sınıf düzeyinde iki özellik ekleyin `Worker`:

- `Map<int, Completer<Object?>> _activeRequests`

- `int _idCounter`

```dart
class Worker {
  final SendPort _commands;
  final ReceivePort _responses;
  final Map<int, Completer<Object?>> _activeRequests = {};
  int _idCounter = 0;
  // ···
}
```

Bu `_activeRequests` harita, çalışan izole cihazına gönderilen bir mesajı bir `Completer` ile ilişkilendirir. Kullanılan anahtarlar , daha fazla mesaj gönderildikçe artırılacak olan `_idCounter` dosyasından alınır.

`parseJson` Ardından, mesajları çalışan izoleye göndermeden önce tamamlayıcılar oluşturacak şekilde yöntemi güncelleyin .

1.  `Completer` Öncelikle bir . oluşturun .

1.  Ardından, her `Completer` birine benzersiz bir numara atanacak `_idCounter` şekilde artırın.

2.  `_activeRequests` Haritaya, anahtarı mevcut sayı `_idCounter`, tamamlayıcısı ise değer olan bir giriş ekleyin .

3.  Mesajı, kimlik numarasıyla birlikte izole çalışana gönderin. Sadece tek bir değer gönderebileceğiniz için `SendPort`, kimlik numarasını ve mesajı bir kayıt içine alın .

4.  Son olarak, tamamlayıcının geleceğini döndürün; bu gelecek nihayetinde işçi izole edicisinden gelen yanıtı içerecektir.

```dart
Future<Object?> parseJson(String message) async {
  final completer = Completer<Object?>.sync();
  final id = _idCounter++;
  _activeRequests[id] = completer;
  _commands.send((id, message));
  return await completer.future;
}
```

``

`_handleResponsesFromIsolate` Ayrıca bu sistemi güncellemeniz ve `_handleCommandsToIsolate` yönetmeniz gerekiyor .

Burada `_handleCommandsToIsolate`, yalnızca JSON metni değil, iki değere sahip bir kayıt olduğunu hesaba katmanız gerekir `message`. Bunu, değerleri ayrıştırarak yapabilirsiniz `message`.

Ardından, JSON'u çözdükten sonra, `sendPort.send` hem kimliği hem de çözülmüş JSON'u bir kayıt kullanarak ana izoleye geri göndermek için çağrıyı güncelleyin.

```dart
static void _handleCommandsToIsolate(
  ReceivePort receivePort,
  SendPort sendPort,
) {
  receivePort.listen((message) {
      final (int id, String jsonText) = message as (int, String); // New
      try {
        final jsonData = jsonDecode(jsonText);
        sendPort.send((id, jsonData)); // Updated
      } catch (e) {
        sendPort.send((id, RemoteError(e.toString(), '')));
      }
  });
}
```

Son olarak, `_handleResponsesFromIsolate` dosyasını güncelleyin.

1.  Öncelikle, mesaj argümanından gelen kimliği ve yanıtı tekrar ayrıştırın.

1.  Ardından, bu isteğe karşılık gelen tamamlayıcıyı haritadan kaldırın `_activeRequests`.

2.  `parseJson` Son olarak, hata fırlatmak veya çözümlenmiş JSON'u yazdırmak yerine, yanıtı ileterek tamamlayıcıyı tamamlayın. Bu işlem tamamlandığında, yanıt ana izole üzerinde çağrı yapan koda geri döndürülecektir .

```dart
void _handleResponsesFromIsolate(dynamic message) {
  final (int id, Object? response) = message as (int, Object?); // New
  final completer = _activeRequests.remove(id)!; // New
  if (response is RemoteError) {
    completer.completeError(response); // Updated
  } else {
    completer.complete(response); // Updated
  }
}
```

#### Adım 6: Bağlantı Noktalarını Kapatma İşlevini Ekleyin

Kodunuz artık izole ortamı kullanmadığında, ana izole ortam ve çalışan izole ortamdaki portları kapatmalısınız.

1.  Öncelikle, portların kapalı olup olmadığını takip eden sınıf düzeyinde bir boolean değişkeni ekleyin.

1.  Ardından `Worker.close` metodu ekleyin. Bu metodun içinde:

    - Güncelleme `_closed` doğru olacak.

    - Çalışan izole cihaza son bir mesaj gönderin. Bu mesaj `String` "kapatma" yazan bir nesne olacak, ancak istediğiniz herhangi bir nesne de olabilir. Bunu bir sonraki kod parçacığında kullanacaksınız.

    - Son olarak, `_activeRequests` boş olup olmadığını kontrol edin. Eğer boşsa, ana izolenin `ReceivePort` adlandırılmış olanını kapatın `_responses`.

```dart
class Worker {
  bool _closed = false;
  // ···
  void close() {
    if (!_closed) {
      _closed = true;
      _commands.send('shutdown');
      if (_activeRequests.isEmpty) _responses.close();
      print('--- port closed --- ');
    }
  }
}
```

Ardından, çalışan izole işleminde "kapatma" mesajını ele almanız gerekiyor. Metoda aşağıdaki kodu ekleyin `_handleCommandsToIsolate`. Bu kod, mesajın `String` "kapatma" olup olmadığını kontrol edecektir. Eğer öyleyse, çalışan izole işlemini kapatacak `ReceivePort` ve geri dönecektir.

```dart
static void _handleCommandsToIsolate(
  ReceivePort receivePort,
  SendPort sendPort,
) {
  receivePort.listen((message) {
      // New if-block.
      if (message == 'shutdown') {
        receivePort.close();
        return;
      }
      final (int id, String jsonText) = message as (int, String);
      try {
        final jsonData = jsonDecode(jsonText);
        sendPort.send((id, jsonData));
      } catch (e) {
        sendPort.send((id, RemoteError(e.toString(), '')));
      }
  });
}
```

Son olarak, mesaj göndermeden önce portların kapalı olup olmadığını kontrol edecek bir kod eklemelisiniz. Metodun içine tek bir satır ekleyin `Worker.parseJson`.

```dart
Future<Object?> parseJson(String message) async {
  if (_closed) throw StateError('Closed'); // New
  final completer = Completer<Object?>.sync();
  final id = _idCounter++;
  _activeRequests[id] = completer;
  _commands.send((id, message));
  return await completer.future;
}
```
