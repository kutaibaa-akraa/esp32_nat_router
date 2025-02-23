# موجه NAT باستخدام ESP32 مع دعم WPA2 Enterprise

هذا البرنامج الثابت لاستخدام ESP32 كموجه NAT لشبكة WiFi. يمكن استخدامه كـ:
- ممدد نطاق بسيط لشبكة WiFi موجودة.
- إعداد شبكة WiFi إضافية بمعرف SSID/كلمة مرور مختلفة للضيوف أو أجهزة IoT.
- تحويل شبكة مؤسسية (WPA2-Enterprise) إلى شبكة عادية، للأجهزة البسيطة.

يمكنه تحقيق عرض نطاق يزيد عن 15 ميجابت/ثانية.

يعتمد الكود على [مكون الكونسول](https://docs.espressif.com/projects/esp-idf/en/latest/api-guides/console.html#console) و[مثال esp-idf-nat-example](https://github.com/jonask1337/esp-idf-nat-example).

## الأداء

تم استخدام `IPv4` وبروتوكول `TCP` في جميع الاختبارات.

| اللوحة | الأدوات | التحسين | تردد المعالج | الإنتاجية | الطاقة |
| ----- | ----- | ------------ | ------------- | ---------- | ----- |
| `ESP32D0WDQ6` | `iperf3` | `0g` | `240MHz` | `16.0 ميجابت/ثانية` | `1.6 واط` |
| `ESP32D0WDQ6` | `iperf3` | `0s` | `240MHz` | `10.0 ميجابت/ثانية` | `1.8 واط` | 
| `ESP32D0WDQ6` | `iperf3` | `0g` | `160MHz` | `15.2 ميجابت/ثانية` | `1.4 واط` |
| `ESP32D0WDQ6` | `iperf3` | `0s` | `160MHz` | `14.1 ميجابت/ثانية` | `1.5 واط` |

## الإقلاع الأول
بعد الإقلاع الأول، سيقدم ESP32 NAT Router شبكة WiFi مع نقطة وصول مفتوحة ومعرف SSID "ESP32_NAT_Router". يمكن إجراء التكوين إما عبر واجهة ويب بسيطة أو عبر الكونسول التسلسلي.

## واجهة تكوين الويب
تسمح واجهة الويب بتكوين جميع المعلمات. قم بتوصيل جهاز الكمبيوتر أو الهاتف الذكي بشبكة WiFi SSID "ESP32_NAT_Router" وتوجه إلى "http://192.168.4.1". يجب أن تظهر هذه الصفحة:

<img src="https://raw.githubusercontent.com/marci07iq/esp32_nat_router/master/ESP32_NAT_UI3.png">

أولاً، أدخل القيم المناسبة لشبكة WiFi uplink، "إعدادات STA". اترك كلمة المرور فارغة للشبكات المفتوحة. انقر على "اتصال". سيعيد ESP32 التشغيل وسيتصل بموجه WiFi الخاص بك.

الآن يمكنك إعادة الاتصال وإعادة تحميل الصفحة وتغيير "إعدادات Soft AP". انقر على "تعيين" وسيعيد ESP32 التشغيل مرة أخرى. الآن أصبح جاهزًا لتوجيه حركة المرور عبر Soft AP الجديد. كن على علم بأن هذه التغييرات تؤثر أيضًا على واجهة التكوين، أي لإجراء المزيد من التكوين، قم بالاتصال بـ ESP32 عبر إحدى شبكات WiFi الجديدة التي تم تكوينها.

إذا كنت تريد إدخال '+' في واجهة الويب، يجب عليك استخدام ترميز HTTP السداسي مثل "Mine%2bYours". سيؤدي هذا إلى إنشاء سلسلة "Mine+Yours". باستخدام هذا الترميز السداسي، يمكنك إدخال أي قيمة بايت تريدها، باستثناء 0 (لأسباب داخلية في لغة C).

إذا كنت تريد تعطيل واجهة الويب (على سبيل المثال، لأسباب أمنية)، انتقل إلى CLI وأدخل:
```
nvs_namespace esp32_nat
nvs_set lock str -v 1
```
بعد إعادة التشغيل، لن يتم بدء خادم الويب بعد الآن. يمكنك فقط إعادة تمكينه باستخدام:
```
nvs_namespace esp32_nat
nvs_set lock str -v 0
```
إذا قمت بخطأ وفقدت كل الاتصال بـ ESP، يمكنك استخدام الكونسول التسلسلي لإعادة تكوينه. يتم تخزين جميع إعدادات المعلمات في NVS (التخزين غير المتطاير)، والذي *لا* يتم مسحه بإعادة تثبيت الثنائيات البسيطة. إذا كنت تريد مسحه، استخدم "esptool.py -p /dev/ttyUSB0 erase_flash".

## الوصول إلى الأجهزة خلف الموجه

إذا كنت تريد الوصول إلى جهاز خلف موجه NAT ESP32؟ `PC -> الموجه المحلي -> esp32NAT -> الخادم`

لنفترض أن "الخادم" يعرض خادم ويب على المنفذ 80 وتريد الوصول إليه من جهاز الكمبيوتر الخاص بك.  
لذلك، تحتاج إلى تكوين portmap (على سبيل المثال، عن طريق الاتصال عبر شاشة UART لـ Arduino IDE عبر USB)

```
portmap add TCP 8080 192.168.4.2 80
                                 ↑ منفذ خادم الويب
                            ↑ عنوان IP للخادم في شبكة esp32NAT
                  ↑ المنفذ المعروض في شبكة الموجه المحلي
```
     
بافتراض أن عنوان IP لـ esp32NAT في `الموجه المحلي` الخاص بك هو `192.168.0.57`، يمكنك الآن الوصول إلى الخادم عن طريق كتابة `192.168.0.57:8080` في المتصفح.

## تفسير LED المدمج

إذا كان ESP32 متصلاً بـ AP uplink، فيجب أن يكون LED المدمج مضاءً، وإلا فهو مطفأ.
إذا كانت هناك أجهزة متصلة بـ ESP32، فسيومض LED المدمج عدة مرات مثل عدد الأجهزة المتصلة.

على سبيل المثال:

جهاز واحد متصل بـ ESP32، و ESP32 متصل بـ uplink: 

`*****.*****`

جهازان متصلان بـ ESP32، ولكن ESP32 غير متصل بـ uplink: 

`....*.*....`

# واجهة سطر الأوامر

للتكوين، يجب استخدام كونسول تسلسلي (Putty أو GtkTerm بسرعة 115200 باود).
استخدم الأمر "set_sta" والأمر "set_ap" لتكوين إعدادات WiFi. يتم تخزين التغييرات بشكل دائم في NVS ويتم تطبيقها بعد إعادة التشغيل التالية. استخدم "show" لعرض التكوين الحالي. مساحة الاسم NVS للمعلمات هي "esp32_nat".

أدخل الأمر `help` للحصول على قائمة كاملة بجميع الأوامر المتاحة:
```
help 
  طباعة قائمة الأوامر المسجلة

free 
  الحصول على الحجم الحالي للذاكرة الحرة

heap 
  الحصول على الحد الأدنى لحجم الذاكرة الحرة المتاحة أثناء تنفيذ البرنامج

version 
  الحصول على إصدار الشريحة و SDK

restart 
  إعادة تشغيل البرنامج للشريحة

deep_sleep  [-t <t>] [--io=<n>] [--io_level=<0|1>]
  الدخول في وضع السكون العميق. يتم دعم وضعين للاستيقاظ: المؤقت و GPIO. إذا لم يتم تحديد خيار استيقاظ، سينام إلى أجل غير مسمى.
  -t, --time=<t>  وقت الاستيقاظ، بالمللي ثانية
      --io=<n>  إذا تم تحديده، الاستيقاظ باستخدام GPIO بالرقم المحدد
  --io_level=<0|1>  مستوى GPIO لتحريك الاستيقاظ

light_sleep  [-t <t>] [--io=<n>]... [--io_level=<0|1>]...
  الدخول في وضع السكون الخفيف. يتم دعم وضعين للاستيقاظ: المؤقت و GPIO. يمكن تحديد عدة دبابيس GPIO باستخدام أزواج من الوسائط 'io' و 'io_level'. سيتم أيضًا الاستيقاظ على إدخال UART.
  -t, --time=<t>  وقت الاستيقاظ، بالمللي ثانية
      --io=<n>  إذا تم تحديده، الاستيقاظ باستخدام GPIO بالرقم المحدد
  --io_level=<0|1>  مستوى GPIO لتحريك الاستيقاظ

tasks 
  الحصول على معلومات عن المهام قيد التشغيل

nvs_set  <key> <type> -v <value>
  تعيين زوج مفتاح-قيمة في مساحة الاسم المحددة.
أمثلة:
 nvs_set VarName i32 -v 
  123 
 nvs_set VarName str -v YourString 
 nvs_set VarName blob -v 0123456789abcdef 
         <key>  مفتاح القيمة المراد تعيينها
        <type>  النوع يمكن أن يكون: i8, u8, i16, u16 i32, u32 i64, u64, str, blob
  -v, --value=<value>  القيمة المراد تخزينها

nvs_get  <key> <type>
  الحصول على زوج مفتاح-قيمة من مساحة الاسم المحددة. 
مثال: nvs_get VarName i32
         <key>  مفتاح القيمة المراد قراءتها
        <type>  النوع يمكن أن يكون: i8, u8, i16, u16 i32, u32 i64, u64, str, blob

nvs_erase  <key>
  مسح زوج مفتاح-قيمة من مساحة الاسم الحالية
         <key>  مفتاح القيمة المراد مسحها

nvs_namespace  <namespace>
  تعيين مساحة الاسم الحالية
   <namespace>  مساحة الاسم المحددة

nvs_list  <partition> [-n <namespace>] [-t <type>]
  قائمة بأزواج المفتاح-القيمة المخزنة في NVS. يمكن تحديد مساحة الاسم والنوع لطباعة أزواج المفتاح-القيمة هذه فقط.
  
الأمر التالي يعرض المتغيرات المخزنة داخل قسم 'nvs'، تحت مساحة الاسم 'storage' مع النوع uint32_t
  مثال: nvs_list nvs -n storage -t u32 

   <partition>  اسم القسم
  -n, --namespace=<namespace>  اسم مساحة الاسم
  -t, --type=<type>  النوع يمكن أن يكون: i8, u8, i16, u16 i32, u32 i64, u64, str, blob

nvs_erase_namespace  <namespace>
  مسح مساحة الاسم المحددة
   <namespace>  مساحة الاسم المراد مسحها

set_sta  <ssid> <passwd>
  تعيين SSID وكلمة المرور لواجهة STA
        <ssid>  SSID
      <passwd>  كلمة المرور
  --, -u, ----username=<ent_username>  اسم مستخدم المؤسسة
  --, -a, ----anan=<ent_identity>  هوية المؤسسة

set_sta_static  <ip> <subnet> <gw>
  تعيين IP ثابت لواجهة STA
          <ip>  IP
      <subnet>  قناع الشبكة الفرعية
          <gw>  عنوان البوابة

set_ap  <ssid> <passwd>
  تعيين SSID وكلمة المرور لـ SoftAP
        <ssid>  SSID لـ AP
      <passwd>  كلمة المرور لـ AP

set_ap_ip  <ip>
  تعيين IP لواجهة AP
          <ip>  IP

portmap  [add|del] [TCP|UDP] <ext_portno> <int_ip> <int_portno>
  إضافة أو حذف تعيين منفذ إلى الموجه
     [add|del]  إضافة أو حذف تعيين المنفذ
     [TCP|UDP]  منفذ TCP أو UDP
  <ext_portno>  رقم المنفذ الخارجي
      <int_ip>  IP الداخلي
  <int_portno>  رقم المنفذ الداخلي

show 
  الحصول على حالة وتكوين الموجه
```

إذا كنت تريد إدخال أحرف غير ASCII أو خاصة (بما في ذلك ' ')، يمكنك استخدام ترميز HTTP السداسي (على سبيل المثال، "My%20AccessPoint" يؤدي إلى إنشاء سلسلة "My AccessPoint").

## تعيين إخراج الكونسول إلى UART أو USB_SERIAL_JTAG (USB-OTG)
جميع لوحات ESP32 الأحدث تحتوي على [وحدة تحكم USB Serial/JTAG](https://docs.espressif.com/projects/esp-idf/en/latest/esp32c3/api-guides/usb-serial-jtag-console.html). 
إذا كان منفذ USB متصلاً مباشرة بوحدة تحكم USB Serial/JTAG، فلن تتمكن من استخدام الكونسول عبر UART.

يمكنك تغيير إخراج الكونسول إلى USB_SERIAL_JTAG:

**Menuconfig:**
`Component config` -> `ESP System Settings` -> `Channel for console output` -> `USB Serial/JTAG Controller`

**تغيير sdkconfig مباشرة**
```
CONFIG_ESP_CONSOLE_UART_DEFAULT=n
CONFIG_ESP_CONSOLE_USB_SERIAL_JTAG=y
```

[قائمة مقارنة اللوحات](https://docs.espressif.com/projects/esp-idf/en/v5.0.4/esp32/hw-reference/chip-series-comparison.html)

## تثبيت الثنائيات المسبقة البناء

احصل وثبّت [esptool](https://github.com/espressif/esptool):

```
cd ~
python3 -m pip install pyserial
git clone https://github.com/espressif/esptool
cd esptool
python3 setup.py install
```

انتقل إلى دليل مشروع esp32_nat_router وقم بالبناء لأي نوع من أهداف esp32.

لـ esp32:

```bash
esptool.py --chip esp32 \
--before default_reset --after hard_reset write_flash \
-z --flash_mode dio --flash_freq 40m --flash_size detect \
0x1000 build/esp32/bootloader.bin \
0x8000 build/esp32/partitions.bin \
0x10000 build/esp32/firmware.bin
```

لـ esp32c3:

```bash
esptool.py --chip esp32c3 \
--before default_reset --after hard_reset write_flash \
-z --flash_size detect \
0x0 build/esp32c3/bootloader.bin \
0x8000 build/esp32c3/partitions.bin \
0x10000 build/esp32c3/firmware.bin
```

كبديل، يمكنك استخدام [أدوات تنزيل Flash من Espressif](https://www.espressif.com/en/products/hardware/esp32/resources) مع المعلمات الموضحة في الشكل أدناه (بفضل mahesh2000)، قم بتحديث أسماء الملفات وفقًا لذلك:

![image](https://raw.githubusercontent.com/martin-ger/esp32_nat_router/master/FlasherUI.jpg)

لاحظ أن الثنائيات المسبقة البناء لا تتضمن دعم WPA2 Enterprise.

## بناء الثنائيات (الطريقة 1 - ESPIDF)
فيما يلي الخطوات المطلوبة لتجميع هذا المشروع:

1. قم بتنزيل وإعداد ESP-IDF.

2. في دليل المشروع، قم بتشغيل `make menuconfig` (أو `idf.py menuconfig` لـ cmake).
    1. *Component config -> LWIP > [x] Enable copy between Layer2 and Layer3 packets.
    2. *Component config -> LWIP > [x] Enable IP forwarding.
    3. *Component config -> LWIP > [x] Enable NAT (new/experimental).
3. قم ببناء المشروع وقم بتثبيته على ESP32.

يمكن العثور على تعليمات مفصلة حول كيفية بناء وتكوين وتثبيت مشروع ESP-IDF في الدليل الرسمي لـ ESP-IDF.

## بناء الثنائيات (الطريقة 2 - Platformio)
فيما يلي الخطوات المطلوبة لتجميع هذا المشروع:

1. قم بتنزيل Visual Studio Code، وملحق Platform IO.
2. في Platformio، قم بتثبيت إطار عمل ESP-IDF.
3. قم ببناء المشروع وقم بتثبيته على ESP32.

### DNS
بمجرد أن يتعلم ESP32 STA عنوان IP لخادم DNS من خادم DNS uplink الخاص به عند الاتصال الأول، فإنه يمرر ذلك إلى العملاء المتصلين حديثًا.
قبل ذلك، بشكل افتراضي، يتم تعيين خادم DNS الذي يتم تقديمه للعملاء المتصلين بـ ESP32 AP إلى 8.8.8.8.
استبدل قيمة *MY_DNS_IP_ADDR* بعنوان IP لخادم DNS المطلوب (بالهيكس) إذا كنت تريد استخدام خادم DNS مختلف.

## استكشاف الأخطاء وإصلاحها

### نهايات الأسطر

تم تكوين نهايات الأسطر في مثال الكونسول لتتناسب مع شاشات تسلسلية معينة. لذلك، إذا ظهرت سجل الإخراج التالي، ففكر في استخدام شاشة تسلسلية مختلفة (على سبيل المثال، Putty لنظام Windows أو GtkTerm على Linux) أو قم بتعديل تكوين UART في المثال.

```
This is an example of ESP-IDF console component.
Type 'help' to get the list of commands.
Use UP/DOWN arrows to navigate through command history.
Press TAB when typing command name to auto-complete.
Your terminal application does not support escape sequences.
Line editing and history features are disabled.
On Windows, try using Putty instead.
esp32>
```