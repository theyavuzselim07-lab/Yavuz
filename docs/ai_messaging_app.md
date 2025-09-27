# Yapay Zeka Destekli Mesajlaşma Uygulaması Tasarımı

## Genel Bakış
- WhatsApp benzeri anlık mesajlaşma deneyimi.
- Yerleşik yapay zekâ destekli asistan: sohbet özetleme, öneri, içerik üretme, toplantı notu çıkarma.
- Hem iOS/Android hem web için tek kod tabanı ile çapraz platform yaklaşımı (React Native + web).
- Uygulama vizyonu: Gerçek zamanlı iletişim + üretken AI iş birliği ile kişisel ve profesyonel sohbetleri hızlandırmak.

## Ürün Hedefleri
- **Yanıt verimliliği:** AI destekli öneriler sayesinde kullanıcıların ortalama cevap süresini %30 azaltmak.
- **Bilgi yönetimi:** Sohbet özetleri ve görev çıkarımı ile kritik kararların kaçırılma oranını %50 düşürmek.
- **Güven ve gizlilik:** Uçtan uca şifreleme ve şeffaf veri politikaları ile kullanıcı memnuniyet skorunu (CSAT) 4.6/5 üzerinde tutmak.
- **Çapraz platform sürekliliği:** Mesaj teslimi ve cihazlar arası senkronizasyon için p95 gecikmeyi 200 ms altında tutmak.

## Kullanıcı Profilleri ve Kayıt
- Telefon numarası doğrulaması ve iki faktörlü kimlik doğrulama
- Kullanıcı adı, profil fotoğrafı, durum mesajı
- Kişi listesi: otomatik rehber eşleştirme + manuel ekleme

## Ana Özellikler
### Sohbetler
- Bire bir ve grup sohbetleri
- Uçtan uca şifreleme (Signal protokolü veya eşdeğeri)
- Mesaj türleri: metin, sesli mesaj, fotoğraf, video, belge, konum
- Mesaj reaksiyonları, emojiler, okundu bilgisini yönetme
- Mesaj düzenleme ve geri çekme (belirli süre içinde)

### Yapay Zeka Asistanı
- Sohbet içi komutlarla tetiklenebilen AI paneli ("@YavuzAI").
- Özetleme: uzun sohbetlerde önemli noktaları çıkartma + otomatik toplantı özeti gönderimi.
- İçerik üretimi: mesaj taslağı, yanıt önerisi, duyuru metni hazırlama, görsel için kısa açıklama üretme.
- Çeviri: seçilen mesajları anında farklı dillere çevirme.
- Akıllı arama: konu/duygu bazlı filtreleme, kişi ve dosya arama.
- Konuşma analitiği: toplantı notu ve görev çıkarımı.
- AI otomasyonları: tetikleyici kelimelerle görev açma (örn. Jira, Trello entegrasyonları) ve takvim daveti hazırlama.

### Çağrı ve Medya
- Sesli ve görüntülü aramalar, grup aramaları
- Ekran paylaşımı ve canlı yayın odaları
- Medya galerisinde yapay zeka ile içerik kategorilendirme (ör. belge vs foto)

### Güvenlik ve Gizlilik
- Uçtan uca şifrelenmiş depolama.
- Zamanlayıcı ile kaybolan mesajlar.
- Gizli sohbet kasası (biyometrik doğrulama).
- AI asistanı tüm veriyi cihaz üzerinde ön işleme tabi tutar, buluta anonim veri gönderilir.
- Veri koruma uyumlulukları: GDPR, KVKK ve SOC2 denetimleri için loglama ve erişim kontrolü.
- Güvenlik olayları için 7/24 SIEM izleme ve otomatik anomali tespiti.

## Mimari
### İstemci
- React Native mobil uygulaması + React web istemcisi.
- Durum yönetimi: Redux Toolkit / Zustand + react-query ile istek önbellekleme.
- WebSockets ile gerçek zamanlı mesajlaşma ve push bildirim desteği (APNs, FCM).
- AI özellikleri için özel panel, istemci tarafı prompt şablonları ve AI öneri kartı bileşenleri.
- Çoklu cihaz desteği: eş zamanlı oturumlar, son mesajın cihazlar arası dağıtımı ve senkronizasyon çakışmalarını çözme stratejileri.

### Sunucu
- Backend: Node.js (NestJS) veya Go tabanlı mikroservisler.
- Kimlik doğrulama servisi (JWT + Refresh token) + donanım bazlı anahtar koruması (HSM).
- Mesaj servisi: WebSocket gateway, mesaj kuyruğu (Kafka), teslim garantisi için en az bir kez mantığı.
- Medya servisi: Dosya depolama (S3 uyumlu), CDN entegrasyonu, içerik denetimi pipeline'ı.
- AI servisi: OpenAI / Azure OpenAI API entegrasyonu + önbellek + prompt yönetim servisi (versiyonlama, güvenlik filtreleri).
- Analitik servisi: Kullanım metrikleri, hata izleme (Prometheus + Grafana) ve olay tabanlı ürün analitiği (PostHog/Amplitude).
- Bildirim servisi: Push/SMS/e-posta tetikleyici orkestrasyonu.

### Veri Tabanı
- Kullanıcı ve sohbet meta verileri: PostgreSQL.
- Mesaj içeriği: Şifrelenmiş olarak MongoDB veya Cassandra.
- Arama ve indeksleme: Elasticsearch + vektör araması (AI önerileri).
- Gerçek zamanlı analitik için ClickHouse veya BigQuery veri ambarı.

## Akışlar
1. **Mesaj Gönderme:** İstemci -> Gateway -> Mesaj servisi -> Depolama -> Alıcı push
2. **AI Özeti:** Kullanıcı komutu -> AI servisi -> Model isteği -> Özet mesajı döner -> Sohbete eklenir
3. **Arama:** Arama sorgusu -> Vektör + tam metin indeks -> Sonuç listesi -> İstemci gösterim

## AI Asistanı Etkileşim Akışları
| Senaryo | Kullanıcı Aksiyonu | AI Adımları | Çıktı |
| --- | --- | --- | --- |
| Sohbet Özeti | Kullanıcı "@YavuzAI özetle" yazar | Mesaj geçmişi toplanır, kişisel veriler maskelenir, GPT-4.1 mini ile özet üretilir | Sohbet içinde özet kartı |
| Yanıt Önerisi | Mesaj uzun basılır -> "AI önerisi" | Prompt şablonu seçilir, bağlam mesajları eklenir, ton/emoji politikası uygulanır | Kullanıcı düzenleyebileceği taslak |
| Görev Çıkarımı | Toplantı sonu komutu | Anahtar karar/aksiyon listesi çıkarılır, Trello/Jira webhook'u tetiklenebilir | Görev listesi + entegrasyon | 
| Çeviri | Kullanıcı mesaj seçer -> "Çevir" | Dil algılama yapılır, hedef dil seçimi UI'da | Çevrilmiş mesaj balonu |

## AI Güvenliği ve Etik İlkeler
- **Veri minimizasyonu:** Model girişine sadece gerekli sohbet parçaları dahil edilir, kimlik bilgileri maskeleme kuralları uygulanır.
- **İzin kontrolü:** Grup sohbetlerinde AI kullanımına dair varsayılan olarak tüm katılımcılardan onay istenir.
- **Şeffaflık:** AI tarafından üretilen içerikler "AI tarafından üretildi" etiketi taşır, düzenleme geçmişi saklanır.
- **İnsan denetimi:** Kritik görev oluşturma, dış entegrasyon tetiklemeleri için kullanıcı doğrulaması gerekir.
- **Kötüye kullanım önleme:** Toxicity, spam ve zararlı içerik filtreleri (OpenAI Moderation + özel kurallar) pipeline'da çalışır.
- **Model gözden geçirmesi:** A/B testleri, hallucination oranı takibi ve kullanıcı geri bildirimi ile periyodik model değerlendirmesi yapılır.

## Yapay Zeka Modeli Seçimi
- Metin için GPT-4.1 mini tabanlı modeller.
- Ses transkripsiyonu için Whisper large-v3.
- İstemci tarafında hafif öneriler için küçük dil modelleri (Llama3 8B edge).
- Özel alan bilgisini güçlendirmek için domain verisiyle ince ayar yapılmış adapter modelleri.
- Model yönetişimi: Prompt/cevap kayıtları için 30 günlük güvenli saklama, anormallik izleme.

## Yol Haritası
1. Temel mesajlaşma altyapısı (MVP): kullanıcı kayıt, sohbet, medya gönderimi, uçtan uca şifreleme.
2. AI asistanı çekirdeği: özetleme, yanıt önerisi, çeviri ve moderasyon pipeline'ı.
3. Medya yönetimi ve arama optimizasyonu: AI destekli klasörleme, gelişmiş filtreler, performans tuning.
4. Gelişmiş gizlilik özellikleri ve üçüncü parti entegrasyonlar: kurum hesapları, SSO, takvim/iş yönetimi entegrasyonları.
5. Monetizasyon ve premium paketler: gelişmiş AI limitleri, yönetici konsolu, uyumluluk raporları.

## Başarı Metrikleri
- Günlük/aylık aktif kullanıcı sayısı.
- Mesaj teslim süresi (p95 < 200ms).
- AI önerisi kullanım oranı ve öneri kabul oranı.
- AI kaynaklı şikâyet/hallucination oranı.
- Kullanıcı memnuniyet anketleri ve NPS.
- Premium paket dönüşüm oranı.

## Tasarım Notları
- Modern, minimal UI: koyu/açık tema, kişiselleştirilebilir sohbet arayüzleri.
- Erişilebilirlik: ekran okuyucu desteği, yüksek kontrast mod, sesli komutlar.
- Performans: medya optimizasyonu, çevrimdışı önbellek, agresif paketleme stratejileri (CodePush, OTA güncellemeler).
- Lokalizasyon: 12+ dil desteği, sağdan sola diller için UI uyarlamaları.

## Operasyon ve DevOps
- CI/CD: GitHub Actions + Fastlane ile otomatik build, test, dağıtım.
- Gözlemlenebilirlik: Dağıtılmış izleme (OpenTelemetry), hata raporlama (Sentry), kullanıcı davranış analizi.
- Sürüm stratejisi: Kanarya yayınları, beta programı, özellik bayrakları ile kontrollü açılış.
- SLA/SLO tanımları: Kritik servisler için %99.9 uptime hedefi, incident response playbook'ları.
- Destek süreçleri: Uygulama içi yardım merkezi, AI destekli self-servis rehber, canlı destek entegrasyonu.
