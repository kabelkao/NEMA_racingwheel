# DIY Formula Wheel for PC

## Cíl projektu

Postavit vlastní USB formulový volant pro PC (F1 2019, Assetto Corsa, BeamNG, ETS2), který:

- nebude mít vůle jako DS4 převod
- bude mít vysoké rozlišení kolem středu
- bude používat pistolové spouště místo pedálů
- bude rozšiřitelný o displej, LED, vibrace a později aktivní odpor volantu
- bude vhodný i pro děti

---

# Koncepce

První verze nebude řešit skutečný Force Feedback.

Prioritou je:

1. přesné snímání natočení
2. nulová mechanická vůle
3. USB HID kompatibilita
4. plyn a brzda na spouštích

Až potom:

- telemetrie
- vibrace
- LED
- aktivní centrování
- soft dorazy
- Force Feedback

---

# Architektura

```text
                    PC
                     │
               USB HID + CDC
                     │
                 ESP32-S3
                     │
 ┌─────────────┬─────────────┬─────────────┐
 │             │             │             │
AS5600      Hall plyn    Hall brzda    MCP23017
řízení       trigger      trigger      tlačítka
```

Rozšíření:

```text
ESP32-S3
 ├─ TFT displej (SPI)
 ├─ WS2812 LED
 ├─ Vibrační motory
 ├─ MCP23017 #1
 └─ MCP23017 #2 (volitelné)
```

---

# Mechanika

## Osa řízení

Jako základ bude použit běžný krokový motor NEMA17 pouze jako mechanická osa.

Zpočátku nebude zapojen elektricky.

Výhody:

- kvalitní ložiska
- přesná hřídel
- jednoduchá montáž
- připraveno pro pozdější aktivní odpor

```text
      Volant
         │
   Tištěný náboj
         │
      Hřídel
         │
      NEMA17
```

---

## Náboj volantu

Materiál:

- PETG
- ASA

Parametry:

```text
5-6 perimetrů
100% výplň
```

Otvor:

```text
5.0 až 5.1 mm
```

Uchycení:

```text
M3 stavěcí šroub
```

V případě potřeby:

```text
2× M3 proti sobě
```

První verze nebude používat kovovou přírubu.

---

# Snímání natočení

## AS5600

Použit bude modul AS5600.

Vlastnosti:

```text
4096 kroků / otáčka
12bit
bezkontaktní měření
```

---

## Magnet

Použít:

```text
diametrálně magnetizovaný neodymový magnet
```

Umístění:

```text
zadní konec hřídele NEMA17

      magnet
         │
      1-2 mm
         │
      AS5600
```

Důležité:

- magnet musí být přesně na ose
- střed magnetu musí být nad středem AS5600
- vzdálenost cca 1-2 mm

---

# Očekávané rozlišení

Původní DS4 joystick:

```text
≈ 256 kroků
```

AS5600:

```text
4096 kroků
```

Při použitém rozsahu přibližně ±90°:

```text
~2000 použitelných pozic
```

Výrazně jemnější řízení než DS4.

---

# Plyn a brzda

Použití:

```text
Hall senzor SS49E
```

Rozložení:

```text
levý ukazovák  = brzda
pravý ukazovák = plyn
```

Výhody:

- bez opotřebení
- žádné potenciometry
- plynulý analogový průběh

---

# Řazení

Rozložení:

```text
levý prostředník  = podřazení
pravý prostředník = řazení nahoru
```

Možnosti:

- mikrospínače
- pádla

---

# Tlačítka

Primárně ovládána palci.

Příklad:

```text
DRS
ERS
Pit limiter
Kamera
Menu
Potvrzení
```

---

# IO Expandér

Použít:

```text
MCP23017
```

Výhoda:

```text
16 GPIO přes I2C
```

Možnost přidat druhý:

```text
MCP23017 #2
```

a získat:

```text
32 GPIO
```

---

# TFT displej

TFT nebude připojen přes MCP23017.

Použití:

```text
SPI
```

Na stejné sběrnici jako AS5600 (pokud bude potřeba jiný SPI senzor, oddělené CS).

První verze displeje:

```text
STEER 2048
THR   1200
BRK    300

BTN1 ON
BTN2 OFF
```

---

# RGB LED

Použít:

```text
WS2812B
```

Funkce:

- shift lights
- upozornění
- diagnostika

Příklad:

```text
🟩🟩🟩🟩🟨🟨🟥🟥
```

---

# Vibrace

Použití:

```text
2× vibrační motorek
```

v rukojetích.

Budoucí efekty:

- obrubníky
- prokluz kol
- ABS
- omezovač

---

# Telemetrie

ESP32 nebude pouze HID.

Současně poběží:

```text
USB HID
+
USB CDC
```

ESP32 tak bude vidět jako:

```text
Volant
+
COM port
```

Telemetrie:

```text
PC
 ↓
malá aplikace
 ↓
ESP32
 ↓
LED
vibrace
displej
```

---

# Aktivní odpor volantu

Priorita po dokončení základní funkce.

HW:

```text
NEMA17
+
TMC2209
```

První funkce:

## Soft dorazy

Např.

```text
±60°
volný pohyb

±65°
silný odpor

±70°
stop
```

---

## Elektronická pružina

```text
síla = K × odchylka od středu
```

Profily:

```text
děti
F1
ETS2
```

---

# Vývojové fáze

## Fáze 1

- ESP32-S3
- AS5600
- NEMA17 jako mechanika
- USB HID osa řízení

Cíl:

```text
joy.cpl
```

a funkční volant ve Windows.

---

## Fáze 2

- plyn
- brzda
- řazení
- tlačítka

---

## Fáze 3

- TFT displej
- diagnostika
- kalibrace

---

## Fáze 4

- telemetrie
- LED
- vibrace

---

## Fáze 5

- TMC2209
- aktivní odpor
- soft dorazy
- centrování

---

# Hlavní zásady

- žádné převody
- žádný joystick z DS4
- žádné pružiny
- žádné mechanické vůle

Primární cíl:

**co nejpřesnější řízení kolem středu a jednoduchá mechanika.**
