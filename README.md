# Multi_Synchronization_RTOS

Bu proje, **STM32** üzerinde **FreeRTOS** kullanarak bir buton kesmesi (EXTI0) ile üç farklı senkronizasyon mekanizmasını (Queue, Counting Semaphore, Task Notification) aynı anda örneklemeyi amaçlar.

## İçindekiler
- [Donanım Yapılandırması](#donanım-yapılandırması)
- [Öncelik ve IRQ Ayarları](#öncelik-ve-irq-ayarları)
- [Stack ve Heap Boyutları](#stack-ve-heap-boyutları)
- [GPIO Ayarları](#gpio-ayarları)
- [Projenin Nasıl Çalıştığı](#projenin-nasıl-çalıştığı)
- [Kullanılan Donanımlar](#kullanılan-donanımlar)
- [Kurulum Adımları](#kurulum-adımları)


---

## Donanım Yapılandırması
- **PendSV, SysTick ve SVCall** kesmeleri için **Generate IRQ** üretilmemeli.
- EXTI0 kesmesi buton girişi (ör. B1_Pin) ile ilişkilendirilmelidir.
- UART2 üzerinden haberleşme sağlanacaksa pin konfigürasyonları (TX, RX) uygun portlara ayarlanmalıdır.

## Öncelik ve IRQ Ayarları
- **Kesme Önceliği**: EXTI0 kesmesi, FreeRTOS ile uyumlu olacak seviyede yapılandırılmalıdır.
  - Örneğin `configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY` değerini aşmamalı.
  - Bu, ISR içerisinde FreeRTOS API fonksiyonlarının (örn. `xQueueSendFromISR`, `xSemaphoreGiveFromISR`) güvenle kullanılabilmesi için gereklidir.

## Stack ve Heap Boyutları
- FreeRTOS görevleri için yeterli **stack** ayrıldığından emin olun. Örnekte her görev için “128” gibi bir değer kullanılmıştır.
- FreeRTOS’un genel **heap** (ör. `configTOTAL_HEAP_SIZE`) boyutu da oluşturulacak görev, semafor, kuyruk vb. için yeterli olmalıdır.
- Bu ayarlar genelde `FreeRTOSConfig.h` içerisinde tanımlanır.

## GPIO Ayarları
- **B1_Pin** (veya projenizde kullandığınız buton pin’i) **EXTI0** olacak şekilde yapılandırılmalıdır.
  - CubeMX kullanıyorsanız **GPIO** sekmesinde `Mode: External Interrupt` olarak seçilebilir.


---

## Projenin Nasıl Çalıştığı
1. **Butona Basıldığında (EXTI0)**:  
   - `button_interrupt_handler()` fonksiyonu çağrılır.  
   - Burada `pressCount` değişkeni arttırılır, **Counting Semaphore** verilir, **Queue**'ya basma sayısı yazılır ve **Task Notification** gönderilir.

2. **Queue** Mekanizması:  
   - `vQueueTask` kuyruktan (Queue) gelecek basma sayısını bekler.  
   - Yeni değer aldığında UART üzerinden `"Button press count = X"` şeklinde bir mesaj basar.

3. **Counting Semaphore** Mekanizması:  
   - `vCountSemTask`, Counting Semaphore’un verilmesini (`xSemaphoreTake`) bekler.  
   - Semafor alındığında `"Semaphore received!"` mesajı gönderir.

4. **Task Notification** Mekanizması:  
   - `vNotifyTask` ise ISR’dan gelen bildirimi (`ulTaskNotifyTake`) bekler.  
   - Bildirim alındığında `"Notification received!"` mesajı gönderir.

Bu sayede bir **buton kesmesi**, üç farklı senkronizasyon yolunu tetikleyebilir ve hepsi kendi içinde işlemlerini yürütür.

---

## Kullanılan Donanımlar
- **STM32 Serisi Mikrodenetleyici** (örneğin STM32F4 Discovery veya Nucleo-64 F4 serisi)
- **Dahili Buton (B1_Pin)** veya harici buton (EXTI0’a bağlı)
- **UART2** (PC ile haberleşmek için USB-UART dönüştürücü)

---

## Kurulum Adımları
1. **STM32CubeMX** ile temel proje oluşturun ve FreeRTOS, UART2, EXTI0 pin ayarlarını yapın.
2. `FreeRTOSConfig.h` dosyasında kesme önceliklerini kontrol edin.
3. `main.c` içine örnek kodu (task’ler, semafor, queue oluşturma kısımları vb.) ekleyin.
4. `.gitignore` ekleyerek derleme çıktı dosyalarını (Debug, Release, *.o, *.d vs.) kayda dahil etmeyin.
5. Projeyi derleyip cihaza yükleyin.
6. Terminal programı (PuTTY, TeraTerm vb.) üzerinden doğru baud rate ile **UART2** çıktılarını takip edin.




