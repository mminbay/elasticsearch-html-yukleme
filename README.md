# elasticsearch-html-yukleme
Bu dokümanda HTML dosyalarının metin olarak ElasticSearch'e (ES) yüklenmesinin üzerinden geçeceğiz.

## Adımlar
1. ES'e herhangi bir dosya yüklemek ES'e yollayacağımız isteğe dosyanın _base64_'e çevrilmiş halini koymamız gerekiyor. ES'in _base64_ ile yazılmış bir istek parametresini okuyabilmesi için de `ingest-attachment` eklentisini kullanmammız gerek. Bunun için `bin` dizininde aşağıdaki komutu çalıştırıyoruz:
```
elasticsearch-plugin install ingest-attachment
```
![1](https://user-images.githubusercontent.com/76704001/177496471-0dae0d95-f3d5-458f-a9af-2b9e82f3df7d.png)

2. Eklenti yüklendikten sonra kullanılabilmesi için ES'i yeniden başlatıyoruz.
3. ES yeniden başladıktan sonra `ingest-attachment` eklentisini kullanacak bir *pipeline* oluşturmamız gerekiyor. Bunun için Kibana'nın _Dev Tools_ terminalinden ES'in `_ingest` API'sine bir istek yolluyoruz.

![dev tools](https://user-images.githubusercontent.com/76704001/177499814-471f525b-895f-4b1d-b503-28e6b7a06412.png)
![api request](https://user-images.githubusercontent.com/76704001/177499839-2e020992-725d-4d71-af07-6c2036d03fd6.png)

Burada `_ingest/pipeline/`'dan sonra gelen `dosya-html` bizim bu pipeline'a verdiğimiz ismi belirtiyor. Bunu istediğiniz gibi değiştirebilirsiniz. Aynı şekilde yolladığımız isteğin `description` parametresini da pipeline'ın içeriğini açıklayacak şekilde istediğiniz gibi değiştirebilirsiniz. 

Pipeline'ın `processors` listesinde ilk olarak birinci adımda eklenti olarak yüklediğimiz `attachment` prosesörünü ve parametrelerini tanımlıyoruz. 
- `field` parametresi bu pipeline'a tabi tutulacak isteğin hangi parametresinin base64 halindeki dosyayı içerdiğini belirtiyor. Bizim örneğimizde isteğin `blob` parametresinin okunması gerektiğini belirtiyoruz.
- `indexed-chars` parametresi `field`dan en fazla kaç tane karakter okunacağının sınırını belirtiyor. Burada bir sınır olmasını istemediğimiz için `-1` olarak belirtiyoruz.
- `remove-binary` parametresi `field`'daki base64 işlenip okunabilir metine çevrildikten sonra oluşturulacak dokümanda base64 halinin saklanıp saklanmayacağını belirtiyor. Bize dosyanın sadece okunabilir hali gerektiği için base64 halini saklamamayı seçiyoruz.

`attachment`'ın bütün parametrelerini [burada](https://www.elastic.co/guide/en/elasticsearch/plugins/current/using-ingest-attachment.html) bulabilirsiniz.

`attachment` prosesörü tanımladığımız alandaki base64'ü okuduktan sonra indekslenecek dokümanda `attachment` diye bir alan yaratıyor ve bu alanı okuduğu içerik ile dolduruyor.

Bundan sonra `html-strip` prosesörünü tanımlıyoruz. Yüklediğimiz html dosyasındaki html taglerinin yapacağımız aramalara tabi olmasını istemediğimiz için onları silmemiz gerekiyor.
- `field` parametresi dokümanın hangi parametresinin işleneceğini belirtiyor. Bir önceki prosesörün okuyup işlediği alanı hedef almak için `attachment.content` olarak belirtiyoruz.

`html-strip` metinden kaldırdığı html tag'lerinin yerlerini `\n` ile dolduruyor. Son olarak da bu `\n`leri kaldırmak için `gsub` prosesörünü tanımlıyoruz.
- `field` parametresi aynı şekilde dokümanın hangi parametresinin işleneceğini belirtiyor.
- `pattern` parametresi alandan kaldırmak istediğimiz ifadeyi belirtiyor.
- `replacement` parametresi alandan kaldırmak istediğimiz ifade yerine ne koyacağımızı belirtiyor.

5. Kurduğumuz pipeline'ın çalışıp çalışmadığını görmek için örnek olarak bir html dosyası yükleyebiliriz.
![html](https://user-images.githubusercontent.com/76704001/177508067-c7f84c4b-d7c6-4b6d-9f5d-9a12eed12355.png)
Dosyayı base64'e çevirdikten sonra pipeline tanımında belirttiğimiz gereksinimleri karşılayacak şekilde bir istek yolluyoruz.
![put req](https://user-images.githubusercontent.com/76704001/177509931-fe8fba2c-63f6-4fba-8681-b46ac7a8ff13.png)
İndekslediğimiz dokümanın `attachment.content` alanında yolladığımız base64'ün okunabilir metine çevrildiğini ve html tag'lerinin kaldırıldığını görebiliriz.
![cok iyi be](https://user-images.githubusercontent.com/76704001/177511926-08b8ff92-23d5-426b-8b8a-100a53742ef0.png)
