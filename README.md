HA-NR-WaktuSolatJakim

Waktu Solat (eSolat JAKIM) Home Assistant menggunakan Node Red. ![visitors](https://visitor-badge.glitch.me/badge?page_id=anas-ivs.ha-nr-waktusolatjakim.visitor-badge)

[Flow Logik](#flow) | [Bagaimana ia berfungsi?](#How) | [Pra-syarat](#Pre) | [Pemasangan](#Install) | [Lovelace](#lovelace) | [Kredit](#Credits) |

![Node-Red Header of HA-NR-WaktuSolatJakim](https://github.com/anas-ivs/HA-NR-WaktuSolatJakim/blob/main/images/header-HA-NR-WaktuSolatJakim.PNG)

Idea dan logik original perkongsian daripada komuniti [Home Assistant Malaysia](https://www.facebook.com/groups/homeassistantmalaysia) termasuk dari 

1. [farxpeace](https://github.com/farxpeace/Home-Assistant-Waktu-Solat-Jakim) yang menggunakan REST calls ke [AzanPro API's](https://api.azanpro.com/). 
2. [Zubir2K](https://github.com/zubir2k/HomeAssistantAdzan?fbclid=IwAR2W31BKaf1-0ADuzTA0zkea0T-tZ7IDa6s5i38-rO0YBjb85t8l7G67iHw) yang menggunakan REST call ke JAKIM eSolat API. 
3. [Niezarm](https://github.com/Niezarm/AzanForHuluLangatZone) khusus bagi zone Hulu Langat.
4. [A Jim AI](https://github.com/jimmy93) bagi link eSolat JSON API

Kelebihan menggunakan Node-Red untuk Waktu Solat ini termasuklah pengaturcaraan yang (jauh) lebih mudah berbanding pengaturcaraan YAML di dalam Home Assistant. Kelebihan ini membolehkan *feature-feature* asas dan tambahan seperti:

1. Tarik (*pull*) dan simpanan data eSolat secara harian, mingguan/bulanan ataupun tahunan - bagi tempoh selain harian, ini membolehkan *automation* masih boleh berjalan jika API/server JAKIM *down.* 
2. Penghantaran Waktu Solat ke Home Assistant sebagai `sensor` untuk *display* di `lovelace`, ataupun untuk digunakan di dalam `automations/scripts` yang ditetapkan dari Home Assistant.
3. Notifikasi melalui Telegram atau Home Assistant companion app secara *push* atau *pull* (*demand)*.
4. Membuat Notifikasi Waktu Solat termasuk;
   1. Notifikasi 15-minit sebelum waktu solat.
   2. Bagi 15-minit sebelum - Automation seperti *play broadcast* live Youtube Makkah atau Madinah  feed ke `chromecast` devices. Sebagai tambahan, pemilihan stream boleh dibuat secara rawak.
   3. Menghidupkan TV melalui node `Wake-on-LAN (WOL)` supaya video azan dapat dimainkan.
   4. Pengumuman waktu menggunakan `Google TTS` ke speaker/chromecast devices.
   5. Membuat aturcaraan spesifik jika masuk waktu Solat Subuh seperti memainkan Azan Subuh.
   6. Penutupan TV secara automatik atau pembatalan penutupan TV melalui notifikasi ke Telefon.
5. `Trigger` bagi menjalankan automation lain seperti membuka lampu luar dan menutup langsir 15 minit sebelum maghrib. 
6. Menentukan tempoh sekarang dan pengiraan tempoh waktu hingga ke Solat seterusnya.
7. Tamabahan - Waktu Solat Node-Red dashboard. Credit to originator flow [@aitalinassim](https://flows.nodered.org/flow/9d9a3abe9707d605c6b12d21ddf08658)/

![Telegram sample](telegram statistics)

![Node-Red Dashboard](https://github.com/anas-ivs/HA-NR-WaktuSolatJakim/blob/main/images/NodeRed%20WaktuSolat%20Dashboard.PNG)

## Flow Logik

Logik flow sebenarnya disediakan di hujung sekali agar pembacaan berikut dibuat dahulu untuk memahami bagaimana ia berfungsi, apa yang perlu ditetapkan dan  pra-syarat yang perlu dipenuhi. 

Setiap bahagian di bawah akan memberi sampel code bagi menerangkan fungsi-fungsi berkaitan. Adalah digalakkan untuk import flow logik secara kesuluruhan berbanding setiap bahagian berikut.

Jika sudah bersedia - bolehlah terus lompat terus ke [sini](#final_flow)


## <a name="How">Bagaimana ia berfungsi? </a>
### Bahagian 1 -  Download Waktu Solat Bulanan
1.  Sebulan sekali, lebih tepat pada setiap 1hb bagi bulan tersebut, jam 1 pagi - Node `cronplus` akan membuat panggilan ke API eSolat JAKIM untuk mendapatkan waktu solat bagi bulan tersebut (terkini).
2.  Data yang diterima dalam bentuk `json` ini akan disimpan melalui `Save to File`  ke file path `/share/waktu_solat/waktu_solat_month.json`. 
> **PERHATIAN** Lokasi  `ZONE`  hendaklah ditetapakan dahulu dalam node `Solat API dan Zone`.  Ubah di line `zone_api = 'SWK02'; 
>
> Rujukan bagi zone yang boleh dipakai boleh dirujuk [sini](https://api.azanpro.com/zones).

![](C:\Users\user\Documents\GitHub\HA-NR-WaktuSolatJakim\images\Step1-Bulanan_cmp.PNG)

```json
[{"id":"928da9fc4f062354","type":"tab","label":"Flow 2","disabled":false,"info":""},{"id":"c877b636.6a9488","type":"comment","z":"928da9fc4f062354","name":"Monthly Download","info":"### ## ### \n### ## ### additional nodes\n### ## ### \n### ## ### node-red-contrib-random-item \n# Via Cronjob","x":150,"y":60,"wires":[]},{"id":"545a4ddd.e0c3d4","type":"function","z":"928da9fc4f062354","name":"Solat API and Zone","func":"\n//Define ZONE here\n\nconst zone_api = 'SWK02';\n\n\nif (flow.get(\"waktu_solat_api\")===undefined)  {\n    \nflow.set(\"waktu_solat_api\", \"JAKIM\");\n}   \n\n//get global variable\nvar g = global.get(\"homeassistant\");\n//get states variable\nvar states = g.homeAssistant.states;\n//get the actual entity that we want\nvar waktu_solat_api = states[\"input_select.waktu_solat_api\"].state;\n\nflow.set(\"waktu_solat_api\", waktu_solat_api);\n\nmsg.waktu_solat_api = waktu_solat_api;\n\nif (waktu_solat_api == \"JAKIM\")\n{\nmsg.url = \"https://www.e-solat.gov.my/index.php?r=esolatApi%2Ftakwimsolat&period=month&zone=\"+zone_api;\n}\nelse if (waktusolat_source == \"AzanPro\")\n{\n    //not yet working use JAKIM\n    //msg.url = \"http://api.azanpro.com/times/today.json?zone=\"+zone_api+\"&format=24-hour\";\n    msg.url = \"https://www.e-solat.gov.my/index.php?r=esolatApi%2Ftakwimsolat&period=month&zone=\"+zone_api;\n\n}\nreturn msg;\n\n\n","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":530,"y":220,"wires":[["1b8368da.094bc7"]]},{"id":"6037208e.54784","type":"inject","z":"928da9fc4f062354","name":"Tekan sekali untuk kali pertama","props":[],"repeat":"","crontab":"","once":false,"onceDelay":"0.1","topic":"","x":290,"y":160,"wires":[["545a4ddd.e0c3d4"]]},{"id":"17261447.7fa05c","type":"debug","z":"928da9fc4f062354","name":"bulanan","active":false,"tosidebar":true,"console":false,"tostatus":false,"complete":"true","targetType":"full","statusVal":"","statusType":"auto","x":1160,"y":100,"wires":[]},{"id":"1b8368da.094bc7","type":"http request","z":"928da9fc4f062354","name":"JAKIM","method":"GET","ret":"obj","paytoqs":"ignore","url":"","tls":"","persist":false,"proxy":"","authType":"","x":710,"y":220,"wires":[["8cc47f84.fdc0f"]]},{"id":"b318b0c4.ff895","type":"file","z":"928da9fc4f062354","name":"Save to File","filename":"/share/waktu_solat/waktu_solat_month.json","appendNewline":false,"createDir":true,"overwriteFile":"true","encoding":"none","x":990,"y":160,"wires":[["17261447.7fa05c","add6f5c.7a71b08"]]},{"id":"712b7fca.e8109","type":"cronplus","z":"928da9fc4f062354","name":"Monthly Refresh","outputField":"payload","timeZone":"","persistDynamic":false,"commandResponseMsgOutput":"output1","outputs":1,"options":[{"name":"schedule1","topic":"schedule1","payloadType":"default","payload":"","expressionType":"cron","expression":"0 0 1 1 * ? *","location":"","offset":"0","solarType":"all","solarEvents":"sunrise,sunset"}],"x":160,"y":220,"wires":[["545a4ddd.e0c3d4"]]},{"id":"5b1dea78.548f44","type":"link in","z":"928da9fc4f062354","name":"i-trigger_solat_update","links":["effa5976.3108a8"],"x":335,"y":260,"wires":[["545a4ddd.e0c3d4"]]},{"id":"8cc47f84.fdc0f","type":"switch","z":"928da9fc4f062354","name":"QC","property":"statusCode","propertyType":"msg","rules":[{"t":"eq","v":"200","vt":"str"},{"t":"else"}],"checkall":"true","repair":false,"outputs":2,"x":830,"y":220,"wires":[["b318b0c4.ff895"],["bbf03c33.abaf9","55d5387a.1e6018"]]},{"id":"bbf03c33.abaf9","type":"debug","z":"928da9fc4f062354","name":"ERROR!","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"true","targetType":"full","statusVal":"","statusType":"auto","x":1080,"y":220,"wires":[]},{"id":"780f0eb6.39ec5","type":"link out","z":"928da9fc4f062354","name":"O-TelegramP1","links":["8718a1dd.5beb7"],"x":1235,"y":300,"wires":[],"icon":"node-red-contrib-telegrambot/telegram.png"},{"id":"add6f5c.7a71b08","type":"link out","z":"928da9fc4f062354","name":"o-trigger_solat_update","links":["45a9b7f2.cb00b8"],"x":1135,"y":160,"wires":[]},{"id":"12c47578.5297db","type":"telegram command","z":"928da9fc4f062354","name":"/refreshwaktusolat","command":"/refreshwaktusolat","description":"","registercommand":false,"language":"","bot":"","strict":false,"hasresponse":true,"useregex":false,"removeregexcommand":false,"outputs":2,"x":270,"y":100,"wires":[["545a4ddd.e0c3d4"],[]]},{"id":"c09e234b.c4258","type":"catch","z":"928da9fc4f062354","name":"","scope":["c89028ea.98f938","72b72987.9187c8","545a4ddd.e0c3d4","74e66f38.426cb","277ffb75.6e3b34","367f3e83.607102"],"uncaught":false,"x":830,"y":300,"wires":[["f4bf6ebd.84eee","bbf03c33.abaf9"]]},{"id":"f4bf6ebd.84eee","type":"template","z":"928da9fc4f062354","name":"Telegram Text","field":"payload","fieldType":"msg","format":"handlebars","syntax":"mustache","template":"⚠️⚠️⚠️🕋 \nWaktu Solat Flow Error\nSource: {{error.source.name}}\n {{error.message}}","output":"str","x":1100,"y":300,"wires":[["780f0eb6.39ec5"]]},{"id":"55d5387a.1e6018","type":"template","z":"928da9fc4f062354","name":"Telegram Text","field":"payload","fieldType":"msg","format":"handlebars","syntax":"mustache","template":"⚠️⚠️⚠️🕋 Data waktu solat bulan ini gagal untuk download dari {{waktu_solat_api}}\n\nError code: {{statusCode}}","output":"str","x":1100,"y":260,"wires":[["780f0eb6.39ec5"]]}]
```



### Bahagian 2 - Baca dan tetapkan Waktu Solat Harian

1.  Sehari sekali, lebih tepat setiap hari jam 1 pagi - Node `cronplus` kali ini akan *load* `waktu_solat_month.json` yang telah disimpan sebelum ini di lokasi `share/waktu_solat/`
2.  Node `Store Solat JAKIM to flow` pula akan membuat beberapa perbandingan dan tetapan seperti:
    1.  Load `array` Waktu Solat bagi hari tersebut di dalam file `waktu_solat_month.json`.
    2.  Membuat perbandingan jika data yang di-*load* adalah sah, melalui perbandingan jika bulan hari ini dan yang ditetapkan dalam fail adalah sama.
    3.  Jika sah, dan waktu terkini selepas jam 8 malam - data waktu solat hari berikutnya akan diambil. Jika sebelum jam 8 malam, makan data bagi hari tersebut akan diambil dan ditetapkan.
    4.  Data Waktu Solat ini :
        1.  Dihantar ke `flow` seterusnya sebagai `sensor.waktu_solat` untuk kegunaan Home Assistant. 
        2.  Disimpan di dalam `Flow Context` membolehkan `node` lain dalam Flow ini merujuk data ini. 
    5.   Data waktu Solat Jakim juga memberi tarikh Hijri - bulan (dalam digit) dipadankan dengan nama bulan bulan Islam. Logik asal membuat perkiraan sendiri melalui [script](https://github.com/xsoh/Hijri.js)
    
    ![](C:\Users\user\Documents\GitHub\HA-NR-WaktuSolatJakim\images\Step2-Harian_cmp.PNG)

```json
[{"id":"72b72987.9187c8","type":"function","z":"979d82cfea624e5f","name":"Store JAKIM SOLAT Data to Flow","func":"// Waktu Solat JAKIM\n// anas-ivs. 4th August 2021\n\nvar today           = new Date();\nvar dd              = String(today.getDate()).padStart(2, '0');\nvar mm              = String(today.getMonth() + 1).padStart(2, '0'); //January is 0!\nvar yyyy            = today.getFullYear();\nvar current_time    = today.getHours();\ntoday               = mm + '/' + dd + '/' + yyyy;\n\n// data validity check :\nvar date_validation = msg.payload.prayerTime[dd-1].date.split(\"-\");\n\n// waktu_solat_hari_ini.date is in format DD-MONTH-YYYY. Month in Malay. Lookup array to match\nvar dict_bulan_bm   = [\"Januari\", \"Februari\", \"Mac\", \"April\", \"Mei\", \"Jun\", \"Julai\", \"Ogos\", \"September\", \"October\", \"November\", \"Disember\" ];\n\nif ( dd != date_validation[0] )\n{\n    msg.topic = \"ERROR\"\n    msg.payload =\"Gagal Data Validation - Hari tak padan\"\n    return msg;\n}\nif ( dict_bulan_bm[mm-1] != date_validation[1] )\n{\n    msg.topic = \"ERROR\"\n    msg.payload =\"Gagal Data Validation - Bulan tak sama\"\n    return msg;\n}\nif ( yyyy != date_validation[2] )\n{\n    msg.topic = \"ERROR\"\n    msg.payload =\"Gagal Data Validation - Tahun dah lain ni\"\n    return msg;\n}\n\n// debugging\n// msg.date_validation = date_validation;\n// msg.data_dd = dd;\n// msg.data_mm = mm;\n// msg.data_yyyy = yyyy;\n\nif (current_time >= 20) {\n    // first check if end of month - would be unable to load data.\n    // dd > array length from JSON\n    // what do we do? ideally load next month data but for now \n    // we ignore and wait for next month data fetch call up by CRON @ midnight\n    if (parseInt(dd) > msg.payload.prayerTime.length)\n    {\n        msg.waktu_solat_hari_ini  = msg.payload.prayerTime[dd-1];\n\n    }\n    else \n    {\n    // if after 8PM, then we get tomorrow's time and update HA\n        msg.waktu_solat_hari_ini  = msg.payload.prayerTime[parseInt(dd)];\n\n    }\n    \n} \nelse {\n    // retrieve from array (day - 1)\n    msg.waktu_solat_hari_ini  = msg.payload.prayerTime[dd-1];\n}\n\n//remove seconds\nmsg.waktu_solat_hari_ini.imsak      = msg.waktu_solat_hari_ini.imsak.split(\":\").splice(0,2).join(\":\");\nmsg.waktu_solat_hari_ini.fajr       = msg.waktu_solat_hari_ini.fajr.split(\":\").splice(0,2).join(\":\");\nmsg.waktu_solat_hari_ini.syuruk     = msg.waktu_solat_hari_ini.syuruk.split(\":\").splice(0,2).join(\":\");\nmsg.waktu_solat_hari_ini.dhuhr      = msg.waktu_solat_hari_ini.dhuhr.split(\":\").splice(0,2).join(\":\"); \nmsg.waktu_solat_hari_ini.asr        = msg.waktu_solat_hari_ini.asr.split(\":\").splice(0,2).join(\":\");\nmsg.waktu_solat_hari_ini.maghrib    = msg.waktu_solat_hari_ini.maghrib.split(\":\").splice(0,2).join(\":\");\nmsg.waktu_solat_hari_ini.isha       = msg.waktu_solat_hari_ini.isha.split(\":\").splice(0,2).join(\":\");\n\n\n// since JAKIM data provides tarik hijri - we use this instead\nvar jakim_hijri         = msg.waktu_solat_hari_ini.hijri.split(\"-\");\nvar dict_bulan_islam    = ['Muharram', 'Safar', 'Rabi\\' ul-awwal', 'Rabi\\' ul-akhir', 'Jumadil-awal', 'Jumadil-akhir', 'Rejab', 'Sha\\'aban', 'Ramadan', 'Shawwal', 'Zulkaedah', 'Zulhijjah'];\nmsg.hijri_date          = jakim_hijri[2]+\" \"+dict_bulan_islam[jakim_hijri[1] - 1]+\" \"+jakim_hijri[0]+\" H\";\n\n\n// for reference in Telegram notice\nmsg.waktu_solat_api =  flow.get(\"waktu_solat_api\");\nmsg.waktu_solat_hari_ini.zone                   = msg.payload.zone;\nmsg.waktu_solat_hari_ini.data_download_date     = msg.payload.serverTime;\n\n// Store in Flow\nflow.set(\"zone\", msg.waktu_solat_hari_ini.zone);\nflow.set(\"data_download_date\", msg.waktu_solat_hari_ini.data_download_date);\nflow.set(\"waktu_imsak\", msg.waktu_solat_hari_ini.imsak);\nflow.set(\"waktu_subuh\", msg.waktu_solat_hari_ini.fajr);\nflow.set(\"waktu_syuruk\", msg.waktu_solat_hari_ini.syuruk);\nflow.set(\"waktu_zohor\", msg.waktu_solat_hari_ini.dhuhr);\nflow.set(\"waktu_asar\", msg.waktu_solat_hari_ini.asr);\nflow.set(\"waktu_maghrib\", msg.waktu_solat_hari_ini.maghrib);\nflow.set(\"waktu_isyak\", msg.waktu_solat_hari_ini.isha);\nflow.set(\"waktu_tarikh_hari\", msg.waktu_solat_hari_ini.day);\nflow.set(\"waktu_tarikh_hijri\", msg.waktu_solat_hari_ini.hijri);\nflow.set(\"waktu_tarikh_masihi\", msg.waktu_solat_hari_ini.date);\n\n//if manage to get this far meaning data is available.\n//filter topic = validated only to proceed and update solat registers\nmsg.topic = \"OK\"\n\nreturn msg;\n\n      ","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":860,"y":340,"wires":[["44fe01b1.351e6","89f062ef.b3e0c"]]},{"id":"cac2e18d.0473a","type":"comment","z":"979d82cfea624e5f","name":"Daily Lookup and Load into Flow","info":"### ## ### \n### ## ### additional nodes\n### ## ### \n### ## ### node-red-contrib-random-item \n# Via Cronjob","x":130,"y":240,"wires":[]},{"id":"a2e493b6.db17e","type":"inject","z":"979d82cfea624e5f","name":"","props":[{"p":"payload"},{"p":"topic","vt":"str"}],"repeat":"","crontab":"","once":true,"onceDelay":"1","topic":"","payload":"1","payloadType":"num","x":250,"y":300,"wires":[["eaabfff6.c4fca"]]},{"id":"eaabfff6.c4fca","type":"file in","z":"979d82cfea624e5f","name":"Retreive from File","filename":"/share/waktu_solat/waktu_solat_month.json","format":"utf8","chunk":false,"sendError":false,"encoding":"none","x":470,"y":340,"wires":[["14c50ab9.ffc0b5"]]},{"id":"14c50ab9.ffc0b5","type":"json","z":"979d82cfea624e5f","name":"","property":"payload","action":"","pretty":false,"x":650,"y":340,"wires":[["72b72987.9187c8"]]},{"id":"89f062ef.b3e0c","type":"debug","z":"979d82cfea624e5f","name":"harian","active":false,"tosidebar":true,"console":false,"tostatus":false,"complete":"true","targetType":"full","statusVal":"","statusType":"auto","x":810,"y":400,"wires":[]},{"id":"f4cefcd7.e6dc7","type":"delay","z":"979d82cfea624e5f","name":"","pauseType":"delay","timeout":"10","timeoutUnits":"seconds","rate":"1","nbRateUnits":"1","rateUnits":"second","randomFirst":"1","randomLast":"5","randomUnits":"seconds","drop":false,"x":275,"y":420,"wires":[["eaabfff6.c4fca"]],"l":false},{"id":"45a9b7f2.cb00b8","type":"link in","z":"979d82cfea624e5f","name":"i-trigger_solat_update","links":["add6f5c.7a71b08"],"x":215,"y":420,"wires":[["f4cefcd7.e6dc7"]]},{"id":"b3129f3.663f26","type":"cronplus","z":"979d82cfea624e5f","name":"Daily Refresh - 6 Hourly","outputField":"payload","timeZone":"","persistDynamic":false,"commandResponseMsgOutput":"output1","outputs":1,"options":[{"name":"schedule1","topic":"schedule1","payloadType":"default","payload":"","expressionType":"cron","expression":"0 0 1 * * ? *","location":"","offset":"0","solarType":"all","solarEvents":"sunrise,sunset"}],"x":130,"y":340,"wires":[["eaabfff6.c4fca"]]},{"id":"f14bd617.555d38","type":"link out","z":"979d82cfea624e5f","name":"T-Covid","links":["8718a1dd.5beb7"],"x":995,"y":520,"wires":[],"icon":"node-red-contrib-telegrambot/telegram.png"},{"id":"f1d9ff16.648c7","type":"template","z":"979d82cfea624e5f","name":"Telegram Text","field":"payload","fieldType":"msg","format":"handlebars","syntax":"mustache","template":"🕋 Waktu Solat bagi zon {{ waktu_solat_hari_ini.zone }} \nTarikh: {{ hijri_date }} \n{{ waktu_solat_hari_ini.date }}\n\n*Imsak*  : {{ waktu_solat_hari_ini.imsak}}\n*Subuh*  : {{ waktu_solat_hari_ini.fajr }}\n*Syuruk* : {{ waktu_solat_hari_ini.syuruk }}\n*Zohor*  : {{ waktu_solat_hari_ini.dhuhr }}\n*Asar*   : {{ waktu_solat_hari_ini.asr}}\n*Maghrib*: {{ waktu_solat_hari_ini.maghrib }}\n*Isyak*  : {{ waktu_solat_hari_ini.isha }}\n\nData download date: {{waktu_solat_hari_ini.data_download_date}}\n\n\n","output":"str","x":860,"y":520,"wires":[["f14bd617.555d38"]]},{"id":"1fa6ec3f.49e3a4","type":"switch","z":"979d82cfea624e5f","name":"","property":"telegramrequest","propertyType":"msg","rules":[{"t":"eq","v":"true","vt":"str"}],"checkall":"true","repair":false,"outputs":1,"x":710,"y":520,"wires":[["f1d9ff16.648c7"]]},{"id":"587e7b3c.00bad4","type":"telegram command","z":"979d82cfea624e5f","name":"/getwaktusolat","command":"/getwaktusolat","description":"","registercommand":false,"language":"","bot":"","strict":false,"hasresponse":true,"useregex":false,"removeregexcommand":false,"outputs":2,"x":150,"y":500,"wires":[["4d33a0ca.1a43a"],[]]},{"id":"71fe748c.fce81c","type":"link in","z":"979d82cfea624e5f","name":"T-BroadcastWaktuSolat","links":["f80100ac.269b"],"x":120,"y":580,"wires":[["4d33a0ca.1a43a"]],"l":true},{"id":"4d33a0ca.1a43a","type":"change","z":"979d82cfea624e5f","name":"Telegram flag","rules":[{"t":"set","p":"telegramrequest","pt":"msg","to":"true","tot":"str"}],"action":"","property":"","from":"","to":"","reg":false,"x":360,"y":540,"wires":[["eaabfff6.c4fca"]]},{"id":"780f0eb6.39ec5","type":"link out","z":"979d82cfea624e5f","name":"O-TelegramP1","links":["8718a1dd.5beb7"],"x":1175,"y":240,"wires":[],"icon":"node-red-contrib-telegrambot/telegram.png"},{"id":"44fe01b1.351e6","type":"switch","z":"979d82cfea624e5f","name":"QC","property":"topic","propertyType":"msg","rules":[{"t":"eq","v":"ERROR","vt":"str"},{"t":"eq","v":"OK","vt":"str"}],"checkall":"true","repair":false,"outputs":2,"x":1130,"y":340,"wires":[["defed156.ef28d"],["7a213032.e050d","78231aab.c50324","42ec6eeb.68c1e","4c277dfb.098854","86133ba6.a8d9f8","8475476f.84bdc8","ec295e23.a85c7","a6f28b9d.b65af8","c8023b05.eb95b8","ffbb9922.968248","1fa6ec3f.49e3a4"]]},{"id":"defed156.ef28d","type":"template","z":"979d82cfea624e5f","name":"Telegram Text","field":"payload","fieldType":"msg","format":"handlebars","syntax":"mustache","template":"⚠️⚠️⚠️🕋 Data waktu solat hari ini gagal ditetapkan. \n\nSebabnya {{payload}}","output":"str","x":1040,"y":280,"wires":[["780f0eb6.39ec5"]]},{"id":"621aaea6.50ac3","type":"group","z":"979d82cfea624e5f","style":{"stroke":"#2e333a","stroke-opacity":"1","fill":"#2e333a","fill-opacity":"0.75","label":true,"label-position":"nw","color":"#a4a4a4"},"nodes":["371ec913.5f6726","86133ba6.a8d9f8","4c277dfb.098854","a8edf8c6.0ac928","f8af1838.d32f78","8870a7b0.dc6df8","4a67e276.26637c","79962577.886d2c","8475476f.84bdc8","ec295e23.a85c7","a6f28b9d.b65af8","c8023b05.eb95b8","ffbb9922.968248","42ec6eeb.68c1e","6f51f1a4.9d5fb","78231aab.c50324","1b2f2219.a0563e","7a213032.e050d","8c7b13db.5de7c","5ca1cb56.9a8594"],"x":1364,"y":159,"w":532,"h":662},{"id":"371ec913.5f6726","type":"ui_text","z":"979d82cfea624e5f","g":"621aaea6.50ac3","group":"40d45c48.fc58f4","order":2,"width":0,"height":0,"name":"","label":"Subuh ","format":"{{msg.payload.prayer_times.subuh}}","layout":"row-spread","x":1750,"y":420,"wires":[]},{"id":"86133ba6.a8d9f8","type":"ha-entity","z":"979d82cfea624e5f","g":"621aaea6.50ac3","name":"Subuh_NodeRedInput","server":"","version":1,"debugenabled":false,"outputs":1,"entityType":"sensor","config":[{"property":"name","value":"Subuh"},{"property":"device_class","value":""},{"property":"icon","value":"mdi:islam"},{"property":"unit_of_measurement","value":""}],"state":"waktu_solat_hari_ini.fajr","stateType":"msg","attributes":[],"resend":true,"outputLocation":"","outputLocationType":"none","inputOverride":"allow","outputOnStateChange":false,"outputPayload":"$entity().state ? \"on\": \"off\"","outputPayloadType":"jsonata","x":1500,"y":420,"wires":[["371ec913.5f6726"]]},{"id":"4c277dfb.098854","type":"ha-entity","z":"979d82cfea624e5f","g":"621aaea6.50ac3","name":"WaktuSolat_NodeRedInput","server":"","version":1,"debugenabled":false,"outputs":1,"entityType":"sensor","config":[{"property":"name","value":"Waktu Solat"},{"property":"device_class","value":""},{"property":"icon","value":"mdi:islam"},{"property":"unit_of_measurement","value":""}],"state":"hijri_date","stateType":"msg","attributes":[{"property":"Subuh","value":"waktu_solat_hari_ini.fajr","valueType":"msg"},{"property":"Syuruk","value":"waktu_solat_hari_ini.syuruk","valueType":"msg"},{"property":"Zohor","value":"waktu_solat_hari_ini.dhuhr","valueType":"msg"},{"property":"Asar","value":"waktu_solat_hari_ini.asr","valueType":"msg"},{"property":"Maghrib","value":"waktu_solat_hari_ini.maghrib","valueType":"msg"},{"property":"Isyak","value":"waktu_solat_hari_ini.isha","valueType":"msg"},{"property":"Zone","value":"waktu_solat_hari_ini.zone","valueType":"msg"},{"property":"Tarikh","value":"waktu_solat_hari_ini.date","valueType":"msg"},{"property":"Data Downloaded Date","value":"waktu_solat_hari_ini.data_download_date","valueType":"msg"},{"property":"API Source","value":"waktu_solat_api","valueType":"msg"},{"property":"Imsak","value":"waktu_solat_hari_ini.imsak","valueType":"msg"}],"resend":true,"outputLocation":"","outputLocationType":"none","inputOverride":"allow","outputOnStateChange":false,"outputPayload":"$entity().state ? \"on\": \"off\"","outputPayloadType":"jsonata","x":1520,"y":360,"wires":[[]]},{"id":"a8edf8c6.0ac928","type":"ui_text","z":"979d82cfea624e5f","g":"621aaea6.50ac3","group":"40d45c48.fc58f4","order":3,"width":0,"height":0,"name":"","label":"Syuruk","format":"{{msg.payload.prayer_times.syuruk}}","layout":"row-spread","x":1740,"y":480,"wires":[]},{"id":"f8af1838.d32f78","type":"ui_text","z":"979d82cfea624e5f","g":"621aaea6.50ac3","group":"40d45c48.fc58f4","order":4,"width":0,"height":0,"name":"","label":"Zohor","format":"{{msg.payload.prayer_times.zohor}}","layout":"row-spread","x":1750,"y":540,"wires":[]},{"id":"8870a7b0.dc6df8","type":"ui_text","z":"979d82cfea624e5f","g":"621aaea6.50ac3","group":"40d45c48.fc58f4","order":5,"width":0,"height":0,"name":"","label":"Asar","format":"{{msg.payload.prayer_times.asar}}","layout":"row-spread","x":1750,"y":600,"wires":[]},{"id":"4a67e276.26637c","type":"ui_text","z":"979d82cfea624e5f","g":"621aaea6.50ac3","group":"40d45c48.fc58f4","order":6,"width":0,"height":0,"name":"","label":"Maghrib","format":"{{msg.payload.prayer_times.maghrib}}","layout":"row-spread","x":1740,"y":660,"wires":[]},{"id":"79962577.886d2c","type":"ui_text","z":"979d82cfea624e5f","g":"621aaea6.50ac3","group":"40d45c48.fc58f4","order":7,"width":0,"height":0,"name":"","label":"Isyak","format":"{{msg.payload.prayer_times.isyak}}","layout":"row-spread","x":1750,"y":720,"wires":[]},{"id":"8475476f.84bdc8","type":"ha-entity","z":"979d82cfea624e5f","g":"621aaea6.50ac3","name":"Syuruk_NodeRedInput","server":"","version":1,"debugenabled":false,"outputs":1,"entityType":"sensor","config":[{"property":"name","value":"Syuruk"},{"property":"device_class","value":""},{"property":"icon","value":"mdi:islam"},{"property":"unit_of_measurement","value":""}],"state":"waktu_solat_hari_ini.syuruk","stateType":"msg","attributes":[],"resend":true,"outputLocation":"","outputLocationType":"none","inputOverride":"allow","outputOnStateChange":false,"outputPayload":"$entity().state ? \"on\": \"off\"","outputPayloadType":"jsonata","x":1500,"y":480,"wires":[["a8edf8c6.0ac928"]]},{"id":"ec295e23.a85c7","type":"ha-entity","z":"979d82cfea624e5f","g":"621aaea6.50ac3","name":"Zohor_NodeRedInput","server":"","version":1,"debugenabled":false,"outputs":1,"entityType":"sensor","config":[{"property":"name","value":"Zohor"},{"property":"device_class","value":""},{"property":"icon","value":"mdi:islam"},{"property":"unit_of_measurement","value":""}],"state":"waktu_solat_hari_ini.dhuhr","stateType":"msg","attributes":[],"resend":true,"outputLocation":"","outputLocationType":"none","inputOverride":"allow","outputOnStateChange":false,"outputPayload":"$entity().state ? \"on\": \"off\"","outputPayloadType":"jsonata","x":1500,"y":540,"wires":[["f8af1838.d32f78"]]},{"id":"a6f28b9d.b65af8","type":"ha-entity","z":"979d82cfea624e5f","g":"621aaea6.50ac3","name":"Asar_NodeRedInput","server":"","version":1,"debugenabled":false,"outputs":1,"entityType":"sensor","config":[{"property":"name","value":"Asar"},{"property":"device_class","value":""},{"property":"icon","value":"mdi:islam"},{"property":"unit_of_measurement","value":""}],"state":"waktu_solat_hari_ini.asr","stateType":"msg","attributes":[],"resend":true,"outputLocation":"","outputLocationType":"none","inputOverride":"allow","outputOnStateChange":false,"outputPayload":"$entity().state ? \"on\": \"off\"","outputPayloadType":"jsonata","x":1500,"y":600,"wires":[["8870a7b0.dc6df8"]]},{"id":"c8023b05.eb95b8","type":"ha-entity","z":"979d82cfea624e5f","g":"621aaea6.50ac3","name":"Maghrib_NodeRedInput","server":"","version":1,"debugenabled":false,"outputs":1,"entityType":"sensor","config":[{"property":"name","value":"Maghrib"},{"property":"device_class","value":""},{"property":"icon","value":"mdi:islam"},{"property":"unit_of_measurement","value":""}],"state":"waktu_solat_hari_ini.maghrib","stateType":"msg","attributes":[],"resend":true,"outputLocation":"","outputLocationType":"none","inputOverride":"allow","outputOnStateChange":false,"outputPayload":"$entity().state ? \"on\": \"off\"","outputPayloadType":"jsonata","x":1510,"y":660,"wires":[["4a67e276.26637c"]]},{"id":"ffbb9922.968248","type":"ha-entity","z":"979d82cfea624e5f","g":"621aaea6.50ac3","name":"Isyak_NodeRedInput","server":"","version":1,"debugenabled":false,"outputs":1,"entityType":"sensor","config":[{"property":"name","value":"Isyak"},{"property":"device_class","value":""},{"property":"icon","value":"mdi:islam"},{"property":"unit_of_measurement","value":""}],"state":"waktu_solat_hari_ini.isha","stateType":"msg","attributes":[],"resend":true,"outputLocation":"","outputLocationType":"none","inputOverride":"allow","outputOnStateChange":false,"outputPayload":"$entity().state ? \"on\": \"off\"","outputPayloadType":"jsonata","x":1500,"y":720,"wires":[["79962577.886d2c"]]},{"id":"42ec6eeb.68c1e","type":"ha-entity","z":"979d82cfea624e5f","g":"621aaea6.50ac3","name":"ZoneWaktuSolat_NodeRedInput","server":"","version":1,"debugenabled":false,"outputs":1,"entityType":"sensor","config":[{"property":"name","value":"Zone Waktu Solat"},{"property":"device_class","value":""},{"property":"icon","value":"mdi:home-map-marker"},{"property":"unit_of_measurement","value":""}],"state":"waktu_solat_hari_ini.zone","stateType":"msg","attributes":[],"resend":true,"outputLocation":"","outputLocationType":"none","inputOverride":"allow","outputOnStateChange":false,"outputPayload":"$entity().state ? \"on\": \"off\"","outputPayloadType":"jsonata","x":1530,"y":300,"wires":[["6f51f1a4.9d5fb"]]},{"id":"6f51f1a4.9d5fb","type":"ui_text","z":"979d82cfea624e5f","g":"621aaea6.50ac3","group":"a0647b57.fedc08","order":1,"width":0,"height":0,"name":"","label":"Zone","format":"{{msg.payload.zone}}","layout":"row-spread","x":1750,"y":300,"wires":[]},{"id":"78231aab.c50324","type":"ha-entity","z":"979d82cfea624e5f","g":"621aaea6.50ac3","name":"TarikhWaktuSolat_NodeRedInput","server":"","version":1,"debugenabled":false,"outputs":1,"entityType":"sensor","config":[{"property":"name","value":"Tarikh Waktu Solat"},{"property":"device_class","value":""},{"property":"icon","value":"mdi:calendar"},{"property":"unit_of_measurement","value":""}],"state":"waktu_solat_hari_ini.date","stateType":"msg","attributes":[],"resend":false,"outputLocation":"","outputLocationType":"none","inputOverride":"allow","outputOnStateChange":false,"outputPayload":"$entity().state ? \"on\": \"off\"","outputPayloadType":"jsonata","x":1540,"y":240,"wires":[["1b2f2219.a0563e"]]},{"id":"1b2f2219.a0563e","type":"ui_text","z":"979d82cfea624e5f","g":"621aaea6.50ac3","group":"a0647b57.fedc08","order":1,"width":0,"height":0,"name":"","label":"Date ","format":"{{msg.payload.prayer_times.date}}","layout":"row-spread","x":1750,"y":240,"wires":[]},{"id":"7a213032.e050d","type":"ui_text","z":"979d82cfea624e5f","g":"621aaea6.50ac3","group":"a0647b57.fedc08","order":1,"width":0,"height":0,"name":"","label":"Datestamp","format":"{{msg.payload.prayer_times.datestamp}}","layout":"row-spread","x":1470,"y":200,"wires":[]},{"id":"8c7b13db.5de7c","type":"ui_text","z":"979d82cfea624e5f","g":"621aaea6.50ac3","group":"a0647b57.fedc08","order":5,"width":0,"height":0,"name":"","label":"HASS Current Time","format":"{{msg.payload}}","layout":"row-spread","x":1790,"y":780,"wires":[]},{"id":"5ca1cb56.9a8594","type":"server-state-changed","z":"979d82cfea624e5f","g":"621aaea6.50ac3","name":"time now","server":"","version":3,"exposeToHomeAssistant":false,"haConfig":[{"property":"name","value":""},{"property":"icon","value":""}],"entityidfilter":"sensor.time","entityidfiltertype":"exact","outputinitially":true,"state_type":"str","haltifstate":"","halt_if_type":"num","halt_if_compare":"is","outputs":1,"output_only_on_state_change":true,"for":"0","forType":"num","forUnits":"minutes","ignorePrevStateNull":false,"ignorePrevStateUnknown":false,"ignorePrevStateUnavailable":false,"ignoreCurrentStateUnknown":false,"ignoreCurrentStateUnavailable":false,"outputProperties":[{"property":"payload","propertyType":"msg","value":"","valueType":"entityState"}],"x":1460,"y":780,"wires":[["8c7b13db.5de7c"]]}]
```







### Bahagian 3 - Bandingkan waktu terkini dan waktu Solat

![Part2 Flow](https://github.com/anas-ivs/HA-NR-WaktuSolatJakim/blob/main/images/Flow_part2.PNG)
1.  Setiap minit, melalui `inject`  - node `function - Compare times` akan dijalankan . Blok ini akan:
    1.  *Load* Waktu Solat melalui `Flow Node-Red Context` yang telah disimpan sebelum ini. 
    2.  Mengambil tarikh dan jam terkini dari system. 
    3.  Menentukan tempoh Waktu solat sekarang, Waktu solat berikutnya dan tempoh masa yang tinggal.
    4.  Menentukan jika sudah masuk Waktu Solat.
2.  Setiap minit - jika belum masuk (5+2) Waktu Solat ; output function block ini akan mengemaskini tempoh masa terkini ke solat seterusnya.
3.  Jika sudah masuk Solat - Waktu solat akan ditetapkan melalui `msg.waktu_solat`.
4.  Node `switch` digunakan untuk membuat tetapan spesifik yang hendak dibuat bagi setiap (5+2 : Imsak, Syuruk) Waktu Solat.

![](C:\Users\user\Documents\GitHub\HA-NR-WaktuSolatJakim\images\Step3-Setiap Minit_cmp.PNG)

```json
[
    {
        "id": "7a8ff58b.91c62c",
        "type": "group",
        "z": "979d82cfea624e5f",
        "style": {
            "stroke": "#2e333a",
            "stroke-opacity": "1",
            "fill": "#2e333a",
            "fill-opacity": "0.75",
            "label": true,
            "label-position": "nw",
            "color": "#a4a4a4"
        },
        "nodes": [
            "1c81239f.dfd61c",
            "fda63eeb.b7931",
            "e79ae7ad.22ab78",
            "2dee4275.f2107e",
            "cdc3643e.25f498",
            "208d52d.827ddae",
            "1b55833e.01921d",
            "124eef07.c54df1",
            "728a09a9.c18338",
            "746147b6.de9f68",
            "8b88caa3.9776d8",
            "29ac26b3.c2dc6a",
            "edaa6b5e.0aa918",
            "e9044ba9.d82468",
            "db1127cc.ff45f8",
            "799108f0.41bb28",
            "a6f0b09e.7dfd2",
            "277ffb75.6e3b34",
            "4945c457.ae3a2c",
            "4541b1d8.57bc5",
            "73dbc05.7ae684",
            "e7de13f9.4c83f",
            "b82459be.056f48",
            "511b9079.f8fcb",
            "ea4e5db9.99087",
            "8f54ce9.8394b3",
            "c25108a8.993cc8",
            "e9044ba9.d82468"
        ],
        "x": 174,
        "y": 79,
        "w": 1702,
        "h": 442
    },
    {
        "id": "1c81239f.dfd61c",
        "type": "switch",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "Solat Spesific Commands",
        "property": "waktu_solat",
        "propertyType": "msg",
        "rules": [
            {
                "t": "eq",
                "v": "Subuh",
                "vt": "str"
            },
            {
                "t": "eq",
                "v": "Syuruk",
                "vt": "str"
            },
            {
                "t": "eq",
                "v": "Zohor",
                "vt": "str"
            },
            {
                "t": "eq",
                "v": "Asar",
                "vt": "str"
            },
            {
                "t": "eq",
                "v": "Maghrib",
                "vt": "str"
            },
            {
                "t": "eq",
                "v": "Isyak",
                "vt": "str"
            }
        ],
        "checkall": "true",
        "repair": false,
        "outputs": 6,
        "x": 390,
        "y": 440,
        "wires": [
            [
                "fda63eeb.b7931",
                "2dee4275.f2107e"
            ],
            [
                "e79ae7ad.22ab78"
            ],
            [
                "fda63eeb.b7931"
            ],
            [
                "fda63eeb.b7931"
            ],
            [
                "fda63eeb.b7931",
                "1b55833e.01921d"
            ],
            [
                "fda63eeb.b7931"
            ]
        ]
    },
    {
        "id": "fda63eeb.b7931",
        "type": "change",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "Craft Message",
        "rules": [
            {
                "t": "set",
                "p": "payload",
                "pt": "msg",
                "to": "\"🕌 Kini telah masuk waktu solat \" & $.waktu_solat",
                "tot": "jsonata"
            },
            {
                "t": "set",
                "p": "data",
                "pt": "msg",
                "to": "$.payload ",
                "tot": "jsonata"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 700,
        "y": 320,
        "wires": [
            [
                "e9044ba9.d82468",
                "799108f0.41bb28",
                "a6f0b09e.7dfd2",
                "b82459be.056f48"
            ]
        ]
    },
    {
        "id": "e79ae7ad.22ab78",
        "type": "link out",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "Syuruk",
        "links": [
            "44aaf8a8.5112c8"
        ],
        "x": 770,
        "y": 440,
        "wires": [],
        "l": true
    },
    {
        "id": "2dee4275.f2107e",
        "type": "link out",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "Subuh",
        "links": [
            "3c0048b4.36eee8"
        ],
        "x": 770,
        "y": 400,
        "wires": [],
        "l": true
    },
    {
        "id": "cdc3643e.25f498",
        "type": "link out",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "T-Waktu Solat",
        "links": [
            "8718a1dd.5beb7"
        ],
        "x": 1215,
        "y": 200,
        "wires": [],
        "icon": "node-red-contrib-telegrambot/telegram.png"
    },
    {
        "id": "208d52d.827ddae",
        "type": "link out",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "WS Speaker Out - TTS",
        "links": [
            "c4134f1d.de9a5"
        ],
        "x": 1215,
        "y": 320,
        "wires": [],
        "icon": "node-red-contrib-cast/google-home-mini2.svg"
    },
    {
        "id": "1b55833e.01921d",
        "type": "link out",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "Maghrib",
        "links": [
            "8a34191d.af0b48",
            "b674ddb2.4b5f7",
            "dd2fe950.30acf8"
        ],
        "x": 780,
        "y": 480,
        "wires": [],
        "l": true
    },
    {
        "id": "124eef07.c54df1",
        "type": "delay",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "",
        "pauseType": "delay",
        "timeout": "45",
        "timeoutUnits": "seconds",
        "rate": "1",
        "nbRateUnits": "1",
        "rateUnits": "second",
        "randomFirst": "1",
        "randomLast": "5",
        "randomUnits": "seconds",
        "drop": false,
        "x": 1260,
        "y": 380,
        "wires": [
            [
                "728a09a9.c18338"
            ]
        ]
    },
    {
        "id": "728a09a9.c18338",
        "type": "api-call-service",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "TTS",
        "server": "",
        "version": 3,
        "debugenabled": false,
        "service_domain": "tts",
        "service": "google_translate_say",
        "entityId": "media_player.chromecast",
        "data": "{\"message\":\"{{payload}}\",\"cache\":\"true\",\"language\":\"id\"}",
        "dataType": "json",
        "mergecontext": "",
        "mustacheAltTags": false,
        "outputProperties": [
            {
                "property": "payload",
                "propertyType": "msg",
                "value": "payload",
                "valueType": "msg"
            }
        ],
        "queue": "none",
        "x": 1410,
        "y": 380,
        "wires": [
            []
        ],
        "icon": "node-red-contrib-cast/google-home-mini1.svg"
    },
    {
        "id": "746147b6.de9f68",
        "type": "debug",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "SolatTimes",
        "active": false,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "true",
        "targetType": "full",
        "statusVal": "",
        "statusType": "auto",
        "x": 670,
        "y": 200,
        "wires": []
    },
    {
        "id": "8b88caa3.9776d8",
        "type": "ha-entity",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "Time to next prayer",
        "server": "",
        "version": 1,
        "debugenabled": false,
        "outputs": 1,
        "entityType": "sensor",
        "config": [
            {
                "property": "name",
                "value": "solat_bakiwaktu"
            },
            {
                "property": "device_class",
                "value": ""
            },
            {
                "property": "icon",
                "value": "mdi:islam"
            },
            {
                "property": "unit_of_measurement",
                "value": ""
            }
        ],
        "state": "time_waktuberikut",
        "stateType": "msg",
        "attributes": [
            {
                "property": "Waktu Solat Sekarang",
                "value": "waktu_solat_sekarang",
                "valueType": "msg"
            },
            {
                "property": "Waktu Solat Berikut",
                "value": "waktu_solat_berikut",
                "valueType": "msg"
            }
        ],
        "resend": true,
        "outputLocation": "",
        "outputLocationType": "none",
        "inputOverride": "allow",
        "outputOnStateChange": false,
        "outputPayload": "$entity().state ? \"on\": \"off\"",
        "outputPayloadType": "jsonata",
        "x": 690,
        "y": 260,
        "wires": [
            []
        ]
    },
    {
        "id": "29ac26b3.c2dc6a",
        "type": "function",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "Youtube/Azan Random",
        "func": "let playlist_random = getRndInteger(1,6);\n\nnode.status({fill:\"green\",shape:\"ring\",text:\" Playlist: \"+playlist_random}); \n\nswitch (playlist_random) {\n    case(1):\n        videoid = \"6IyJWdsbbYs\";\n        break;\n    case(2):\n        videoid = \"T7s3IFMktLo\";\n        break;\n    case(3):\n        videoid = \"vXQhE2CMhhM\";\n        break;\n    case(4):\n        videoid = \"my-IGBTNnYE\";\n        break;\n    case(5):\n        videoid = \"DdC7R3s7eCY\";\n        break\n    case(6):\n        videoid = \"z2xEwSi2vaI\";\n        break\n    default:\n        videoid = \"uwXEOccuRyU\";\n}\n\nmsg.payload = {\n\"app\": \"YouTube\",\n\"type\": \"MEDIA\",\n\"videoId\": videoid\n};\n\nreturn msg;\n\n// this code gets a random interger\nfunction getRndInteger(min, max) {\n    return Math.floor(Math.random() * (max - min + 1) ) + min;\n}",
        "outputs": 1,
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "libs": [],
        "x": 1500,
        "y": 240,
        "wires": [
            [
                "db1127cc.ff45f8"
            ]
        ]
    },
    {
        "id": "edaa6b5e.0aa918",
        "type": "function",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "Youtube/Azan Subuh",
        "func": "let playlist_random = getRndInteger(1,6);\n\nnode.status({fill:\"green\",shape:\"ring\",text:\" Playlist: \"+playlist_random}); \n\nswitch (playlist_random) {\n    case(1):\n        videoid = \"qhp3gy2rDUU\";\n        break;\n    case(2):\n        videoid = \"kYgg0IW4Cpk\";\n        break;\n    case(3):\n        videoid = \"kutazqNu0OU\";\n        break;\n    case(4):\n        videoid = \"FTFyP-p3VTI\";\n        break;\n    case(5):\n        videoid = \"C5GaDD2gAqU\";\n        break\n    case(6):\n        videoid = \"pVi8UTeKAso\";\n        break\n    default:\n        videoid = \"FTFyP-p3VTI\";\n}\n\nmsg.payload = {\n\"app\": \"YouTube\",\n\"type\": \"MEDIA\",\n\"videoId\": videoid\n};\n\nreturn msg;\n\n// this code gets a random interger\nfunction getRndInteger(min, max) {\n    return Math.floor(Math.random() * (max - min + 1) ) + min;\n}",
        "outputs": 1,
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "libs": [],
        "x": 1500,
        "y": 180,
        "wires": [
            [
                "db1127cc.ff45f8"
            ]
        ]
    },
    {
        "id": "e9044ba9.d82468",
        "type": "wake on lan",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "mac": "20:17:42:d8:4e:8f",
        "host": "192.168.0.255",
        "udpport": 9,
        "name": "LGTV",
        "x": 1250,
        "y": 200,
        "wires": []
    },
    {
        "id": "db1127cc.ff45f8",
        "type": "link out",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "out-googletvchromecast",
        "links": [
            "56211c60.08f6c4",
            "cf92c33d.6c7d5"
        ],
        "x": 1835,
        "y": 260,
        "wires": []
    },
    {
        "id": "799108f0.41bb28",
        "type": "api-call-service",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "Turn ON",
        "server": "",
        "version": 3,
        "debugenabled": false,
        "service_domain": "media_player",
        "service": "turn_on",
        "entityId": "media_player.chromecast",
        "data": "",
        "dataType": "jsonata",
        "mergecontext": "",
        "mustacheAltTags": false,
        "outputProperties": [],
        "queue": "none",
        "x": 1040,
        "y": 380,
        "wires": [
            [
                "124eef07.c54df1"
            ]
        ]
    },
    {
        "id": "a6f0b09e.7dfd2",
        "type": "delay",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "",
        "pauseType": "delay",
        "timeout": "60",
        "timeoutUnits": "seconds",
        "rate": "1",
        "nbRateUnits": "1",
        "rateUnits": "second",
        "randomFirst": "1",
        "randomLast": "5",
        "randomUnits": "seconds",
        "drop": false,
        "x": 1040,
        "y": 240,
        "wires": [
            [
                "cdc3643e.25f498",
                "c25108a8.993cc8"
            ]
        ]
    },
    {
        "id": "277ffb75.6e3b34",
        "type": "function",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "Compare Times",
        "func": "\n// construct newmsg\nnewmsg = {};\n\n// retrieve waktu solat from flow\nlet zone                = flow.get(\"zone\");\nlet data_download_date  = flow.get(\"data_download_date\");\nlet waktu_imsak         = flow.get(\"waktu_imsak\");\nlet subuhtime           = flow.get(\"waktu_subuh\");\nlet syuruktime          = flow.get(\"waktu_syuruk\");\nlet zohortime           = flow.get(\"waktu_zohor\");\nlet asartime            = flow.get(\"waktu_asar\");\nlet maghribtime         = flow.get(\"waktu_maghrib\");\nlet isyaktime           = flow.get(\"waktu_isyak\");\n\n// define other timing variables \n// PREFERENCE: add 1 minute with delay timer later to get exact azan at the minute vs. delayed\nvar minutesToAdd=1;\n\nconst now = new Date();\nconst hours = now.getHours().toString().padStart(2,'0');\nconst minutes = now.getMinutes().toString().padStart(2,'0');\n\nconst day = now.getDate();\nconst month = now.getMonth();\nconst year = now.getFullYear();\n\nconst compared_minute = new Date(now.getTime() + minutesToAdd*60000);\nconst compared_hours = compared_minute.getHours().toString().padStart(2,'0');\nconst compared_minutes = compared_minute.getMinutes().toString().padStart(2,'0');\n\n// used time for comparing later\nconst time = `${compared_hours}:${compared_minutes}`;\n\nif (time > subuhtime && time < syuruktime)\n{\n     flow.set(\"waktusolatsekarang\", \"Subuh\");\n     flow.set(\"waktusolatberikut\", \"Syuruk\");  \n     flow.set(\"time_waktusolatberikut\", syuruktime);  \n}\nelse if (time > syuruktime && time < zohortime)\n{\n    flow.set(\"waktusolatsekarang\", \"Syuruk\");\n    flow.set(\"waktusolatberikut\", \"Zohor\"); \n    flow.set(\"time_waktusolatberikut\", zohortime);  \n}\nelse if (time > zohortime && time < asartime)\n{\n    flow.set(\"waktusolatsekarang\", \"Zohor\");\n    flow.set(\"waktusolatberikut\", \"Asar\"); \n    flow.set(\"time_waktusolatberikut\", asartime);  \n}    \nelse if (time > asartime && time < maghribtime)\n{\n    flow.set(\"waktusolatsekarang\", \"Asar\");\n    flow.set(\"waktusolatberikut\", \"Maghrib\"); \n    flow.set(\"time_waktusolatberikut\", maghribtime);  \n}        \nelse if (time > maghribtime && time < isyaktime)\n{\n    flow.set(\"waktusolatsekarang\", \"Maghrib\");\n    flow.set(\"waktusolatberikut\", \"Isyak\"); \n    flow.set(\"time_waktusolatberikut\", isyaktime);  \n}     \nelse if (time > isyaktime || time < subuhtime)\n{\n    flow.set(\"waktusolatsekarang\", \"Isyak\");\n    flow.set(\"waktusolatberikut\", \"Subuh\"); \n    flow.set(\"time_waktusolatberikut\", subuhtime);  \n} \n// else \n// {\n//     flow.set(\"waktusolatsekarang\", \"unknown\");\n//     flow.set(\"waktusolatberikut\", \"unknown\"); \n//     flow.set(\"time_waktusolatberikut\", time); \n    \n// }\n  \n\nlet waktu_solat_sekarang = flow.get(\"waktusolatsekarang\");\nlet waktu_solat_berikut = flow.get(\"waktusolatberikut\");\nlet time_waktusolat_berikut = flow.get(\"time_waktusolatberikut\");\n\n// compare time to trigger for waktu solat\nif (time === subuhtime) \n{\n    newmsg.payload = \"Subuh\";\n    flow.set(\"waktusolatsekarang\", \"Subuh\");\n    flow.set(\"waktusolatberikut\", \"Syuruk\");   \n    flow.set(\"time_waktusolatberikut\", syuruktime);\n\n} else if (time  === syuruktime) \n{\n    newmsg.payload = \"Syuruk\";\n    flow.set(\"waktusolatsekarang\", \"Syuruk\");\n    flow.set(\"waktusolatberikut\", \"Zohor\");   \n    flow.set(\"time_waktusolatberikut\", zohortime);\n} else if (time  === zohortime) \n{\n    newmsg.payload = \"Zohor\";\n    flow.set(\"waktusolatsekarang\", \"Zohor\");\n    flow.set(\"waktusolatberikut\", \"Asar\");   \n    flow.set(\"time_waktusolatberikut\", asartime);    \n} else if (time  === asartime) \n{\n    newmsg.payload = \"Asar\";\n    flow.set(\"waktusolatsekarang\", \"Asar\");\n    flow.set(\"waktusolatberikut\", \"Maghrib\");   \n    flow.set(\"time_waktusolatberikut\", maghribtime);\n} else if (time  === maghribtime) \n{\n    newmsg.payload = \"Maghrib\";\n    flow.set(\"waktusolatsekarang\", \"Maghrib\");\n    flow.set(\"waktusolatberikut\", \"Isyak\");   \n    flow.set(\"time_waktusolatberikut\", isyaktime);\n} else if (time  === isyaktime) \n{\n    newmsg.payload = \"Isyak\";\n    flow.set(\"waktusolatsekarang\", \"Isyaka\");\n    flow.set(\"waktusolatberikut\", \"Subuha\");   \n    flow.set(\"time_waktusolatberikut\", isyaktime);\n} \nelse \n{  \n    //set payload empty so following flow does not trigger\n    newmsg.payload = \"\";\n}\n\n// Calculate remaining time (FUTURE - convert to function?)\n// ________________________________________________________\n// retrieve waktu solat set in flow context (from previous runs or updated above)\n\n//time_waktusolatberikut = time_waktusolatberikut;\n\n// retrieve next solat time hour and minutes\nconst ns_time_hour  = time_waktusolat_berikut.substring(0,2);\nconst ns_time_minutes = time_waktusolat_berikut.substring(3,5);\n\n// construct new date using current time and next solat time\nconst current_time  = new Date(year,month,day,hours,minutes);\nconst nextsolat_time  = new Date(year,month,day,ns_time_hour,ns_time_minutes);\n\n// establish next day midnight/day/month/year for Subuh calculation\nvar midnight_time   = now;\nmidnight_time.setHours(24,0,0,0);\nconst ns_next_day   = midnight_time.getDate();\nconst ns_next_month = midnight_time.getMonth();\nconst ns_next_year  = midnight_time.getFullYear();\n\n// Check if current time is before midnight and Subuh is next\n\n// at midnight current hour = 0\n// hence at midnight this should not run, and isya no earlier than 1800\nif ( waktu_solat_berikut == \"Subuh\" && hours > 18  )\n{\n    // then we calculate using time difference for next day time\n    var subuh_time      = new Date(ns_next_year,ns_next_month,ns_next_day,ns_time_hour,ns_time_minutes);\n    newmsg.time_waktuberikut = msConversion(subuh_time-current_time);\n}\nelse {\n    // else (for subuh) we expect current day is equal, after midnight so day is the same.\n    // for other prayers - it would be the same.\n    newmsg.time_waktuberikut = msConversion(nextsolat_time-current_time);\n}\n\n// used for later flows as payload will be replaced.\nnewmsg.waktu_solat = newmsg.payload;\nnewmsg.waktu_solat_sekarang = waktu_solat_sekarang;\nnewmsg.waktu_solat_berikut = waktu_solat_berikut;\n\n//for debugging\nnewmsg.time = time;\nnewmsg.currentnow = now;\nnewmsg.current_hour = hours;\nnewmsg.current_time = current_time;\nnewmsg.current_time_locale = current_time.toLocaleString();\nnewmsg.nextsolat_time = nextsolat_time.toLocaleString();\nnewmsg.midnight_time = midnight_time.toLocaleString();\n\n\nreturn newmsg;\n\nfunction msConversion(millis) {\n  let sec = Math.floor(millis / 1000);\n  let hrs = Math.floor(sec / 3600);\n  sec -= hrs * 3600;\n  let min = Math.floor(sec / 60);\n  sec -= min * 60;\n\n  sec = '' + sec;\n  sec = ('00' + sec).substring(sec.length);\n\n  if (hrs > 0) {\n    min = '' + min;\n    min = ('00' + min).substring(min.length);\n    return hrs + \" jam \" + min + \" minit\";\n  }\n  else {\n    return min + \" minit\";\n  }\n}",
        "outputs": 1,
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "libs": [],
        "x": 440,
        "y": 200,
        "wires": [
            [
                "746147b6.de9f68",
                "8b88caa3.9776d8",
                "1c81239f.dfd61c"
            ]
        ]
    },
    {
        "id": "4945c457.ae3a2c",
        "type": "inject",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "",
        "props": [
            {
                "p": "payload"
            },
            {
                "p": "topic",
                "vt": "str"
            }
        ],
        "repeat": "60",
        "crontab": "",
        "once": false,
        "onceDelay": "",
        "topic": "",
        "payload": "1",
        "payloadType": "num",
        "x": 510,
        "y": 120,
        "wires": [
            [
                "e7de13f9.4c83f"
            ]
        ]
    },
    {
        "id": "4541b1d8.57bc5",
        "type": "comment",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "Minute by minute lookup",
        "info": "### ## ### \n### ## ### additional nodes\n### ## ### \n### ## ### node-red-contrib-random-item \n# Via Cronjob",
        "x": 310,
        "y": 120,
        "wires": []
    },
    {
        "id": "73dbc05.7ae684",
        "type": "link in",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "i-minute-pulse",
        "links": [
            "e7de13f9.4c83f"
        ],
        "x": 225,
        "y": 200,
        "wires": [
            [
                "277ffb75.6e3b34"
            ]
        ]
    },
    {
        "id": "e7de13f9.4c83f",
        "type": "link out",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "o-minute-pulse",
        "links": [
            "73dbc05.7ae684",
            "d659ebad.1d80a8"
        ],
        "x": 615,
        "y": 120,
        "wires": []
    },
    {
        "id": "b82459be.056f48",
        "type": "delay",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "",
        "pauseType": "delay",
        "timeout": "50",
        "timeoutUnits": "seconds",
        "rate": "1",
        "nbRateUnits": "1",
        "rateUnits": "second",
        "randomFirst": "1",
        "randomLast": "5",
        "randomUnits": "seconds",
        "drop": false,
        "x": 1040,
        "y": 320,
        "wires": [
            [
                "208d52d.827ddae"
            ]
        ]
    },
    {
        "id": "511b9079.f8fcb",
        "type": "function",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "Audio/Azan Subuh",
        "func": "\nmsg.payload = \"azansubuh.mp3\";\nreturn msg;",
        "outputs": 1,
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "libs": [],
        "x": 1490,
        "y": 120,
        "wires": [
            [
                "8f54ce9.8394b3"
            ]
        ]
    },
    {
        "id": "ea4e5db9.99087",
        "type": "function",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "Audio/Azan Random",
        "func": "let playlist_random = getRndInteger(1,4);\n\nnode.status({fill:\"green\",shape:\"ring\",text:\" Playlist: \"+playlist_random}); \n\nswitch (playlist_random) {\n    case(1):\n        mp3_id = \"azan_malaysia_tv3.mp3\";\n        break;\n    case(2):\n        mp3_id = \"azan_malaysia_shahalam.mp3\";\n        break;\n    case(3):\n        mp3_id = \"azan_misyari_rasyid_1.mp3\";\n        break;\n    case(4):\n        mp3_id = \"azan_misyari_rasyid_2.mp3\";\n        break;\n    default:\n        mp3_id = \"azan_malaysia_tv3.mp3\";\n}\n\nmsg.payload = mp3_id;\nreturn msg;\n\n// this code gets a random interger\nfunction getRndInteger(min, max) {\n    return Math.floor(Math.random() * (max - min + 1) ) + min;\n}\n\n",
        "outputs": 1,
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "libs": [],
        "x": 1500,
        "y": 300,
        "wires": [
            [
                "8f54ce9.8394b3"
            ]
        ]
    },
    {
        "id": "8f54ce9.8394b3",
        "type": "link out",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "WS Speaker Out - Audio/MP3",
        "links": [
            "b08e9a29.a14828"
        ],
        "x": 1835,
        "y": 200,
        "wires": [],
        "icon": "node-red-contrib-cast/google-home-mini2.svg"
    },
    {
        "id": "c25108a8.993cc8",
        "type": "switch",
        "z": "979d82cfea624e5f",
        "g": "7a8ff58b.91c62c",
        "name": "",
        "property": "waktu_solat",
        "propertyType": "msg",
        "rules": [
            {
                "t": "eq",
                "v": "Subuh",
                "vt": "str"
            },
            {
                "t": "neq",
                "v": "Subuh",
                "vt": "str"
            }
        ],
        "checkall": "true",
        "repair": false,
        "outputs": 2,
        "x": 1250,
        "y": 280,
        "wires": [
            [
                "edaa6b5e.0aa918",
                "511b9079.f8fcb"
            ],
            [
                "29ac26b3.c2dc6a",
                "ea4e5db9.99087"
            ]
        ]
    }
]
```

### Bahagian 4 - Tambahan - Notifikasi 15 minit sebelum masuk waktu.
![Part2 Flow](https://github.com/anas-ivs/HA-NR-WaktuSolatJakim/blob/main/images/Flow_part3.PNG)

1.  Bahagian ini mirip fungsi perbandingan waktu untuk menentukan Waktu Solat. 
2.  Sebaliknya - jam diawalkan sebanyak 15 minit agar perbandingan waktu dibuat 15 minit sebelum masuknya Waktu Solat tersebut.
3.  Ini membolehkan notifikasi/automation yang berasignan dibuat berbanding jika sudah masuk waktu - misalnya tutup langsir sebelum Maghrib atau memainkan Youtube Live video Makkah/Madinah di TV.

![](C:\Users\user\Documents\GitHub\HA-NR-WaktuSolatJakim\images\Step3-Setiap Minit_cmp.PNG)

```json [
    {
        "id": "98788efd.44a6e",
        "type": "group",
        "z": "979d82cfea624e5f",
        "style": {
            "stroke": "#2e333a",
            "stroke-opacity": "1",
            "fill": "#2e333a",
            "fill-opacity": "0.75",
            "label": true,
            "label-position": "nw",
            "color": "#a4a4a4"
        },
        "nodes": [
            "74e66f38.426cb",
            "fcf14654.85b548",
            "c0e5a10a.1b4a1",
            "b34fbd1d.58cc8",
            "1e72e30b.c9fddd",
            "b5db9719.a92f08",
            "4002d28e.02073c",
            "259600f8.a7623",
            "f80100ac.269b",
            "9ad00e7.b9d25f",
            "5c500f37.d472a",
            "a1ed1f34.48f3a",
            "119b99fd.b03276",
            "ec033520.b1ec68",
            "6e8fb92d.21aca8",
            "20abe24.1e9aa1e",
            "453f964b.e5b708",
            "c52dbcac.54844",
            "81104bf2.740698",
            "d1e1ccf6.3d214",
            "a1e4f146.707a1",
            "7bde58f9.fc4918",
            "d659ebad.1d80a8"
        ],
        "x": 174,
        "y": 579,
        "w": 1772,
        "h": 402
    },
    {
        "id": "74e66f38.426cb",
        "type": "function",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "Compare Times 15 minutes earlier",
        "func": "\n// construct newmsg\nnewmsg = {};\n\n// retrieve waktu solat from flow\nlet zone                = flow.get(\"zone\");\nlet data_download_date  = flow.get(\"data_download_date\");\nlet waktu_imsak         = flow.get(\"waktu_imsak\");\nlet subuhtime           = flow.get(\"waktu_subuh\");\nlet syuruktime          = flow.get(\"waktu_syuruk\");\nlet zohortime           = flow.get(\"waktu_zohor\");\nlet asartime            = flow.get(\"waktu_asar\");\nlet maghribtime         = flow.get(\"waktu_maghrib\");\nlet isyaktime           = flow.get(\"waktu_isyak\");\n\n//add 15 minutes to current time to compare with actual to 15 minute pre info\nvar minutesToAdd        = 15;\nvar currentDate         = new Date();\nvar previousDate        = new Date(currentDate.getTime() + minutesToAdd*60000);\n\nconst prehours          = previousDate.getHours().toString().padStart(2,'0');\nconst preminutes        = previousDate.getMinutes().toString().padStart(2,'0');\nconst pretime           = `${prehours}:${preminutes}`;\n\nnewmsg.kawasan = zone;\n\n\nif (pretime === subuhtime) \n{\n    newmsg.payload = \"Subuh\";\n    newmsg.solat_time = subuhtime;\n    newmsg.waktu = newmsg.payload;\n    return  newmsg;\n    \n} else if (pretime  === syuruktime) \n{\n    newmsg.payload = \"Syuruk\";\n    newmsg.solat_time = syuruktime;\n    newmsg.waktu = newmsg.payload;\n    return  newmsg;\n    \n} else if (pretime  === zohortime) \n{\n    newmsg.payload = \"Zohor\";\n    newmsg.solat_time = zohortime;\n    newmsg.waktu = newmsg.payload;\n    return  newmsg;\n    \n} else if (pretime  === asartime) \n{\n    newmsg.payload = \"Asar\";\n    newmsg.solat_time = asartime;\n    newmsg.waktu = newmsg.payload;\n    return  newmsg;\n    \n} else if (pretime  === maghribtime) \n{\n    newmsg.payload = \"Maghrib\";\n    newmsg.solat_time = maghribtime;\n    newmsg.waktu = newmsg.payload;\n    return  newmsg;\n    \n} else if (pretime  === isyaktime) \n{\n    newmsg.payload = \"Isyak\";\n    newmsg.solat_time = isyaktime;\n    newmsg.waktu = newmsg.payload;\n    return  newmsg;\n    \n} \n\nreturn newmsg;\n\n\n// debugging\n//return newmsg;\n//newmsg.waktu = newmsg.payload;\n//newmsg.topic = pretime;\n\n\n",
        "outputs": 1,
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "libs": [],
        "x": 460,
        "y": 620,
        "wires": [
            [
                "c0e5a10a.1b4a1",
                "7bde58f9.fc4918"
            ]
        ]
    },
    {
        "id": "fcf14654.85b548",
        "type": "change",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "Craft Message",
        "rules": [
            {
                "t": "set",
                "p": "payload",
                "pt": "msg",
                "to": "\"🕌 Assalamualaikum dan perhatian. Dalam masa 15 minit lagi akan masuk waktu \" & $.payload &\" pada jam \" & $.solat_time &\" bagi zon \" & $.kawasan",
                "tot": "jsonata"
            },
            {
                "t": "set",
                "p": "data",
                "pt": "msg",
                "to": "$.payload ",
                "tot": "jsonata"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 920,
        "y": 700,
        "wires": [
            [
                "4002d28e.02073c",
                "259600f8.a7623",
                "6e8fb92d.21aca8",
                "20abe24.1e9aa1e",
                "c52dbcac.54844"
            ]
        ]
    },
    {
        "id": "c0e5a10a.1b4a1",
        "type": "switch",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "Pre Solat Spesific Commands",
        "property": "payload",
        "propertyType": "msg",
        "rules": [
            {
                "t": "eq",
                "v": "Subuh",
                "vt": "str"
            },
            {
                "t": "eq",
                "v": "Syuruk",
                "vt": "str"
            },
            {
                "t": "eq",
                "v": "Zohor",
                "vt": "str"
            },
            {
                "t": "eq",
                "v": "Asar",
                "vt": "str"
            },
            {
                "t": "eq",
                "v": "Maghrib",
                "vt": "str"
            },
            {
                "t": "eq",
                "v": "Isyak",
                "vt": "str"
            }
        ],
        "checkall": "true",
        "repair": false,
        "outputs": 6,
        "x": 410,
        "y": 800,
        "wires": [
            [
                "fcf14654.85b548",
                "f80100ac.269b"
            ],
            [],
            [
                "fcf14654.85b548",
                "119b99fd.b03276"
            ],
            [
                "fcf14654.85b548",
                "119b99fd.b03276"
            ],
            [
                "b34fbd1d.58cc8",
                "fcf14654.85b548"
            ],
            [
                "fcf14654.85b548"
            ]
        ]
    },
    {
        "id": "b34fbd1d.58cc8",
        "type": "link out",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "Pre-Maghrib",
        "links": [
            "901dc84f.a4cdf8"
        ],
        "x": 810,
        "y": 820,
        "wires": [],
        "l": true
    },
    {
        "id": "1e72e30b.c9fddd",
        "type": "delay",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "",
        "pauseType": "delay",
        "timeout": "4",
        "timeoutUnits": "seconds",
        "rate": "1",
        "nbRateUnits": "1",
        "rateUnits": "second",
        "randomFirst": "1",
        "randomLast": "5",
        "randomUnits": "seconds",
        "drop": false,
        "x": 1300,
        "y": 880,
        "wires": [
            [
                "b5db9719.a92f08"
            ]
        ]
    },
    {
        "id": "b5db9719.a92f08",
        "type": "api-call-service",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "Google say",
        "server": "",
        "version": 3,
        "debugenabled": false,
        "service_domain": "tts",
        "service": "google_translate_say",
        "entityId": "media_player.chromecast",
        "data": "{\"message\":\"{{payload}}\",\"cache\":\"true\",\"language\":\"id\"}",
        "dataType": "json",
        "mergecontext": "",
        "mustacheAltTags": false,
        "outputProperties": [
            {
                "property": "payload",
                "propertyType": "msg",
                "value": "payload",
                "valueType": "msg"
            }
        ],
        "queue": "none",
        "x": 1470,
        "y": 880,
        "wires": [
            []
        ],
        "icon": "node-red-contrib-cast/google-home-mini1.svg"
    },
    {
        "id": "4002d28e.02073c",
        "type": "link out",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "T-PreWaktuSolat",
        "links": [
            "8718a1dd.5beb7"
        ],
        "x": 1115,
        "y": 820,
        "wires": [],
        "icon": "node-red-contrib-telegrambot/telegram.png"
    },
    {
        "id": "259600f8.a7623",
        "type": "link out",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "WS Speaker Out - TTS",
        "links": [
            "c4134f1d.de9a5"
        ],
        "x": 1115,
        "y": 660,
        "wires": [],
        "icon": "node-red-contrib-cast/google-home-mini2.svg"
    },
    {
        "id": "f80100ac.269b",
        "type": "link out",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "Waktu Solat Broadcast",
        "links": [
            "42d27463.d3c9ac",
            "71fe748c.fce81c"
        ],
        "x": 840,
        "y": 760,
        "wires": [],
        "l": true
    },
    {
        "id": "9ad00e7.b9d25f",
        "type": "delay",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "",
        "pauseType": "delay",
        "timeout": "30",
        "timeoutUnits": "seconds",
        "rate": "1",
        "nbRateUnits": "1",
        "rateUnits": "second",
        "randomFirst": "1",
        "randomLast": "5",
        "randomUnits": "seconds",
        "drop": false,
        "x": 1320,
        "y": 700,
        "wires": [
            [
                "ec033520.b1ec68"
            ]
        ]
    },
    {
        "id": "5c500f37.d472a",
        "type": "change",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "Override TV Scheduler - Turn ON",
        "rules": [
            {
                "t": "set",
                "p": "payload",
                "pt": "msg",
                "to": "on",
                "tot": "str"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 900,
        "y": 940,
        "wires": [
            [
                "a1ed1f34.48f3a"
            ]
        ]
    },
    {
        "id": "a1ed1f34.48f3a",
        "type": "link out",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "out-tv-scheduler-on",
        "links": [
            "eeaddb3.ba5fc28"
        ],
        "x": 1075,
        "y": 940,
        "wires": []
    },
    {
        "id": "119b99fd.b03276",
        "type": "api-current-state",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "Scheduler Active",
        "server": "",
        "version": 2,
        "outputs": 2,
        "halt_if": "off",
        "halt_if_type": "str",
        "halt_if_compare": "is",
        "entity_id": "switch.mos_tv_scheduler",
        "state_type": "str",
        "blockInputOverrides": false,
        "outputProperties": [],
        "x": 630,
        "y": 940,
        "wires": [
            [
                "5c500f37.d472a"
            ],
            []
        ]
    },
    {
        "id": "ec033520.b1ec68",
        "type": "function",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "Randomizer + Youtube",
        "func": "let playlist_random = getRndInteger(1,5);\nvar now = new Date();\n\nnode.status({fill:\"green\",shape:\"ring\",text:\" Playlist: \"+playlist_random}); \n\n//get global variable\nvar g = global.get(\"homeassistant\");\n//get states variable\nvar states = g.homeAssistant.states;\n//get the actual entity that we want\nvar video1 = states[\"input_text.azan_streaming_link_1\"].state;\nvar video2 = states[\"input_text.azan_streaming_link_2\"].state;\nvar video3 = states[\"input_text.azan_streaming_link_3\"].state;\nvar video4 = states[\"input_text.azan_streaming_link_4\"].state;\nvar video5 = states[\"input_text.azan_streaming_link_5\"].state;\n\nswitch (playlist_random) {\n    case(1):\n        // live makkah\n        // videoid = \"YsPvZXBFJko\";\n        videoid = video1;\n        break;\n    case(2):\n        // live madinah\n        //videoid = \"ERl92J7JREg\";\n        videoid = video2;\n        break;\n    case(3):\n        // madinah playback\n        //videoid = \"kGCCzo5jYhQ\";\n        videoid = video3;\n        break;\n    case(4):\n        // surah kahfi\n        //videoid = \"-rzG4nLUq-8\";\n        videoid = video4;\n        break;\n    case(5):\n        // live makkah\n        //videoid = \"2KRh6jCWEzI\";\n        videoid = video5;\n        break\n    default:\n        videoid = \"-rzG4nLUq-8\";\n}\n\nmsg.videoid = videoid;\nmsg.playlist = playlist_random;\nmsg.triggertime = now;\n\nmsg.payload = {\n\"app\": \"YouTube\",\n\"type\": \"MEDIA\",\n\"videoId\": videoid\n};\n\nreturn msg;\n\n// this code gets a random interger\nfunction getRndInteger(min, max) {\n    return Math.floor(Math.random() * (max - min + 1) ) + min;\n}",
        "outputs": 1,
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "libs": [],
        "x": 1560,
        "y": 700,
        "wires": [
            [
                "d1e1ccf6.3d214",
                "a1e4f146.707a1",
                "453f964b.e5b708"
            ]
        ]
    },
    {
        "id": "6e8fb92d.21aca8",
        "type": "api-call-service",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "Turn ON",
        "server": "",
        "version": 3,
        "debugenabled": false,
        "service_domain": "media_player",
        "service": "turn_on",
        "entityId": "media_player.chromecast",
        "data": "",
        "dataType": "jsonata",
        "mergecontext": "",
        "mustacheAltTags": false,
        "outputProperties": [],
        "queue": "none",
        "x": 1140,
        "y": 880,
        "wires": [
            [
                "1e72e30b.c9fddd"
            ]
        ]
    },
    {
        "id": "20abe24.1e9aa1e",
        "type": "wake on lan",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "mac": "20:17:42:d8:4e:8f",
        "host": "192.168.0.255",
        "udpport": 9,
        "name": "LGTV",
        "x": 1150,
        "y": 620,
        "wires": []
    },
    {
        "id": "453f964b.e5b708",
        "type": "link out",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "out-googletvchromecast",
        "links": [
            "56211c60.08f6c4",
            "cf92c33d.6c7d5"
        ],
        "x": 1875,
        "y": 700,
        "wires": []
    },
    {
        "id": "c52dbcac.54844",
        "type": "switch",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "",
        "property": "waktu",
        "propertyType": "msg",
        "rules": [
            {
                "t": "neq",
                "v": "Isyak",
                "vt": "str"
            },
            {
                "t": "eq",
                "v": "Isyak",
                "vt": "str"
            }
        ],
        "checkall": "true",
        "repair": false,
        "outputs": 2,
        "x": 1150,
        "y": 720,
        "wires": [
            [
                "9ad00e7.b9d25f"
            ],
            [
                "81104bf2.740698"
            ]
        ]
    },
    {
        "id": "81104bf2.740698",
        "type": "link out",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "o-presolat_broadcast",
        "links": [
            "82755dc7.5621f"
        ],
        "x": 1295,
        "y": 760,
        "wires": []
    },
    {
        "id": "d1e1ccf6.3d214",
        "type": "debug",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "",
        "active": false,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "true",
        "targetType": "full",
        "statusVal": "",
        "statusType": "auto",
        "x": 1850,
        "y": 760,
        "wires": []
    },
    {
        "id": "a1e4f146.707a1",
        "type": "ha-entity",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "PreSolat Youtube Log",
        "server": "",
        "version": 1,
        "debugenabled": false,
        "outputs": 1,
        "entityType": "sensor",
        "config": [
            {
                "property": "name",
                "value": "PreSolat Youtube Log"
            },
            {
                "property": "device_class",
                "value": ""
            },
            {
                "property": "icon",
                "value": "mdi:youtube"
            },
            {
                "property": "unit_of_measurement",
                "value": ""
            }
        ],
        "state": "playlist",
        "stateType": "msg",
        "attributes": [
            {
                "property": "Youtube Video",
                "value": "videoid",
                "valueType": "msg"
            },
            {
                "property": "Triggered time",
                "value": "triggertime",
                "valueType": "msg"
            }
        ],
        "resend": true,
        "outputLocation": "",
        "outputLocationType": "none",
        "inputOverride": "allow",
        "outputOnStateChange": false,
        "outputPayload": "$entity().state ? \"on\": \"off\"",
        "outputPayloadType": "jsonata",
        "x": 1800,
        "y": 640,
        "wires": [
            []
        ]
    },
    {
        "id": "7bde58f9.fc4918",
        "type": "debug",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "PreSolat",
        "active": false,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "true",
        "targetType": "full",
        "statusVal": "",
        "statusType": "auto",
        "x": 740,
        "y": 620,
        "wires": []
    },
    {
        "id": "d659ebad.1d80a8",
        "type": "link in",
        "z": "979d82cfea624e5f",
        "g": "98788efd.44a6e",
        "name": "i-minute-pulse",
        "links": [
            "e7de13f9.4c83f"
        ],
        "x": 215,
        "y": 620,
        "wires": [
            [
                "74e66f38.426cb"
            ]
        ]
    }
]
```

#### Bahagian 5 - Pengumuan Waktu, Memainkan Azan di Speaker dan Youtube Azan/Live Makkah/Madinah di TV.

1. Pengumuman Waktu
   1. Servis  Home Assistant `google_translate_say` digunakan. Pastikan servis ini sudah ditetapkan di dalam `configuration.yaml` bagi Integration  [Google TTS](https://www.home-assistant.io/integrations/google_translate/).
   2. Disebabkan kekangan platform Google TTS (yang disediakan oleh Home Assistant) - bahasa terdekat yang dapat diambil berbanding Bahasa Malaysia adalah Bahasa Indonesia dengan penetapan `language:id`.
   3. Node `craft Message` menetapkan apa yang mahu dikatakan secara dinamik - yakni melalui `payload` yang diterima dari node-node sebelum ini, termasuk pengumuman zone waktu solat.
2. Bagi memainkan audio Azan di speaker:
   1. Servis dari Home Assistant `play_media` digunakan. Servis ini mencari fail yang ditetapkan di path `media-source://media_source/local/` atau jika dari perspektif direktori HA : folder `media` (sama level dengan direktori `config`).
   2. Speaker boleh ditetapkan secara sendirian atau `group`.
   3. Beberapa pilihan `mp3` azan disimpan di `media` folder - agar pemilihan azan boleh dibuat secara rawak.
3. Bagi memainkan video Youtube Azan/Live Makkah/Madinah di TV:
   1. Berbanding diatas dan setelah beberapa perbandingan dibuat - node `cast2tv` digunakan yang membolehkan Node-Red berhubung terus dengan `Chromecast` devices. 
   2. Perbezaan (dan pengamatan) - node `cast2tv` ini adalah ia lebih mudah dan baik berbanding `media_player` disebabkan ia tidak bergantung kepada Home Assistant untuk memulakan `Cast`. Mutu dan kualiti video yang dimainkan juga lebih baik menggunakan library npm `cast2tv` berbanding Home Assistant yang menggunakan library `pyChromecast` bagi tujuan sama.
   3. Google Chromecast devices pada asasnya juga ada memberi signal `CEC HDMI` yang membolehkan TV dihidupkan secara automatik apabila *cast* dimulakan tetapi signal ini tidaklah dapat dipercayai berfungsi selalui, lebih-lebih lagi bagi yang begantung notifikasi Azan Subuh. Jadi tambahan node Wake-on-LAN `WOL` digunakan untuk menghidupkan TV dahulu (tidak semua TV support ini termasuk Wifi - LAN adalah cara terbaik).
   4. Sama seperti audio - beberapa *playlist* boleh ditetapkan supaya pemilihan secara rawak dapat dibuat. Bagi Live Makkah / Madinah - livestream ini acapkali ditukar (diturunkan sebab copyright notice) jadi tambahan penepatan dibuat supaya *playlist* ini boleh diubah di Home Assistant melalui `input_text` tanpa perlu mengubahnya di Node-Red semula. 

![](C:\Users\user\Documents\GitHub\HA-NR-WaktuSolatJakim\images\Step 5 - Casting and Playback_cmp.PNG)

```json [
    {
        "id": "cf92c33d.6c7d5",
        "type": "link in",
        "z": "979d82cfea624e5f",
        "name": "in-googletvchromecast",
        "links": [
            "db1127cc.ff45f8",
            "453f964b.e5b708"
        ],
        "x": 835,
        "y": 1220,
        "wires": [
            [
                "5c8b6549.8a20dc"
            ]
        ]
    },
    {
        "id": "c05e8a0a.be02d8",
        "type": "api-call-service",
        "z": "979d82cfea624e5f",
        "name": "TTS",
        "server": "",
        "version": 3,
        "debugenabled": false,
        "service_domain": "tts",
        "service": "google_translate_say",
        "entityId": "media_player.living_room_speaker",
        "data": "{\"message\":\"{{payload}}\",\"cache\":\"true\",\"language\":\"id\"}",
        "dataType": "json",
        "mergecontext": "",
        "mustacheAltTags": false,
        "outputProperties": [],
        "queue": "none",
        "x": 1210,
        "y": 1320,
        "wires": [
            []
        ],
        "icon": "node-red-contrib-cast/google-home-mini2.svg"
    },
    {
        "id": "8a9cb680.d7e8f8",
        "type": "api-call-service",
        "z": "979d82cfea624e5f",
        "name": "Set Volume 0.7",
        "server": "",
        "version": 3,
        "debugenabled": false,
        "service_domain": "media_player",
        "service": "volume_set",
        "entityId": "media_player.living_room_speaker",
        "data": "{\"volume_level\":\"0.7\"}",
        "dataType": "jsonata",
        "mergecontext": "",
        "mustacheAltTags": false,
        "outputProperties": [],
        "queue": "none",
        "x": 940,
        "y": 1280,
        "wires": [
            [
                "c05e8a0a.be02d8"
            ]
        ]
    },
    {
        "id": "c4134f1d.de9a5",
        "type": "link in",
        "z": "979d82cfea624e5f",
        "name": "WS Speaker In - TTS",
        "links": [
            "208d52d.827ddae",
            "259600f8.a7623"
        ],
        "x": 400,
        "y": 1320,
        "wires": [
            [
                "aa2959c9.a2e4e8"
            ]
        ],
        "icon": "node-red-contrib-cast/google-home-mini2.svg",
        "l": true
    },
    {
        "id": "aa2959c9.a2e4e8",
        "type": "api-current-state",
        "z": "979d82cfea624e5f",
        "name": "WS Not occupied",
        "server": "",
        "version": 2,
        "outputs": 2,
        "halt_if": "on",
        "halt_if_type": "str",
        "halt_if_compare": "is_not",
        "entity_id": "switch.flag_nomotion_walidstudy",
        "state_type": "str",
        "blockInputOverrides": false,
        "outputProperties": [
            {
                "property": "data",
                "propertyType": "msg",
                "value": "",
                "valueType": "entityState"
            }
        ],
        "override_topic": false,
        "state_location": "payload",
        "override_payload": "msg",
        "entity_location": "data",
        "override_data": "msg",
        "x": 630,
        "y": 1320,
        "wires": [
            [
                "8a9cb680.d7e8f8"
            ],
            [
                "968d63f1.d7f6c"
            ]
        ]
    },
    {
        "id": "968d63f1.d7f6c",
        "type": "api-call-service",
        "z": "979d82cfea624e5f",
        "name": "Set Volume 0.2",
        "server": "",
        "version": 3,
        "debugenabled": false,
        "service_domain": "media_player",
        "service": "volume_set",
        "entityId": "media_player.living_room_speaker",
        "data": "{\"volume_level\":\"0.2\"}",
        "dataType": "jsonata",
        "mergecontext": "",
        "mustacheAltTags": false,
        "outputProperties": [],
        "queue": "none",
        "x": 940,
        "y": 1360,
        "wires": [
            [
                "c05e8a0a.be02d8"
            ]
        ]
    },
    {
        "id": "42554b7d.917a04",
        "type": "api-call-service",
        "z": "979d82cfea624e5f",
        "name": "Audio/Azan Subuh",
        "server": "",
        "version": 3,
        "debugenabled": true,
        "service_domain": "media_player",
        "service": "play_media",
        "entityId": "media_player.living_room_speaker",
        "data": "{\"media_content_id\":\"media-source://media_source/local/{{payload}}\",\"media_content_type\":\"audio/mp3\"}",
        "dataType": "json",
        "mergecontext": "",
        "mustacheAltTags": false,
        "outputProperties": [],
        "queue": "none",
        "x": 1250,
        "y": 1460,
        "wires": [
            []
        ]
    },
    {
        "id": "b08e9a29.a14828",
        "type": "link in",
        "z": "979d82cfea624e5f",
        "name": "WS Speaker In - Audio/MP3",
        "links": [
            "8f54ce9.8394b3",
            "8cf62c63.4fabb",
            "b22d32be.52878"
        ],
        "x": 380,
        "y": 1460,
        "wires": [
            [
                "58eb9f86.83d79"
            ]
        ],
        "icon": "node-red-contrib-cast/google-home-mini2.svg",
        "l": true
    },
    {
        "id": "a10738af.905948",
        "type": "api-call-service",
        "z": "979d82cfea624e5f",
        "name": "Set Volume 0.8",
        "server": "",
        "version": 3,
        "debugenabled": false,
        "service_domain": "media_player",
        "service": "volume_set",
        "entityId": "media_player.living_room_speaker",
        "data": "{\"volume_level\":\"0.8\"}",
        "dataType": "jsonata",
        "mergecontext": "",
        "mustacheAltTags": false,
        "outputProperties": [],
        "queue": "none",
        "x": 940,
        "y": 1420,
        "wires": [
            [
                "42554b7d.917a04"
            ]
        ]
    },
    {
        "id": "58eb9f86.83d79",
        "type": "api-current-state",
        "z": "979d82cfea624e5f",
        "name": "WS Not occupied",
        "server": "",
        "version": 2,
        "outputs": 2,
        "halt_if": "on",
        "halt_if_type": "str",
        "halt_if_compare": "is_not",
        "entity_id": "switch.flag_nomotion_walidstudy",
        "state_type": "str",
        "blockInputOverrides": false,
        "outputProperties": [
            {
                "property": "data",
                "propertyType": "msg",
                "value": "",
                "valueType": "entityState"
            }
        ],
        "override_topic": false,
        "state_location": "payload",
        "override_payload": "msg",
        "entity_location": "data",
        "override_data": "msg",
        "x": 630,
        "y": 1460,
        "wires": [
            [
                "a10738af.905948"
            ],
            [
                "5c14cb4c.940c34"
            ]
        ]
    },
    {
        "id": "5c14cb4c.940c34",
        "type": "api-call-service",
        "z": "979d82cfea624e5f",
        "name": "Set Volume 0.15",
        "server": "",
        "version": 3,
        "debugenabled": false,
        "service_domain": "media_player",
        "service": "volume_set",
        "entityId": "media_player.living_room_speaker",
        "data": "{\"volume_level\":\"0.15\"}",
        "dataType": "jsonata",
        "mergecontext": "",
        "mustacheAltTags": false,
        "outputProperties": [],
        "queue": "none",
        "x": 940,
        "y": 1500,
        "wires": [
            [
                "42554b7d.917a04"
            ]
        ]
    },
    {
        "id": "5c8b6549.8a20dc",
        "type": "castv2-sender",
        "z": "979d82cfea624e5f",
        "name": "Google TV",
        "connection": "1f6c530c.e04cdd",
        "x": 950,
        "y": 1220,
        "wires": [
            []
        ]
    },
    {
        "id": "1f6c530c.e04cdd",
        "type": "castv2-connection",
        "name": "",
        "target": "GoogleTV",
        "host": "",
        "port": "8009"
    }
]
```

Bahagian 6 - Lain-lain

-  Telegram node `/getwaktusolat`  for manual call request and sets flags to identify at output to only send Telegram update when requested. 
- PRAYER TIMES ACTION - CUSTOMIZE TO YOUR PREFERENCE
![ACTION!](https://github.com/anas-ivs/HA-NR-WaktuSolatJakim/blob/main/images/Flow_Actions.PNG)
1. Trigger Telegram Notification:

2. Trigger Chromecast to play TTS message.

3. Trigger to play Youtube video on `media_player` i.e. TV based on randomize list. 


-  Telegram node `/getwaktusolat`  for manual call request and sets flags to identify at output to only send Telegram update when requested. 


## <a name="Pre"> Pra-Syarat</a>
1.  Home Assistant with Node-Red. Ada banyak tutorial/videos dalam tajuk ini; pastikan di dalam Node Red boleh dicapai melalui Home Assistant.
    1.  [Tutorial Pertama](https://www.juanmtech.com/get-started-with-node-red-and-home-assistant/) dari juanmtech.
    2.  [Tutorial Kedua](http://shahrulnizam.com/home-assistant-nodered/) dari ShahrulNizam.com
2.  Melalui HACS store -  install [Node-Red companion integration](https://github.com/zachowj/hass-node-red). Ini memboleh entity seperti `sensor` dibuat oleh Node Red.
3.  Bagi Node-Red; pallette berikut diperlukan (Ikon burger di atas hujung kanan -> Manage Pallette -> Install ) 
    1.  Kritikal
        1.  `node-red-contrib-home-assistant-websocket` -  Pre-installed jika Node-Red dipasang melalui Home Assistant Supervisor.
        2.  `node-red-contrib-cron-plus` - Bagi menjalankan tugasan berkala mengikut setting yang ditetapkan. Di dalam dunia linux - ini adalah seperti `cronjobs`.
    2.  Tambahan (Node ini boleh dibuang jika tidak memerlukan feature-feature ini).
        1.  `node-red-contrib-telegrambot` - Bagi memberi notifkasi melalui Telegram.
        2.  `node-red-dashboard` - Bagi *feature* dashboard Node-Red.
        3.  `node-red-node-wol` - Bagi *feature Wake on LAN* supaya TV dapat dihidupkan dari automation.
        4.  `node-red-contrib-castv2` - Bagi membolehkan Node-Red berhubung terus dengan `Chromecast` devices dan memulakan servis, misalnya casting Youtube video.

5. Bagi Home Assistant, berikut diperlukan:

   1.  [Node-Red companion integration](https://github.com/zachowj/hass-node-red); install melalui [HACS](https://hacs.xyz/). Ini membolehkan entity seperti `sensor` dibuat oleh Node Red berbanding konfigurasi Home Assistant menggunkan `configuration.yaml` atau `helpers`. 

   > **Tip:** Nama entity node hendaklah di-isi dahulu sebelum deploy. Ini membolehkan nama entity yang diberi mengikut kehendak kita. Jika tidka di-isi Node-Red/HA akan memberi nama secara rawak dan ini tidak boleh ditukar melainkan node dibuang dan dibuat semula.

   2. [Vertical stack in card](https://github.com/ofekashery/vertical-stack-in-card) membolehkan penyusunan lovelace secara kompak tanpa border.

6. Bagi Telegram - mempunyai Telegram bot and chat ids. [Tutorial](https://www.thesmarthomebook.com/2020/10/13/a-guide-to-using-telegram-with-node-red-and-home-assistant/) dari TheSmartHomeBook ini mudah difahami dan diikuti. 

> **Tip:** Bagi yang sudah mempunyai Telegram integration di Home Assistant; asignkan bot dengan menghidupkan satu lagi bot dan channel khas bagi Node Red. 

## <a name="Install">##Pemasangan</a>
1. Tentukan Zone salah bagi kawasan anda melalui [listing](https://api.azanpro.com/zones) ini. Contohnya; Bagi kawasan Miri; kodnya adalah SWK02. 

2. Import [Flow](#Flow) ke dalam Node Red (Ikon burger di atas hujung kanan -> Import). Satu tab baru akan dengan nama 'Waktu Solat'

3. Selesai Import - sebelum click Deploy, beberapa ketatapan perlu dibuat dahulu:

   1. Ubah zone di dalam node `Solat API dan Zone`. Gantikan const zone_api = 'SWK02';` dengan zone kawasan anda.

      ![](C:\Users\user\Documents\GitHub\HA-NR-WaktuSolatJakim\images\set zone.PNG)

   2. Node-node yang menunjukan segi-tiga merah hendaklah diubah *configuration* dahulu.

      ![Gambar!!](C:\Users\user\Documents\GitHub\HA-NR-WaktuSolatJakim\images\node_setup.PNG)

      1. Node Home Assistant (Warna biru) - Tetapkan supaya ia dapat akses ke Home Assistant. Bagi yang install melalui Supervisor - pilih server Home Assistant yang sudah disediakan.

         ​	![GAMBAR](C:\Users\user\Documents\GitHub\HA-NR-WaktuSolatJakim\images\entity_homeassistant.PNG)

      2. Telegram (Logo Telegram) - tetapkan (sekali sahaj perlu) Bot yang digunakan. Boleh rujuk seperti [tutorial](https://www.thesmarthomebook.com/2020/10/13/a-guide-to-using-telegram-with-node-red-and-home-assistant/). Boleh rujuk juga tips saya [https://github.com/anas-ivs/HA-NR-Flows#Telegram](https://github.com/anas-ivs/HA-NR-Flows#Telegram) bagi memanfaatkan notifikasi Telegram supaya tidaklah selalu menggangu.

         ![](C:\Users\user\Documents\GitHub\HA-NR-WaktuSolatJakim\images\entity_telegram.PNG)

      3. Video - Cast2TV (Logo Chromecast) - Tetapkan device `chromecast` dengan menekan butang `search`. Senarai device yang mempunyai feature `chromecast`  akan disenaraikan - pilih dan tetapkan satu (yang boleh mengeluarkan video).

         ![](C:\Users\user\Documents\GitHub\HA-NR-WaktuSolatJakim\images\entity_cast2tv.PNG)

      4. Audio - Media Player - Tetapkan entity id bagi `media_player` tersedia yang digunakan di dalam Home Assistant.

         ![](C:\Users\user\Documents\GitHub\HA-NR-WaktuSolatJakim\images\entity_media_player.PNG)

      5. Wake On Lan (Logo Mentol) - tetapkan MAC ID dan IP Address bagi TV anda yang dihubungkan melalui LAN.

         ![](C:\Users\user\Documents\GitHub\HA-NR-WaktuSolatJakim\images\entity_WOL_cmp.PNG)

   

   3. Setelah semua `error` dan *settings* ditetapkan - baru click 'Deploy'.
   4. Bagi kali pertama - Klik inject node `Tekan sekali untuk kali pertama` - ini membolehkan data didownload terlebih dahulu tanpa menunggu bulan hadapan.

3. Home Assistant

![](C:\Users\user\Documents\GitHub\HA-NR-WaktuSolatJakim\images\Lovelace-entities-WaktuSolat.PNG)

- Melalui [Node-Red companion integration](https://github.com/zachowj/hass-node-red) yang sudah awal tadi diinstall -pergi ke `Configuration -> Integrations -> Node-Red` . Entity-entity berikut sekarang tersedia:

  -`sensor.zone_waktu_solat`

  -`sensor.solat_bakiwaktu`

  -`sensor.waktu_solat` - Dengan `attribute` setiap waktu solat.

  -`sensor.subuh`, `sensor.syuruk`, `sensor.zohor`, `sensor.asar`, `sensor.maghrib`, `sensor.isyak`


3. Lovelace - Import and customize to your liking.

![](C:\Users\user\Documents\GitHub\HA-NR-WaktuSolatJakim\images\Lovelace-Waktu Solat.PNG)

```yaml
type: custom:vertical-stack-in-card
cards:
  - type: entity
    entity: sensor.waktu_solat
    name: 'Waktu Solat '
  - type: markdown
    content: >
      Sekarang waktu
      **{{state_attr('sensor.solat_bakiwaktu','waktu_solat_sekarang')}}**. 
      Waktu {{state_attr('sensor.solat_bakiwaktu','waktu_solat_berikut')}} dalam
      masa {{states('sensor.solat_bakiwaktu')}}.
  - type: glance
    style: |
      ha-card {
        background-color: var(--primary-background-color);
        border-radius: 15px;
        margin: 10px;
        font-size: 6 px
        box-shadow:
          {% if is_state('sun.sun', 'above_horizon') %}
            -4px -4px 8px rgba(255, 255, 255, .5), 5px 5px 8px rgba(0, 0, 0, .03);
          {% elif is_state('sun.sun', 'below_horizon') %}
            -5px -5px 8px rgba(50, 50, 50, .2), 5px 5px 8px rgba(0, 0, 0, .08);
          {% endif %}
       }
        .card-header {
        font-size: 6 px
      }
    entities:
      - entity: sensor.subuh
      - entity: sensor.syuruk
      - entity: sensor.zohor
      - entity: sensor.asar
      - entity: sensor.maghrib
      - entity: sensor.isyak
    state_color: true
    columns: 6


```

## <a name="Credits"> Credits </a>
##### Credit to @aitalinassim / 
- [@farxpeace]() Original implementation in HA.
- [@aitalinassim]() Node-Red baseline flow.
- [@xsoh](https://github.com/xsoh/Hijri.js/blob/master/Hijri.js) Gregorian to Hijri calendar convert. Added 15/6

##### [Home Assistant Malaysia](https://www.facebook.com/groups/homeassistantmalaysia)
