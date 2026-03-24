# Laboratorium 2 — Rezystancja, Prawo Ohma, Rezystor

## Kompletny przewodnik z rozwiązaniami

---

## Zagadnienia teoretyczne

### Prawo Ohma — wzór i zależności

Prawo Ohma opisuje zależność między napięciem (U), prądem (I) i rezystancją (R):

```
U = I × R
I = U / R
R = U / I
```

Gdzie:
- **U** — napięcie w woltach [V]
- **I** — natężenie prądu w amperach [A]
- **R** — rezystancja w omach [Ω]

---

### Tolerancja rezystora

**Co to jest?** Tolerancja rezystora określa maksymalne dopuszczalne odchylenie rzeczywistej wartości rezystancji od wartości nominalnej, wyrażone w procentach.

**Przykład: tolerancja 5% dla rezystora 1kΩ:**

```
Wartość nominalna:  1000 Ω
Tolerancja:         ±5%
Odchylenie:         1000 × 0.05 = ±50 Ω
Zakres rzeczywisty: 950 Ω  –  1050 Ω
```

Typowe tolerancje: ±1% (brązowy pasek), ±5% (złoty pasek), ±10% (srebrny pasek).

---

### Moc na rezystorze

**Wzory na moc:**

```
P = U × I
P = I² × R
P = U² / R
```

Gdzie **P** — moc w watach [W].

**Moc rezystora (moc znamionowa)** — to maksymalna moc, jaką rezystor może bezpiecznie rozpraszać w postaci ciepła bez ryzyka uszkodzenia. Typowe wartości: 1/8W, 1/4W, 1/2W, 1W, 2W, 5W.

**Tolerancja mocy (+20%):** Producenci stosują zapas bezpieczeństwa, rezystor oznaczony jako 1/4W może krótkotrwale wytrzymać do 0.3W, ale nie należy na tym polegać w projektowaniu.

---

## Zadanie 1 — Obliczenia mocy i prądu

### Schemat układu

```
    +---[R]---+
    |         |
   (+)       (-)
    |         |
    +---[V]---+

V = źródło napięcia
R = rezystor
```

Oznaczenia na schemacie: strzałka prądu I płynie od + do - (kierunek umowny), napięcie U mierzone na rezystorze.

---

### Przypadek 1: R = 470Ω, U = 5V

```
I = U / R = 5 / 470 = 0.01064 A ≈ 10.64 mA

P = U² / R = 25 / 470 = 0.05319 W ≈ 53.2 mW
```

**Wynik: I ≈ 10.64 mA, P ≈ 53.2 mW**
Rezystor 1/4W (250 mW) — **TAK, wystarczy** (53.2 mW << 250 mW)

---

### Przypadek 2: R = 470Ω, U = 12V

```
I = U / R = 12 / 470 = 0.02553 A ≈ 25.53 mA

P = U² / R = 144 / 470 = 0.30638 W ≈ 306.4 mW
```

**Wynik: I ≈ 25.53 mA, P ≈ 306.4 mW**
Rezystor 1/4W (250 mW) — **NIE, moc za duża!** (306.4 mW > 250 mW). Potrzebny rezystor min. 1/2W.

---

### Przypadek 3: R = 1kΩ, U = 24V

```
I = U / R = 24 / 1000 = 0.024 A = 24 mA

P = U² / R = 576 / 1000 = 0.576 W = 576 mW
```

**Wynik: I = 24 mA, P = 576 mW**
Rezystor 1/4W (250 mW) — **NIE, moc za duża!** (576 mW > 250 mW). Potrzebny rezystor min. 1W.

---

### Przypadek 4: R = 100kΩ, U = 230V

```
I = U / R = 230 / 100000 = 0.0023 A = 2.3 mA

P = U² / R = 52900 / 100000 = 0.529 W = 529 mW
```

**Wynik: I = 2.3 mA, P = 529 mW**
Rezystor 1/4W (250 mW) — **NIE, moc za duża!** (529 mW > 250 mW). Potrzebny rezystor min. 1W.

> **UWAGA:** Napięcie 230V to napięcie sieciowe — wymaga specjalnych rezystorów o odpowiednim napięciu znamionowym (nie tylko mocy!). Standardowe rezystory THT mają napięcie max. 200-500V.

---

### Podsumowanie Zadania 1

| R | U | I | P | Rezystor 1/4W? |
|---|---|---|---|-----------------|
| 470Ω | 5V | 10.64 mA | 53.2 mW | **TAK** |
| 470Ω | 12V | 25.53 mA | 306.4 mW | **NIE** (min. 1/2W) |
| 1kΩ | 24V | 24 mA | 576 mW | **NIE** (min. 1W) |
| 100kΩ | 230V | 2.3 mA | 529 mW | **NIE** (min. 1W) |

---

## Zadanie 2 — Pomiary i charakterystyka U/I

### Cel

Zbudować obwód z rezystorem i amperomierzem, zmierzyć prąd przy różnych napięciach, narysować charakterystykę U/I (która dla rezystora liniowego powinna być linią prostą).

### Schemat pomiarowy

```
    +---[A]---[R]---+
    |               |
   (+)             (-)
    |               |
    +----[V_zas]----+

A = amperomierz (szeregowo!)
R = rezystor
V_zas = zasilacz laboratoryjny (regulowany)

Woltomierz podłączamy RÓWNOLEGLE do rezystora.
```

---

### Pomiar 1: R = 100Ω, zakres 0–5V

**Obliczenie prądu maksymalnego (sprawdzenie bezpieczeństwa):**

```
I_max = U_max / R = 5 / 100 = 50 mA
P_max = U² / R = 25 / 100 = 250 mW = 1/4 W
```

Amperomierz na zakresie mA — 50 mA jest bezpieczne. Rezystor 1/4W — na granicy, ale OK.

**Przykładowa tabela pomiarowa:**

| U [V] | I obliczone [mA] |
|-------|-------------------|
| 0.0 | 0.0 |
| 0.5 | 5.0 |
| 1.0 | 10.0 |
| 1.5 | 15.0 |
| 2.0 | 20.0 |
| 2.5 | 25.0 |
| 3.0 | 30.0 |
| 3.5 | 35.0 |
| 4.0 | 40.0 |
| 4.5 | 45.0 |
| 5.0 | 50.0 |

**Wzór do obliczenia:** `I [mA] = U [V] / 100 [Ω] × 1000`

---

### Pomiar 2: R = 1kΩ, zakres 0–12V

**Obliczenie prądu maksymalnego:**

```
I_max = 12 / 1000 = 12 mA
P_max = 144 / 1000 = 144 mW
```

Bezpieczne dla amperomierza i rezystora 1/4W.

**Przykładowa tabela pomiarowa:**

| U [V] | I obliczone [mA] |
|-------|-------------------|
| 0 | 0.0 |
| 1 | 1.0 |
| 2 | 2.0 |
| 3 | 3.0 |
| 4 | 4.0 |
| 5 | 5.0 |
| 6 | 6.0 |
| 7 | 7.0 |
| 8 | 8.0 |
| 9 | 9.0 |
| 10 | 10.0 |
| 11 | 11.0 |
| 12 | 12.0 |

**Wzór:** `I [mA] = U [V] / 1000 [Ω] × 1000 = U [V]`

---

### Pomiar 3: R = 4.7kΩ, zakres 0–24V

**Obliczenie prądu maksymalnego:**

```
I_max = 24 / 4700 = 5.106 mA
P_max = 576 / 4700 = 122.6 mW
```

Bezpieczne dla amperomierza i rezystora 1/4W.

**Przykładowa tabela pomiarowa:**

| U [V] | I obliczone [mA] |
|-------|-------------------|
| 0 | 0.00 |
| 2 | 0.43 |
| 4 | 0.85 |
| 6 | 1.28 |
| 8 | 1.70 |
| 10 | 2.13 |
| 12 | 2.55 |
| 14 | 2.98 |
| 16 | 3.40 |
| 18 | 3.83 |
| 20 | 4.26 |
| 22 | 4.68 |
| 24 | 5.11 |

**Wzór:** `I [mA] = U [V] / 4700 [Ω] × 1000`

---

### Jak rysować charakterystykę U/I

Na wykresie:
- Oś X = napięcie U [V]
- Oś Y = prąd I [mA]
- Punkty pomiarowe połączone linią

Dla rezystora idealnego (liniowego) wykres to **linia prosta** przechodząca przez punkt (0,0). Nachylenie prostej = 1/R (konduktancja). Im mniejszy R, tym bardziej stroma linia.

---

## Zadanie 3 — Dobór rezystora

### Przypadek A: I = 15 mA, U = 10V

```
R = U / I = 10 / 0.015 = 666.67 Ω
```

**Najbliższe wartości z szeregu E24:**
- **680Ω** (najbliższa standardowa wartość)

Sprawdzenie z R = 680Ω:
```
I = 10 / 680 = 14.71 mA    (blisko 15 mA)
P = 10² / 680 = 147 mW     (OK dla 1/4W)
```

**Sprawdzenie mocy:**
```
P = U × I = 10 × 0.015 = 150 mW — OK dla 1/4W
```

---

### Przypadek B: I = 1.5 mA, U = 5V

```
R = U / I = 5 / 0.0015 = 3333.33 Ω
```

**Najbliższe wartości z szeregu E24:**
- **3.3kΩ** (najbliższa standardowa wartość)

Sprawdzenie z R = 3.3kΩ:
```
I = 5 / 3300 = 1.515 mA    (blisko 1.5 mA)
P = 25 / 3300 = 7.6 mW     (OK dla 1/4W)
```

**Sprawdzenie mocy:**
```
P = U × I = 5 × 0.0015 = 7.5 mW — OK dla 1/4W
```

---

### Podsumowanie Zadania 3

| Wymaganie | R obliczone | R dobrane (E24) | I rzeczywiste | P |
|-----------|------------|-----------------|---------------|---|
| 15 mA / 10V | 666.7 Ω | **680 Ω** | 14.71 mA | 147 mW |
| 1.5 mA / 5V | 3333 Ω | **3.3 kΩ** | 1.52 mA | 7.6 mW |

---

## Popularne szeregi rezystorów (do doboru)

**Szereg E12** (tolerancja ±10%):
```
1.0  1.2  1.5  1.8  2.2  2.7  3.3  3.9  4.7  5.6  6.8  8.2
```

**Szereg E24** (tolerancja ±5%):
```
1.0  1.1  1.2  1.3  1.5  1.6  1.8  2.0  2.2  2.4  2.7  3.0
3.3  3.6  3.9  4.3  4.7  5.1  5.6  6.2  6.8  7.5  8.2  9.1
```

Wartości powyższe mnożymy przez dekady: ×1, ×10, ×100, ×1k, ×10k, ×100k, ×1M.

---

## Szybki kalkulator — wzory do kopiowania

```
# Prawo Ohma
I = U / R
U = I * R
R = U / I

# Moc
P = U * I
P = I² * R        →  P = I*I*R
P = U² / R        →  P = U*U/R

# Tolerancja
R_min = R_nom * (1 - tolerancja/100)
R_max = R_nom * (1 + tolerancja/100)

# Dobór rezystora
R = U / I
```

---

## Kod Python — kalkulator do obliczeń laboratoryjnych

Poniższy skrypt pozwala szybko obliczyć wszystkie wartości z zadań:

```python
# === KALKULATOR LABORATORIUM 2 ===
# Prawo Ohma, moc, tolerancja, dobór rezystora

def prawo_ohma(U=None, I=None, R=None):
    """Oblicz brakującą wielkość z prawa Ohma."""
    if U is None:
        return I * R
    elif I is None:
        return U / R
    elif R is None:
        return U / I

def moc(U=None, I=None, R=None):
    """Oblicz moc na rezystorze."""
    if U is not None and I is not None:
        return U * I
    elif U is not None and R is not None:
        return U**2 / R
    elif I is not None and R is not None:
        return I**2 * R

def tolerancja(R_nom, tol_procent):
    """Oblicz zakres rezystancji z tolerancją."""
    odch = R_nom * tol_procent / 100
    return R_nom - odch, R_nom + odch

def czy_1_4W(P_wat):
    """Sprawdź czy rezystor 1/4W wystarczy."""
    return P_wat <= 0.25

# ==========================================
# ZADANIE 1 — Obliczenia
# ==========================================
print("=" * 50)
print("ZADANIE 1 — Obliczenia mocy i pradu")
print("=" * 50)

przypadki = [
    (470, 5),
    (470, 12),
    (1000, 24),
    (100000, 230),
]

for R, U in przypadki:
    I = prawo_ohma(U=U, R=R)
    P = moc(U=U, R=R)
    ok = "TAK" if czy_1_4W(P) else "NIE"
    print(f"\nR={R}Ω, U={U}V:")
    print(f"  I = {I*1000:.2f} mA")
    print(f"  P = {P*1000:.1f} mW")
    print(f"  Rezystor 1/4W wystarczy? {ok}")

# ==========================================
# TOLERANCJA
# ==========================================
print(f"\n{'=' * 50}")
print("TOLERANCJA — Rezystor 1k, 5%")
print("=" * 50)
r_min, r_max = tolerancja(1000, 5)
print(f"Zakres: {r_min:.0f} Ω – {r_max:.0f} Ω")

# ==========================================
# ZADANIE 3 — Dobór rezystora
# ==========================================
print(f"\n{'=' * 50}")
print("ZADANIE 3 — Dobor rezystora")
print("=" * 50)

# Przypadek A
R_a = prawo_ohma(U=10, I=0.015)
print(f"\na) I=15mA, U=10V → R = {R_a:.1f} Ω → dobieramy 680Ω")
print(f"   Sprawdzenie: I = {10/680*1000:.2f} mA, P = {moc(U=10, R=680)*1000:.1f} mW")

# Przypadek B
R_b = prawo_ohma(U=5, I=0.0015)
print(f"\nb) I=1.5mA, U=5V → R = {R_b:.1f} Ω → dobieramy 3.3kΩ")
print(f"   Sprawdzenie: I = {5/3300*1000:.2f} mA, P = {moc(U=5, R=3300)*1000:.1f} mW")

# ==========================================
# ZADANIE 2 — Generowanie tabel pomiarowych
# ==========================================
print(f"\n{'=' * 50}")
print("ZADANIE 2 — Tabele pomiarowe (wartosci teoretyczne)")
print("=" * 50)

pomiary = [
    (100, 0, 5, 0.5),
    (1000, 0, 12, 1),
    (4700, 0, 24, 2),
]

for R, U_min, U_max, krok in pomiary:
    print(f"\nR = {R}Ω, zakres {U_min}-{U_max}V:")
    print(f"  {'U [V]':>8} | {'I [mA]':>10} | {'P [mW]':>10}")
    print(f"  {'-'*8}-+-{'-'*10}-+-{'-'*10}")
    U = U_min
    while U <= U_max + 0.01:
        I_val = U / R * 1000  # mA
        P_val = U**2 / R * 1000  # mW
        print(f"  {U:8.1f} | {I_val:10.2f} | {P_val:10.2f}")
        U += krok
```

---

## Wskazówki praktyczne

**Podłączanie amperomierza:** Amperomierz zawsze podłączamy **szeregowo** (w obwód). Nigdy równolegle — spowoduje to zwarcie i uszkodzenie miernika.

**Podłączanie woltomierza:** Woltomierz podłączamy **równolegle** do elementu, na którym mierzymy napięcie.

**Zasilacz laboratoryjny:** Przed włączeniem ustaw napięcie na 0V. Powoli zwiększaj napięcie obserwując amperomierz — prąd nie powinien przekroczyć wartości obliczonej.

**Odczyt kodu paskowego rezystora (4 paski):**
```
Pasek 1: pierwsza cyfra
Pasek 2: druga cyfra
Pasek 3: mnożnik (liczba zer)
Pasek 4: tolerancja (złoty=5%, srebrny=10%, brązowy=1%)

Przykład: brązowy-czarny-czerwony-złoty = 1 0 ×100 = 1kΩ ±5%
```

---

*Źródło zadań: Wojciech Tarnawski — Elementy Elektroniki i Elektrotechniki, Laboratorium 2*
