readme_content = """# ESPHome HAN Port Reader (Czech Smart Meters)

Tento projekt obsahuje konfiguraci pro **ESPHome** běžící na mikrokontroleru **Raspberry Pi Pico W** (RP2040), která slouží k pasivnímu vyčítání dat z digitálních inteligentních elektroměrů (smart meterů) v České republice (ČEZ Distribuce, EG.D, PRE) přes jejich **HAN rozhraní** (sériová linka / UART přes fyzickou vrstvu RS485).

Projekt řeší spolehlivý sběr surových dat z UART bufferu, precizní detekci konce zpráv (packet timeout) a okamžité publikování čistých entit (OBIS kódů) do **Home Assistanta**.

## Hlavní vlastnosti

* **Optimalizace pro RP2040 (Raspberry Pi Pico W):** Využívá hardwarový UART s dostatečnou vyrovnávací pamětí (`rx_buffer_size: 2048`), což brání ztrátě bajtů při rychlém burst přenosu z elektroměru (kdy naráz přichází stovky bajtů).
* **Precizní detekce konce zprávy (Packet Timeout):** Pomocí 200ms časové smyčky (`interval: 200ms`) kód spolehlivě identifikuje utichnutí linky po odeslání celého balíčku a teprve poté spustí parsování. Zamezuje se tím chybám z neúplných dat.
* **Event-driven aktualizace (Šetření sítě & CPU):** Jednotlivé template senzory mají vypnutou automatickou periodu aktualizace (`update_interval: never`). Do Home Assistanta se data posílají pouze tehdy, když elektroměr reálně odešle nový telegram (Push režim).
* **Bezpečné Native API:** Veškerá komunikace s Home Assistantem je šifrována pomocí vestavěného noise šifrovacího klíče.

## Hardwarové požadavky

1. **Raspberry Pi Pico W** (případně jiný podporovaný RP2040/ESP32 čip).
2. **Převodník RS485 na TTL / UART:** Je naprosto klíčové použít modul, který podporuje **3.3V logiku** (např. s čipem SP3485). Běžné levné 5V moduly (MAX485) mohou Pico W poškodit, pokud nemají posun napěťových úrovní (level shifter).
3. **Kabel RJ12:** Zapojený do HAN portu elektroměru dle specifikace vašeho distributora.

### Příklad schématu zapojení
