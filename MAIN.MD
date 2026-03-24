# Technika Cyfrowa i Mikroprocesorowa — Laboratorium 2

## Kompletny przewodnik z rozwiązaniami (ATmega328P / Arduino Nano / EduTar-NANO)

---

## Spis treści

1. [Obsługa portów — wyjścia i wejścia](#1-obsługa-portów--wyjścia-i-wejścia)
2. [Generowanie dźwięku — buzzer](#2-generowanie-dźwięku--buzzer)
3. [Wyświetlacz 7-segmentowy — 1 cyfra](#3-wyświetlacz-7-segmentowy--1-cyfra)
4. [Wyświetlacz 7-segmentowy — 4 cyfry — multipleksowanie](#4-wyświetlacz-7-segmentowy--4-cyfry--multipleksowanie)
5. [Zadanie domowe — funkcja nieblokująca](#5-zadanie-domowe--funkcja-nieblokująca)

---

## 1. Obsługa portów — wyjścia i wejścia

### Zadanie 1.1 — Pytania i odpowiedzi

#### Jak ustawić PB0 jako wyjście w stanie WYSOKIM?

```c
DDRB |= (1 << PB0);   // PB0 jako wyjście (1 w rejestrze DDR)
PORTB |= (1 << PB0);  // PB0 w stanie wysokim (1 w rejestrze PORT)
```

**Wyjaśnienie:**
- `DDRB` (Data Direction Register B) — ustawia kierunek pinu: `1` = wyjście, `0` = wejście
- `PORTB` — ustawia stan wyjścia: `1` = stan wysoki (HIGH, ~5V), `0` = stan niski (LOW, 0V)
- `|=` — operator OR z przypisaniem, ustawia wybrany bit bez zmieniania pozostałych

#### Jak ustawić PB0 jako wyjście w stanie NISKIM?

```c
DDRB |= (1 << PB0);    // PB0 jako wyjście
PORTB &= ~(1 << PB0);  // PB0 w stanie niskim
```

**Wyjaśnienie:**
- `&= ~(1 << PB0)` — zeruje bit PB0, nie ruszając pozostałych bitów

#### Co to jest stan wysokiej impedancji (Hi-Z)?

Stan wysokiej impedancji (Hi-Z) to domyślny stan pinu mikrokontrolera po resecie — pin jest ustawiony jako **wejście** (`DDR = 0`) bez podciągania (`PORT = 0`). W tym stanie pin "wisi w powietrzu" i jest podatny na zakłócenia elektromagnetyczne — może odczytywać losowe wartości.

**Jak go uniknąć?**

```c
// Metoda 1: Włączyć wewnętrzny pull-up
DDRB &= ~(1 << PB0);  // PB0 jako wejście
PORTB |= (1 << PB0);  // Włączenie pull-up (stan domyślny = HIGH)

// Metoda 2: Ustawić pin jako wyjście
DDRB |= (1 << PB0);   // PB0 jako wyjście
```

#### Co to są drgania styków?

Drgania styków (contact bounce) — zjawisko wielokrotnego zwarcia i rozwarcia styku przy jednym naciśnięciu przycisku. Trwa ok. 5-50 ms. Mikrokontroler jest tak szybki, że odczytuje te wielokrotne impulsy jako wiele naciśnięć.

**Jak sobie z nimi radzić?**

```c
// Metoda programowa — debouncing z opóźnieniem
if (!(PINC & (1 << PC0))) {     // Przycisk naciśnięty (aktywny LOW)
    _delay_ms(20);               // Czekamy 20ms aż drgania ustaną
    if (!(PINC & (1 << PC0))) {  // Sprawdzamy ponownie
        // Przycisk faktycznie naciśnięty — wykonaj akcję
    }
}
```

#### Jak ustawić PC0 jako wejście z PULL-UP?

```c
DDRC &= ~(1 << PC0);  // PC0 jako wejście (0 w DDR)
PORTC |= (1 << PC0);  // Włączenie wewnętrznego pull-up
```

**Wyjaśnienie:**
- Gdy `DDR = 0` (wejście) i `PORT = 1` → aktywny wewnętrzny rezystor pull-up (~20-50kΩ)
- Pin domyślnie odczytuje `1` (HIGH)
- Naciśnięcie przycisku (zwierającego do GND) daje `0` (LOW)
- Odczyt: `if (!(PINC & (1 << PC0)))` — prawda gdy przycisk naciśnięty

### Podsumowanie rejestrów portów

```
DDRx  — kierunek:  0 = wejście,  1 = wyjście
PORTx — wyjście:   0 = LOW,      1 = HIGH
                    (wejście:  0 = Hi-Z,     1 = pull-up)
PINx  — odczyt stanu pinu (tylko do odczytu)
```

| DDRx | PORTx | Tryb pinu |
|------|-------|-----------|
| 0 | 0 | Wejście Hi-Z (domyślne) |
| 0 | 1 | Wejście z pull-up |
| 1 | 0 | Wyjście LOW |
| 1 | 1 | Wyjście HIGH |

---

## 2. Generowanie dźwięku — buzzer

### Zadanie 2.1 — Sterowanie buzzerem

Buzzer na płytce EduTar-NANO jest **buzzerem pasywnym** — wymaga sygnału prostokątnego (zmiennego HIGH/LOW) o odpowiedniej częstotliwości. Podłączamy go do wybranego pinu i generujemy falę prostokątną.

**Sterowanie buzzerem — sygnał prostokątny:**

```c
// Buzzer podłączony np. do PB1
#define BUZZER PB1

void buzzer_on(uint16_t freq_hz, uint16_t duration_ms) {
    uint16_t period_us = 1000000UL / freq_hz;  // okres w us
    uint16_t half_period = period_us / 2;
    uint32_t cycles = (uint32_t)freq_hz * duration_ms / 1000;

    for (uint32_t i = 0; i < cycles; i++) {
        PORTB |= (1 << BUZZER);
        _delay_us(half_period);      // Uwaga: _delay_us wymaga stałej
        PORTB &= ~(1 << BUZZER);
        _delay_us(half_period);
    }
}
```

**Prostsza wersja z _delay_ms:**

```c
void beep(uint16_t duration_ms) {
    // Generuj ~1kHz przez zadany czas
    for (uint16_t i = 0; i < duration_ms; i++) {
        PORTB |= (1 << BUZZER);
        _delay_us(500);
        PORTB &= ~(1 << BUZZER);
        _delay_us(500);
    }
}
```

### Zadanie 2.2 — Odliczanie do 8 z LED i buzzerem

**Pełny program:**

```c
#include <avr/io.h>
#include <util/delay.h>

#define BUZZER PB1  // Dostosuj pin buzzera do płytki

// Krótki sygnał dźwiękowy (~100ms, ~1kHz)
void beep_short(void) {
    for (uint16_t i = 0; i < 100; i++) {
        PORTB |= (1 << BUZZER);
        _delay_us(500);
        PORTB &= ~(1 << BUZZER);
        _delay_us(500);
    }
}

// Przerywany sygnał dźwiękowy
void beep_alarm(uint16_t duration_s) {
    for (uint16_t s = 0; s < duration_s * 5; s++) {
        beep_short();
        _delay_ms(100);
    }
}

int main(void) {
    // === KONFIGURACJA ===
    DDRD = 0xFF;            // Port D jako wyjście (8 diod LED)
    DDRB |= (1 << BUZZER); // Pin buzzera jako wyjście
    PORTD = 0x00;           // Wszystkie LED wyłączone

    // === ODLICZANIE DO 8 ===
    for (uint8_t i = 0; i < 8; i++) {
        _delay_ms(1000);                  // Czekaj 1 sekundę

        PORTD = (PORTD << 1) | 0x01;     // Zapal kolejną diodę (narastający pasek)
        // Alternatywnie: PORTD |= (1 << i);

        beep_short();                     // Krótki sygnał dźwiękowy
    }

    // === MIGANIE + ALARM PRZEZ 5 SEKUND ===
    for (uint8_t j = 0; j < 25; j++) {   // 5s / 0.2s = 25 cykli
        PORTD = 0xFF;       // Wszystkie LED ON
        beep_short();       // Dźwięk
        _delay_ms(100);
        PORTD = 0x00;       // Wszystkie LED OFF
        _delay_ms(100);
    }

    // === KONIEC — program nic nie robi ===
    PORTD = 0x00;           // LED wyłączone
    while (1) {
        // Pusta pętla — czekaj na reset
    }

    return 0;
}
```

**Wyjaśnienie działania paska LED:**

```
Krok 0: PORTD = 0b00000000  (wszystkie OFF)
Krok 1: PORTD = 0b00000001  (1 LED)
Krok 2: PORTD = 0b00000011  (2 LED)
Krok 3: PORTD = 0b00000111  (3 LED)
...
Krok 8: PORTD = 0b11111111  (8 LED — wszystkie ON)
```

---

## 3. Wyświetlacz 7-segmentowy — 1 cyfra

### Zadanie 3.1 — Jak działa wyświetlacz 7-segmentowy?

Wyświetlacz 7-segmentowy składa się z 7 diod LED (segmenty a-g) + kropka (dp), ułożonych w kształt cyfry 8. Każdy segment jest sterowany osobnym pinem.

```
    aaaa
   f    b
   f    b
    gggg
   e    c
   e    c
    dddd  .dp
```

**Typy wyświetlaczy:**
- **Wspólna anoda (common anode)** — wspólny +, segmenty włączane stanem LOW (0)
- **Wspólna katoda (common cathode)** — wspólny GND, segmenty włączane stanem HIGH (1)

### Zadanie 3.2 — Sterowanie 1 cyfrą

**Tablica kodów dla wyświetlacza (wspólna katoda, segmenty na porcie D):**

Zakładamy mapowanie: PD0=a, PD1=b, PD2=c, PD3=d, PD4=e, PD5=f, PD6=g

```c
//                    gfedcba
// Cyfra 0: a,b,c,d,e,f ON   = 0b0111111 = 0x3F
// Cyfra 1: b,c ON            = 0b0000110 = 0x06
// Cyfra 2: a,b,d,e,g ON     = 0b1011011 = 0x5B
// itd.

const uint8_t tab[10] = {
    0x3F,  // 0 → segmenty: a,b,c,d,e,f
    0x06,  // 1 → segmenty: b,c
    0x5B,  // 2 → segmenty: a,b,d,e,g
    0x4F,  // 3 → segmenty: a,b,c,d,g
    0x66,  // 4 → segmenty: b,c,f,g
    0x6D,  // 5 → segmenty: a,c,d,f,g
    0x7D,  // 6 → segmenty: a,c,d,e,f,g
    0x07,  // 7 → segmenty: a,b,c
    0x7F,  // 8 → segmenty: a,b,c,d,e,f,g
    0x6F   // 9 → segmenty: a,b,c,d,f,g
};
```

**Dla wspólnej anody (odwrócone bity):**

```c
const uint8_t tab[10] = {
    ~0x3F & 0x7F,  // 0
    ~0x06 & 0x7F,  // 1
    ~0x5B & 0x7F,  // 2
    ~0x4F & 0x7F,  // 3
    ~0x66 & 0x7F,  // 4
    ~0x6D & 0x7F,  // 5
    ~0x7D & 0x7F,  // 6
    ~0x07 & 0x7F,  // 7
    ~0x7F & 0x7F,  // 8
    ~0x6F & 0x7F   // 9
};
```

**Pełny program — licznik 0-9:**

```c
#include <avr/io.h>
#include <util/delay.h>

// Tablica segmentów (wspólna katoda)
// Mapowanie: PD0=a, PD1=b, PD2=c, PD3=d, PD4=e, PD5=f, PD6=g
const uint8_t tab[10] = {
    0x3F, 0x06, 0x5B, 0x4F, 0x66,
    0x6D, 0x7D, 0x07, 0x7F, 0x6F
};

int main(void) {
    // === KONFIGURACJA ===
    DDRD = 0xFF;            // Port D (segmenty) jako wyjście
    DDRC |= (1 << PC0);    // PC0 (7SEG_S1) jako wyjście — cyfra 1

    PORTC |= (1 << PC0);   // Włącz cyfrę 1 (aktywuj wyświetlacz)

    // === PĘTLA GŁÓWNA ===
    while (1) {
        for (uint8_t i = 0; i <= 9; i++) {
            PORTD = tab[i];     // Wyświetl cyfrę
            _delay_ms(1000);    // Czekaj 1 sekundę
        }
    }

    return 0;
}
```

### Mapowanie segmentów — tabela referencyjna

```
Cyfra  | a | b | c | d | e | f | g | HEX  | BIN
-------|---|---|---|---|---|---|---|------|----------
  0    | 1 | 1 | 1 | 1 | 1 | 1 | 0 | 0x3F | 0111111
  1    | 0 | 1 | 1 | 0 | 0 | 0 | 0 | 0x06 | 0000110
  2    | 1 | 1 | 0 | 1 | 1 | 0 | 1 | 0x5B | 1011011
  3    | 1 | 1 | 1 | 1 | 0 | 0 | 1 | 0x4F | 1001111
  4    | 0 | 1 | 1 | 0 | 0 | 1 | 1 | 0x66 | 1100110
  5    | 1 | 0 | 1 | 1 | 0 | 1 | 1 | 0x6D | 1101101
  6    | 1 | 0 | 1 | 1 | 1 | 1 | 1 | 0x7D | 1111101
  7    | 1 | 1 | 1 | 0 | 0 | 0 | 0 | 0x07 | 0000111
  8    | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 0x7F | 1111111
  9    | 1 | 1 | 1 | 1 | 0 | 1 | 1 | 0x6F | 1101111
```

---

## 4. Wyświetlacz 7-segmentowy — 4 cyfry — multipleksowanie

### Zadanie 4.1 — Multipleksowanie

**Czym jest multipleksowanie?**
Multipleksowanie polega na szybkim przełączaniu między cyframi wyświetlacza — w danym momencie świeci tylko jedna cyfra, ale przy odpowiednio szybkim przełączaniu (>100Hz) ludzkie oko widzi wszystkie 4 cyfry jednocześnie.

**Podłączenie:**
- Segmenty (a-g) → Port D (wspólne dla wszystkich cyfr)
- Cyfra 1 (7SEG_S1) → PC0
- Cyfra 2 (7SEG_S2) → PC1
- Cyfra 3 (7SEG_S3) → PC2
- Cyfra 4 (7SEG_S4) → PC3

**Program wyświetlający 4-cyfrową liczbę:**

```c
#include <avr/io.h>
#include <util/delay.h>

const uint8_t tab[10] = {
    0x3F, 0x06, 0x5B, 0x4F, 0x66,
    0x6D, 0x7D, 0x07, 0x7F, 0x6F
};

// Wyświetl 4-cyfrową liczbę z multipleksowaniem
void display_number(uint16_t number, uint16_t duration_ms) {
    uint8_t digits[4];

    // Rozbij liczbę na poszczególne cyfry
    digits[0] = number / 1000;          // Tysiące
    digits[1] = (number / 100) % 10;    // Setki
    digits[2] = (number / 10) % 10;     // Dziesiątki
    digits[3] = number % 10;            // Jedności

    // Multipleksowanie przez zadany czas
    for (uint16_t t = 0; t < duration_ms / 20; t++) {
        for (uint8_t i = 0; i < 4; i++) {
            PORTC &= ~(0x0F);          // Wyłącz wszystkie cyfry
            PORTD = tab[digits[i]];    // Ustaw segmenty
            PORTC |= (1 << i);        // Włącz odpowiednią cyfrę
            _delay_ms(5);             // Czas świecenia jednej cyfry
        }
    }
}

int main(void) {
    DDRD = 0xFF;                        // Port D (segmenty) — wyjście
    DDRC |= (1 << PC0) | (1 << PC1)    // Cyfry 1-4 — wyjście
          | (1 << PC2) | (1 << PC3);

    uint16_t liczba = 1234;             // Dowolna 4-cyfrowa liczba

    while (1) {
        display_number(liczba, 1000);   // Wyświetlaj przez 1s
    }

    return 0;
}
```

### Zadanie 4.2 — Timer do gotowania

**Pełny program:**

```c
#include <avr/io.h>
#include <util/delay.h>

#define BUZZER PB1

// Przyciski
#define SW1 PC4   // Zwiększ czas (+1s)
#define SW2 PC5   // Zmniejsz czas (-1s)
#define SW3 PC6   // Kasuj / przerwij — dostosuj do płytki
#define SW4 PC7   // Start odliczania — dostosuj do płytki

// Alternatywnie, jeśli SW3/SW4 na innym porcie:
// #define SW3 PB2
// #define SW4 PB3

const uint8_t tab[10] = {
    0x3F, 0x06, 0x5B, 0x4F, 0x66,
    0x6D, 0x7D, 0x07, 0x7F, 0x6F
};

// Wyświetlanie multipleksowane (jedno przejście ~20ms)
void display_refresh(uint16_t number) {
    uint8_t digits[4];
    digits[0] = number / 1000;
    digits[1] = (number / 100) % 10;
    digits[2] = (number / 10) % 10;
    digits[3] = number % 10;

    for (uint8_t i = 0; i < 4; i++) {
        PORTC &= ~(0x0F);          // Wyłącz cyfry
        PORTD = tab[digits[i]];    // Ustaw segmenty
        PORTC |= (1 << i);        // Włącz cyfrę i
        _delay_ms(5);
    }
}

// Krótki beep
void beep_short(void) {
    for (uint16_t i = 0; i < 100; i++) {
        PORTB |= (1 << BUZZER);
        _delay_us(500);
        PORTB &= ~(1 << BUZZER);
        _delay_us(500);
    }
}

// Sprawdź przycisk z debouncingiem
uint8_t button_pressed(volatile uint8_t *pin_reg, uint8_t pin) {
    if (!(*pin_reg & (1 << pin))) {
        _delay_ms(20);
        if (!(*pin_reg & (1 << pin))) {
            while (!(*pin_reg & (1 << pin)));  // Czekaj na puszczenie
            return 1;
        }
    }
    return 0;
}

int main(void) {
    // === KONFIGURACJA ===
    DDRD = 0xFF;                          // Segmenty
    DDRC |= 0x0F;                        // Cyfry S1-S4 (PC0-PC3) jako wyjścia
    DDRC &= ~((1 << SW1) | (1 << SW2));  // Przyciski jako wejścia
    PORTC |= (1 << SW1) | (1 << SW2);    // Pull-up na przyciskach
    // Konfiguracja SW3, SW4 analogicznie

    DDRB |= (1 << BUZZER);               // Buzzer jako wyjście

    uint16_t czas_s = 0;     // Ustawiony czas w sekundach

    while (1) {
        // =========================================
        // TRYB USTAWIANIA CZASU
        // =========================================
        while (1) {
            display_refresh(czas_s);

            // SW1 — zwiększ czas
            if (button_pressed(&PINC, SW1)) {
                if (czas_s < 9999) czas_s++;
            }

            // SW2 — zmniejsz czas
            if (button_pressed(&PINC, SW2)) {
                if (czas_s > 0) czas_s--;
            }

            // SW3 — kasuj
            if (button_pressed(&PINC, SW3)) {
                czas_s = 0;
            }

            // SW4 — start odliczania
            if (button_pressed(&PINC, SW4) && czas_s > 0) {
                break;  // Przejdź do odliczania
            }
        }

        // =========================================
        // TRYB ODLICZANIA
        // =========================================
        while (czas_s > 0) {
            // Wyświetlaj przez ~1 sekundę (50 × 20ms = 1000ms)
            for (uint8_t t = 0; t < 50; t++) {
                display_refresh(czas_s);

                // Sprawdź SW3 — przerwij odliczanie
                if (button_pressed(&PINC, SW3)) {
                    czas_s = 0;
                    goto tryb_ustawiania;
                }
            }
            czas_s--;
        }

        // =========================================
        // ALARM — CZAS MINĄŁ
        // =========================================
        display_refresh(0);

        // Sygnał dźwiękowy przez 2 sekundy
        for (uint8_t a = 0; a < 10; a++) {
            beep_short();
            _delay_ms(100);
        }

        tryb_ustawiania:
        czas_s = 0;  // Powrót do trybu ustawiania
    }

    return 0;
}
```

---

## 5. Zadanie domowe — funkcja nieblokująca wyświetlacza

**Wymagania:**
- Zmienna globalna `uint16_t display_value` przechowuje liczbę do wyświetlenia
- Funkcja wywoływana cyklicznie z `main()` — bez `delay`!
- Każde wywołanie wyświetla jedną cyfrę (1→2→3→4→1→...)
- Zmienna `static` pamięta aktualnie wyświetlaną cyfrę

```c
#include <avr/io.h>

const uint8_t tab[10] = {
    0x3F, 0x06, 0x5B, 0x4F, 0x66,
    0x6D, 0x7D, 0x07, 0x7F, 0x6F
};

// Zmienna globalna — liczba do wyświetlenia
volatile uint16_t display_value = 0;

// Funkcja nieblokująca — wywołuj cyklicznie z main()
void display_multiplex(void) {
    static uint8_t current_digit = 0;  // Pamięta, którą cyfrę wyświetlamy

    uint8_t digits[4];
    digits[0] = display_value / 1000;
    digits[1] = (display_value / 100) % 10;
    digits[2] = (display_value / 10) % 10;
    digits[3] = display_value % 10;

    // Wyłącz wszystkie cyfry
    PORTC &= ~(0x0F);

    // Ustaw segmenty dla aktualnej cyfry
    PORTD = tab[digits[current_digit]];

    // Włącz odpowiednią cyfrę
    PORTC |= (1 << current_digit);

    // Przejdź do następnej cyfry (0→1→2→3→0→...)
    current_digit++;
    if (current_digit >= 4) {
        current_digit = 0;
    }
}

int main(void) {
    DDRD = 0xFF;      // Segmenty — wyjście
    DDRC |= 0x0F;     // Cyfry S1-S4 — wyjście

    display_value = 4567;  // Przykładowa liczba

    while (1) {
        display_multiplex();  // Wywołuj cyklicznie

        // Tu możesz robić inne rzeczy...
        // np. sprawdzać przyciski, liczyć czas, itp.

        // Małe opóźnienie — ~5ms daje ~50Hz na cyfrę
        // (w pełnym programie lepiej użyć timera)
        _delay_ms(5);
    }

    return 0;
}
```

**Wyjaśnienie modyfikatora `static`:**

```c
static uint8_t current_digit = 0;
```

- `static` w funkcji → zmienna zachowuje wartość między wywołaniami funkcji
- Bez `static` → zmienna byłaby zerowana przy każdym wywołaniu
- Inicjalizacja `= 0` wykonuje się tylko raz (przy pierwszym wywołaniu)

**Dlaczego to lepsze od wersji z `delay`?**
- Nie blokuje programu — `main()` może robić inne rzeczy
- Łatwo dodać obsługę przycisków, timera, komunikacji
- W prawdziwych projektach używa się przerwań timera zamiast `_delay_ms(5)`

---

## Przydatne operacje bitowe — ściąga

```c
// Ustawienie bitu (SET)
PORTB |= (1 << PB0);           // Ustaw bit 0

// Zerowanie bitu (CLEAR)
PORTB &= ~(1 << PB0);          // Wyzeruj bit 0

// Przełączenie bitu (TOGGLE)
PORTB ^= (1 << PB0);           // Odwróć bit 0

// Sprawdzenie bitu (TEST)
if (PINB & (1 << PB0)) { }     // Czy bit 0 jest 1?
if (!(PINB & (1 << PB0))) { }  // Czy bit 0 jest 0?

// Ustawienie wielu bitów
DDRB |= (1 << PB0) | (1 << PB1) | (1 << PB2);

// Cały port na raz
DDRD = 0xFF;    // Wszystkie piny portu D jako wyjścia
PORTD = 0x00;   // Wszystkie piny portu D w stan niski
```

---

## Konfiguracja projektu w Atmel Studio / Microchip Studio

```c
// Na początku każdego programu:
#define F_CPU 16000000UL    // Częstotliwość zegara 16MHz (Arduino Nano)
#include <avr/io.h>
#include <util/delay.h>

// Opcjonalnie:
#include <avr/interrupt.h>  // Jeśli używamy przerwań
#include <stdint.h>         // uint8_t, uint16_t, itd.
```

**Ustawienia projektu:**
- MCU: ATmega328P
- Częstotliwość: 16 MHz
- Optymalizacja: -Os (wymagane dla `_delay_ms`)

---

*Źródło zadań: Wojciech Tarnawski — Technika Cyfrowa i Mikroprocesorowa, Laboratorium 2*
