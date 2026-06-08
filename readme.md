# ESPHome HAN Port Reader (Czech Smart Meters)

Tento projekt obsahuje konfiguraci pro **ESPHome** běžící na mikrokontroleru **Raspberry Pi Pico W** (RP2040), která slouží k pasivnímu vyčítání dat z digitálních inteligentních elektroměrů (smart meterů) v České republice (ČEZ Distribuce, EG.D, PRE) přes jejich **HAN rozhraní** (sériová linka / UART přes fyzickou vrstvu RS485). Vyčítání dat z elektoměrů tímto způsobem je povolené a není třeba ditributora žádat.

Projekt řeší spolehlivý sběr surových dat z UART bufferu, precizní detekci konce zpráv (packet timeout) a okamžité publikování čistých entit (OBIS kódů) do **Home Assistanta**.

https://www.cezdistribuce.cz/pro-zakazniky/potrebuji-vyresit/elektromery-a-odecty/komunikacni-rozhrani-z-elektromeru

## Hlavní vlastnosti

* **Optimalizace pro RP2040 (Raspberry Pi Pico W):** Využívá hardwarový UART s dostatečnou vyrovnávací pamětí (`rx_buffer_size: 2048`), což brání ztrátě bajtů při rychlém burst přenosu z elektroměru (kdy naráz přichází stovky bajtů).
* **Precizní detekce konce zprávy (Packet Timeout):** Pomocí 200ms časové smyčky (`interval: 200ms`) kód spolehlivě identifikuje utichnutí linky po odeslání celého balíčku a teprve poté spustí parsování. Zamezuje se tím chybám z neúplných dat.
* **Event-driven aktualizace (Šetření sítě & CPU):** Jednotlivé template senzory mají vypnutou automatickou periodu aktualizace (`update_interval: never`). Do Home Assistanta se data posílají pouze tehdy, když elektroměr reálně odešle nový telegram (Push režim).
* **Bezpečné Native API:** Veškerá komunikace s Home Assistantem je šifrována pomocí vestavěného noise šifrovacího klíče.

## Hardwarové požadavky

1. **Raspberry Pi Pico W** (případně jiný podporovaný RP2040/ESP32 čip).
   Raspberry Pi Pico WH - RP2040 ARM Cortex M0 + CYW43439 - WiFi - s konektory  RPI-21575
   https://botland.cz/moduly-a-soupravy-pro-raspberry-pi-pico/21575-raspberry-pi-pico-wh-rp2040-arm-cortex-m0-cyw43439-wifi-s-konektory-5056561800196.html
2. **Převodník RS485 na TTL / UART:** Je naprosto klíčové použít modul, který podporuje **3.3V logiku** (např. s čipem SP3485). Běžné levné 5V moduly (MAX485) mohou Pico W poškodit, pokud nemají posun napěťových úrovní (level shifter).
 https://botland.cz/dalsi-moduly-pro-raspberry-pi-pico/20098-2kanalovy-rs485-2kanalovy-uart-rs485-sp3485-pro-raspberry-pi-pico-waveshare-19717-5904422380144.html
  2kanálový RS485 - 2kanálový UART -RS485 SP3485 - pro Raspberry Pi Pico - Waveshare 19717 WSR-20098
3. **Kabel RJ12:** Zapojený do HAN portu elektroměru dle specifikace vašeho distributora.
https://www.aliexpress.com/item/1005006226706163.html?spm=a2g0o.order_list.order_list_main.62.1cca1802kPYa0H
4. Zásuvka RJ12
https://www.aliexpress.com/item/1005005928565508.html?spm=a2g0o.order_list.order_list_main.67.1cca1802kPYa0H
5. Napajecí zdroj

### Sestavená čtečka
<img width="350" height="507" alt="image" src="https://github.com/user-attachments/assets/72549c73-d633-4722-b1a3-b7c745a6be78" />

### Čtečka se obvykle připojuje k relé boxu, pokud není osazen tak přimo na ELM.
<img width="666" height="662" alt="image" src="https://github.com/user-attachments/assets/e5f7141f-db1d-4685-82e4-43318a14f6ee" />


### Příklad schématu zapojení

https://www.cezdistribuce.cz/file/edee/distribuce/ppnn/vp_1-13.pdf


<img width="904" height="110" alt="image" src="https://github.com/user-attachments/assets/ef8cc6d5-3e69-44bc-8c33-abce36fe932c" />

<img width="677" height="457" alt="image" src="https://github.com/user-attachments/assets/3106deb2-7171-413f-a116-c6681e63ca0b" />

### Potřebné doplňky a nastavení v Home Assistantovi
Aby vše správně fungovalo a data se korektně zobrazovala, je potřeba mít v Home Assistantovi připravené následující součásti:

1. ESPHome Device Builder (Add-on)
Pro správu kódu, kompilaci firmwaru a následné bezdrátové aktualizace (OTA) je nutné mít nainstalovaný doplněk ESPHome Device Builder (v seznamu doplňků HA dostupný jako oficiální ESPHome Add-on).

Slouží jako webový editor pro váš YAML soubor.

Zajišťuje kompilaci kódu pro čip RP2040 (Pico W) na pozadí.

Umožňuje nahrání nového firmwaru vzduchem přes Wi-Fi přímo z rozhraní Home Assistanta bez nutnosti odpojovat hardware od elektroměru.

2. ESPHome Integrace (Jádro HA)
Propojení mezi běžícím modulem a systémem HA zajišťuje vestavěná integrace ESPHome.

Jakmile se Pico W poprvé úspěšně připojí k vaší Wi-Fi, Home Assistant zařízení automaticky detekuje a zobrazí upozornění v sekci Nastavení -> Zařízení a služby.

Při spárování budete vyzváni k zadání šifrovacího klíče (encryption key), který zkopírujete ze své konfigurace (sekce api: encryption: key:).

## Skrývání interních/pomocných senzorů
Pokud v konfiguraci používáte pomocné textové nebo číselné senzory (např. pro zachycení surového řetězce z UARTu, diagnostiku textu nebo mezivýpočty) a nechcete, aby vám tyto pomocné proměnné zbytečně zaplňovaly přehled entit v Home Assistantovi, skryjte je pomocí parametru internal: true. Část senzorů je již v konfiguraci skryta.
 
Senzor tak zůstane plně funkční pro interní potřeby automatizací a lambda kódů uvnitř ESPHome, ale vůbec se neexportuje do Home Assistanta:

### Logování a diagnostika
Při spuštění můžete v logu vidět pravidelné hlášky:
[uart:014]: Reading from UART timed out at byte 0!
To je v tomto zapojení zcela normální chování. Znamená to pouze, že v daném 200ms okně elektroměr zrovna nic nevysílal (jelikož čeká na svůj minutový interval). Jakmile elektroměr data pošle, v logu se objeví úspěšné sestavení: Complete frame (355 bytes).

## Poděkování / Autoři
Celé toto technické řešení, logiku vyčítání z UART bufferu vymyslel a navrhl kamarád Michal ve spolupráci s umělou inteligencí (AI).  

## Licence
  
Tento projekt je šířen pod licencí MIT. Můžete jej volně upravovat a přizpůsobovat svým podmínkám.

## Senzory v HA
<img width="350" height="748" alt="image" src="https://github.com/user-attachments/assets/8dfdc969-cfda-43fa-a38e-8d14e21abb5d" />

<img width="318" height="714" alt="image" src="https://github.com/user-attachments/assets/f9c6193f-a5da-4436-a689-451b86bcf965" />


## Dashboard příklad
<img width="505" height="609" alt="image" src="https://github.com/user-attachments/assets/35031954-7672-4e8c-a439-79c9dd88f659" />

