# Bölüm 14: BigBlueButton 3.0 (2026 Standartları) ve UX İyileştirmeleri

Artık 2026 yılı itibarıyla eski 2.x sürümleri tamamen terkedilmiş olup, **BigBlueButton 3.x serisi eğitim sektörünün de-facto standardı** haline gelmiştir. BigBlueButton 3.0 serisi, devasa kullanıcı deneyimi (UX) ve performans güncellemelerini olgunlaştıran ve hatasızlaştıran versiyondur. Sistem yöneticileri olarak bu standartların farkında olmak, öğretmen ve öğrencilere verilecek desteği kolaylaştırır.

## 14.1 Yeni TLDraw Beyaz Tahta (Whiteboard)

BBB 3.0 ile birlikte eski klasik Flash-vari beyaz tahta tamamen çöpe atılmış ve modern **TLDraw** altyapısına geçilmiştir.

- **Vektörel Çizim:** Artık kullanıcılar tahtaya çizdikleri nesneleri sonradan seçebilir, taşıyabilir, gruplayabilir veya silebilirler. Eski sürümde çizilen çizim, resmin üzerine kazınmış (bitmap) gibiydi.
- **Performans Algoritması:** Yeni tahta olayları (events) NodeJS ve Redis üzerinden daha az veri paketiyle iletilir. Bu da sunucudaki yükü ciddi oranda düşürür.
- **Gelişmiş Araçlar:** Yapışkan notlar (Sticky notes), akıllı oklar (bağlayıcılar) ve serbest el çizimlerinin daha pürüzsüz (smooth) olması sağlanmıştır.

## 14.2 Kamerada Sanal Arka Plan ve Bulanıklaştırma (Virtual Backgrounds)

Kullanıcıların en büyük taleplerinden biri olan kamera arkasını flulaştırma veya görsel ekleme özelliği 3.0 sürümüyle yerel hale gelmiştir. 

- Bu işlem doğrudan tarayıcı (Client-Side) üzerinde yapılır. Yani sunucuya **ekstra bir CPU yükü binmez**.
- Kullanıcılar kendi arka plan görsellerini yükleyebilirler. 
- *SysAdmin Notu:* Sisteme yüklenebilecek varsayılan kurum arkaplan resimlerini Nginx üzerinden `/var/www/bigbluebutton-default/backgrounds/` tarzında sunup kullanıcıların paneline forced (zorunlu) olarak ekleyebilirsiniz.

## 14.3 Yeni Arayüz Tepkimeleri ve Kamera Düzeni

- **Canlı Tepkiler (Live Reactions):** Kullanıcıların emoji fırlatması ekranda hareketli (animasyonlu) olarak belirir. Öğretmenin anlık nabız ölçmesini kolaylaştırır.
- **Sabit Kamera Oranları:** Önceki sürümlerde insanların kameraları kırpılıyor veya sayfa düzenini bozuyordu. Artık daha esnek, otomatik ızgara (Grid) algoritmaları ve "Kamerayı Öne Çıkar" (Pin/Spotlight) özelliği bulunmaktadır.

## 14.4 Güçlendirilmiş Moderatör Araçları

- **Uyuyan Kullanıcıyı Uyandırma:** Dikkat takibi özelliği sayesinde, BBB sekmesinden başka bir sekmeye geçen veya 10 dakikadır hareket etmeyen kullanıcıları öğretmenler görebilir.
- **Toplu Yönetim:** Breakout (Kırılım) odalarından ana odaya herkesi saniyeler içinde geri çekme, tüm mikrofonları sadece tek bir tuşla anında kitleme (Hard-Mute) gibi aksiyonların API tepki süreleri salise seviyelerine çekilmiştir.

## 14.5 Altyapısal Değişim: Mediasoup ve Ubuntu 22.04 LTS

SysAdmin'lerin bilmesi gereken en büyük altyapı farkı, Kurento'nun artık kod tabanından tamamen kazınmasıdır. Sadece Mediasoup (NodeJS/C++ tabanlı SFU) görev yapmaktadır.
Ayrıca sistem çekirdeği tamamen Ubuntu 22.04 LTS'ye oturmuş durumda olup Ruby, Node.js ve Java versiyonları (API tarafında) son güvenlik yamalarıyla güncellenmiştir.
