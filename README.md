# 📻 rpiFMServer - Ecosistema de Radio FM - Proyecto Completo

Sistema completo de receptor de radio FM usando RDA5807M con Raspberry Pi, incluyendo backend API, cliente web, app móvil y addon de Kodi.

## 📋 Tabla de Contenidos

1. [Arquitectura del Sistema](#arquitectura-del-sistema)
2. [Hardware Requerido](#hardware-requerido)
3. [Estructura del Proyecto](#estructura-del-proyecto)
4. [Backend (Raspberry Pi)](#backend-raspberry-pi)
5. [Cliente Web](#cliente-web)
6. [App Móvil (React Native)](#app-móvil-react-native)
7. [Addon de Kodi](#addon-de-kodi)
8. [API Documentation](#api-documentation)
9. [Instalación y Configuración](#instalación-y-configuración)

---

## Arquitectura del Sistema

```
┌─────────────────────────────────────────────┐
│         Raspberry Pi (Backend Core)         │
│  ┌────────────────────────────────────┐    │
│  │  RDA5807M → USB Audio → FFmpeg     │    │
│  └────────────────────────────────────┘    │
│                    ↓                         │
│  ┌────────────────────────────────────┐    │
│  │    API REST (Flask/FastAPI)        │    │
│  │  - Control de frecuencia           │    │
│  │  - Streaming de audio              │    │
│  │  - WebSocket (estado en tiempo real)│   │
│  └────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
                    ↓
        ┌───────────┼───────────┐
        ↓           ↓           ↓
   ┌────────┐  ┌────────┐  ┌────────┐
   │  Web   │  │  Kodi  │  │ Mobile │
   │ Client │  │ Addon  │  │  App   │
   └────────┘  └────────┘  └────────┘
```

## Hardware Requerido

### Componentes principales:
- **Raspberry Pi** (3/4/Zero 2W)
- **RDA5807M** breakout board
- **PCM1808** - ADC I²S (para captura de audio digital)
- **Protoboard** 830 puntos
- **Antena FM** - Dos opciones:
  - **Opción A:** Antena simple (75cm de cable)
  - **Opción B:** Antena de TV existente (via cable coaxial) - **RECOMENDADO**
- Cables jumper para conexiones I²C e I²S
- Componentes electrónicos básicos (capacitores, resistencias)

### Conexiones:

**RDA5807M → Raspberry Pi (I²C):**
- SDIO (pin 8) → GPIO 2 (SDA) - pin 3
- SCLK (pin 7) → GPIO 3 (SCL) - pin 5
- VDD → 3.3V - pin 1 o 17
- GND → GND - cualquier pin GND

**RDA5807M → PCM1808 (Audio Analógico):**
- LOUT → [Capacitor 10µF] → LINL (entrada izquierda)
- ROUT → [Capacitor 10µF] → LINR (entrada derecha)
- GND común

**PCM1808 → Raspberry Pi (I²S):**
- BCK → GPIO 18 (pin 12)
- LRCK → GPIO 19 (pin 35)
- DOUT → GPIO 21 (pin 40)
- VDD → 3.3V o 5V
- GND → GND

**Antena:**
- **Opción A:** Cable de 75cm directamente a RFIN
- **Opción B:** Cable coaxial de infraestructura existente → RFIN (ver sección detallada abajo)

---

## Estructura del Proyecto

```
fmradio-ecosystem/
├── backend/                 # Servidor central (Raspberry Pi)
│   ├── app.py              # API principal
│   ├── radio_controller.py # Control del RDA5807M
│   ├── streaming.py        # Gestión de streaming
│   ├── presets.json        # Presets guardados
│   └── requirements.txt
│
├── web-client/             # Cliente web
│   ├── index.html
│   ├── app.js
│   └── styles.css
│
├── kodi-addon/             # Addon de Kodi
│   └── plugin.audio.fmradio/
│       ├── addon.xml
│       ├── default.py
│       ├── icon.png
│       ├── fanart.jpg
│       └── resources/
│           ├── settings.xml
│           └── lib/
│               ├── rda5807m.py
│               └── radio.py
│
├── mobile-app/             # App móvil (React Native)
│   ├── App.js
│   ├── package.json
│   ├── android/
│   └── ios/
│
└── docs/
    └── API.md
```

---

## 🔧 Hardware Detallado

### 📦 Lista Completa de Materiales (BOM)

| Cantidad | Componente | Especificación | Precio aprox. |
|----------|------------|----------------|---------------|
| 1 | Raspberry Pi | 3B+ o 4 | $35-55 |
| 1 | Módulo RDA5807M | Breakout board I²C | $3-5 |
| 1 | Módulo PCM1808 | ADC I²S 24-bit | $8-12 |
| 1 | Protoboard | 830 puntos | $3-5 |
| 2 | Capacitor electrolítico | 10µF, 25V | $0.20 |
| 2 | Resistencia | 10kΩ, 1/4W | $0.10 |
| 20 | Cables jumper | Macho-Macho | $2-3 |
| 1 | Fuente 5V 3A | Para Raspberry Pi | $8-10 |
| - | **Antena (elegir una opción)** | - | - |
| **Opción A** | Cable para antena simple | 75cm, alambre calibre 18-22 | $0.50 |
| **Opción B** | Kit cable coaxial (ver abajo) | Para conexión a antena TV | $15-30 |
| - | **TOTAL Opción A** | - | **~$60-90** |
| - | **TOTAL Opción B** | - | **~$75-120** |

### 🎯 Especificaciones de Componentes Clave

#### RDA5807M - Receptor FM
- **Chip:** RDA5807M/RDA5807FP
- **Voltaje:** 2.7V - 3.3V (⚠️ NO conectar a 5V)
- **Interfaz:** I²C (dirección 0x11)
- **Rango FM:** 87-108 MHz
- **Características:**
  - Soporte RDS/RBDS
  - Control de volumen digital (0-15)
  - Búsqueda automática
  - RSSI (indicador de señal)
  - Consumo: ~20-50mA

#### PCM1808 - ADC I²S
- **Chip:** Texas Instruments PCM1808
- **Resolución:** 24-bit, hasta 96kHz
- **Interfaz:** I²S (esclavo)
- **SNR:** 98dB
- **Voltaje:** 3.3V o 5V
- **Consumo:** ~5-10mA
- **Impedancia entrada:** >16kΩ

### 📐 Diagrama de Conexiones Completo

```
┌─────────────────────────────────────────────────────────────┐
│                      RASPBERRY PI                            │
│                                                              │
│  3.3V ═══════════════════════════════════════════╦═══════   │
│  GPIO 2 (SDA) ◄────────────────┐                 ║          │
│  GPIO 3 (SCL) ◄──────────┐     │                 ║          │
│  GPIO 18 (I2S BCK)  ─────┼─────┼────┐            ║          │
│  GPIO 19 (I2S LRCK) ─────┼─────┼────┼───┐        ║          │
│  GPIO 21 (I2S DOUT) ─────┼─────┼────┼───┼──┐     ║          │
│  GND ════════════════════╬═════╬════╬═══╬══╬═════╩══════    │
└──────────────────────────╬─────╬────╬───╬──╬────────────────┘
                           ║     │    │   │  │
                           ║     │    │   │  │
        ┌──────────────────╫─────┘    │   │  │
        │   RDA5807M       ║          │   │  │
        ├──────────────────╨──────────┘   │  │
        │ SCLK (I²C Clock) ────────────────┘  │
        │ SDIO (I²C Data)  ───────────────────┘
        │ VDD ══════════════════[3.3V]
        │ GND ══════════════════[GND]
        │                  
        │ LOUT ─────┬──[Cap 10µF]──┬───┐
        │ ROUT ──┐  │              │   │
        │ RFIN ─┐│  │              │   │
        └───────┼┼──┘      [R 10kΩ]   │
                ││              │      │
             [Antena]         GND     │
                │                     │
                │                     │
                │    ┌────────────────┴────────────┐
                │    │      PCM1808 (ADC)          │
                │    ├─────────────────────────────┤
                └────┼──[Cap 10µF]── LINL          │
                     │              [R 10kΩ]       │
                     │                │            │
                     ├── LINR ────────┴─── GND     │
                     │                             │
                     │ BCK  ───────────────────────┘
                     │ LRCK ─────────────────────┘
                     │ DOUT ───────────────────┘
                     │ VDD ════[3.3V/5V]
                     │ GND ════[GND]
                     │ FMT ────[GND] (I²S format)
                     │ MD0 ════[3.3V] (slave mode)
                     │ MD1 ────[GND] (256fs)
                     └─────────────────────────────┘
```

### 🔌 Montaje en Protoboard

```
                         PROTOBOARD
┌────────────────────────────────────────────────────────────┐
│  [Rail +3.3V]  ═══════════════════════════════════════     │
│                                                             │
│  RDA5807M                      PCM1808                      │
│  ┌──────────┐                  ┌─────────────┐            │
│  │ 1  VDD ○═╪═════════════════╪○ VDD         │            │
│  │ 2  GND ○─╪─────────────────╪○ GND         │            │
│  │ 3  SCLK○─┼─────────────────┼──────────────┼─→ GPIO 3   │
│  │ 4  SDIO○─┼─────────────────┼──────────────┼─→ GPIO 2   │
│  │ 5  RFIN○ │ [Antena]        │              │            │
│  │ 6  LOUT○─┼──[C1]───[R1]───╪○ LINL        │            │
│  │ 7  ROUT○─┼──[C2]───[R2]───╪○ LINR        │            │
│  │ 8  GPIO1○ │                │              │            │
│  └──────────┘ │                │ BCK  ○───────┼─→ GPIO 18 │
│               │                │ LRCK ○───────┼─→ GPIO 19 │
│    C1, C2: 10µF electrolíticos │ DOUT ○───────┼─→ GPIO 21 │
│    R1, R2: 10kΩ                │ FMT  ○───[GND]           │
│                                │ MD0  ○═══[3.3V]          │
│                                │ MD1  ○───[GND]           │
│                                └─────────────┘            │
│                                                             │
│  [Rail GND]  ════════════════════════════════════════      │
└────────────────────────────────────────────────────────────┘

Notas importantes:
• Capacitores electrolíticos: Respetar polaridad (+ hacia LOUT/ROUT)
• Cables I²C: Mantener cortos (<30cm) o usar twisted pair
• Cables I²S: Mantener cortos (<30cm)
• Verificar con multímetro antes de alimentar
```

### 📡 Opciones de Antena

#### Opción A: Antena Simple (Monopolo λ/4)

**Cálculo de longitud:**
- Frecuencia central FM: 98 MHz
- Longitud de onda: λ = c/f = 300/98 = 3.06 metros
- λ/4 = **75 cm** (longitud óptima)

**Construcción:**
```
Cable rígido 75cm (calibre 18-22 AWG)
      │
      │ Vertical para mejor recepción
      │
      ├──→ Conectar a RFIN
      │
     GND (referencia)
```

**Ventajas:**
- ✅ Construcción inmediata
- ✅ Costo mínimo ($0.50)
- ✅ Portátil
- ✅ Omnidireccional

**Desventajas:**
- ⚠️ Alcance limitado (10-30 km)
- ⚠️ Ganancia baja (0-2 dBi)
- ⚠️ Sensible a ubicación

#### Opción B: Antena de TV + Infraestructura Coaxial ⭐ RECOMENDADO

**¿Por qué usar la antena de TV existente?**
- ✅ **Ya instalada** - Aprovechar infraestructura
- ✅ **Mejor ubicación** - Típicamente en techo/altura
- ✅ **Mayor ganancia** - 8-15 dBi (profesional)
- ✅ **Mejor alcance** - 50-100+ km
- ✅ **Múltiples emisoras** - Mayor cantidad detectadas

**Arquitectura típica en casa:**
```
                        TECHO/EXTERIOR
                             │
                    ┌────────┴────────┐
                    │   ANTENA TV     │
                    │  (VHF/UHF/FM)   │
                    └────────┬────────┘
                             │
                      [Cable RG-6/RG-59]
                             │
                             ↓
                    ┌────────────────┐
                    │  DISTRIBUIDOR  │ ← Ubicar este
                    │   PRINCIPAL    │
                    │   (Splitter)   │
                    └────┬───┬───┬───┘
                         │   │   │
            ┌────────────┘   │   └────────────┐
            │                │                │
            ↓                ↓                ↓
      [Habitación 1]   [Habitación 2]   [Sala/Comedor]
            │                │                │
         [Toma]           [Toma]           [Toma]
          Coaxial         Coaxial          Coaxial
```

**Kit necesario para conexión coaxial:**
```
MATERIALES ADICIONALES (Opción B):
□ Divisor 2 vías (5-2400 MHz)          $8
□ Cable coaxial RG-6/RG-59 (según necesidad) $5-15
□ Conectores F machos (2-4 unidades)   $2-4
□ Crimpadora de conectores F           $10
□ Capacitor 100pF-1nF (protección)     $0.50
□ Acopladores F hembra-hembra (2×)     $3
────────────────────────────────────────────────
Total adicional: ~$30-45
```

### 🔧 Conexión de Cable Coaxial al RDA5807M

**Método 1: Punto de instalación cerca de toma existente**

```
Toma coaxial de pared
         │
         ↓
    [Conector F]
         │
         ↓
    ┌────────────┐
    │  DIVISOR   │
    │  2 vías    │
    └──┬─────┬───┘
       │     │
       │     └──→ TV / Otro dispositivo
       │
       ↓
   [Cable corto 1-3m]
       │
       ↓
   Raspberry Pi
```

**Método 2: Adaptación directa del cable coaxial**

```
PREPARACIÓN DEL CABLE:

Paso 1 - Cable coaxial RG-6/RG-59:
┌─────────────────────────────────┐
│ Negro - Cubierta PVC exterior   │
├─────────────────────────────────┤
│ Plateado - Malla (blindaje)     │ ← A GND
├─────────────────────────────────┤
│ Blanco - Dieléctrico (espuma)   │
├─────────────────────────────────┤
│ Cobre - Conductor central       │ ← A RFIN
└─────────────────────────────────┘


Paso 2 - Extremo para conector F (instalación estándar):
         Conector F
         ┌────────┐
  ───────┤        ├──→ Conductor sobresale
         └────────┘


Paso 3 - Extremo para RDA5807M:

       ←─── 2cm ───→
  ┌──────────────────┐
  │ Retirar cubierta │
  │ y malla          │
  └────────┬─────────┘
           │
           │ ←─ 5mm ─→
           ├─────────┐
           │Dieléctrico│
           └───┬─────┘
               │
          Conductor ──→ Soldar a RFIN
          
  Malla (aparte) ──────→ Soldar a GND
```

**Circuito de protección recomendado:**

```
Cable coaxial
     │
     ↓
┌────────────────┐
│   PROTECCIÓN   │
├────────────────┤
│                │
│  [100pF-1nF]   │ ← Capacitor bloquea DC
│                │
│  [Varistor     │ ← Protege contra picos
│   30V] (opc)   │
│                │
└────┬───────────┘
     │
     ├──→ Conductor central → RFIN (RDA5807M)
     │
     └──→ Malla → GND (RDA5807M)
```

**Implementación en protoboard:**

```
┌─────────────────────────────────────┐
│           PROTOBOARD                │
│                                     │
│  [Coax centro] ──[100pF]── RFIN    │
│                     │               │
│                 [Varistor]          │
│                     │               │
│  [Coax malla] ──────┴────── GND    │
│                                     │
└─────────────────────────────────────┘
```

### ⚡ Consideraciones de Alimentación

**Consumo total del sistema:**
```
Componente          Corriente Típica    Corriente Máxima
────────────────────────────────────────────────────────
Raspberry Pi 3B+    400mA (idle)        1.4A (carga)
Raspberry Pi 4      600mA (idle)        1.2A (carga)
RDA5807M            20mA                50mA
PCM1808             5mA                 10mA
────────────────────────────────────────────────────────
TOTAL TÍPICO        425-625mA           1.5-1.3A
TOTAL MÁXIMO        ~1.5A               ~2A

Fuente recomendada: 5V 3A (oficial Raspberry Pi)
```

**Distribución de voltajes:**
```
┌──────────────────┐
│  Fuente 5V 3A    │
└────────┬─────────┘
         │
    [USB-C/microUSB]
         │
    ┌────┴──────────┐
    │ Raspberry Pi  │
    │  Regulador    │
    │   interno     │
    └────┬──────────┘
         │
    ┌────┴─────┐
    │  3.3V    │ ←─── RDA5807M (50mA max)
    │  Rail    │ ←─── PCM1808 (10mA max)
    └──────────┘      Total: ~60mA
                      Margen suficiente ✓
```

**⚠️ ADVERTENCIAS:**
- ❌ NO conectar RDA5807M a 5V (solo 3.3V)
- ❌ NO conectar cargas pesadas al rail 3.3V
- ✅ Verificar polaridad de capacitores electrolíticos
- ✅ Verificar continuidad antes de alimentar

### 🧪 Proceso de Ensamblaje Paso a Paso

**Fase 1: Preparación (15 minutos)**
1. Inspeccionar todos los componentes
2. Verificar pines con multímetro
3. Preparar protoboard (limpiar, marcar rails)
4. Organizar cables por función (I²C, I²S, alimentación)

**Fase 2: Montaje de módulos (20 minutos)**
5. Insertar RDA5807M en protoboard
6. Insertar PCM1808 (separado 5-10 filas)
7. Instalar capacitores 10µF (verificar polaridad)
8. Instalar resistencias 10kΩ pull-down

**Fase 3: Cableado de alimentación (10 minutos)**
9. Conectar rails: +3.3V desde Raspberry Pi pin 1
10. Conectar rails: GND desde Raspberry Pi pin 6
11. Alimentar RDA5807M (VDD, GND)
12. Alimentar PCM1808 (VDD, GND)

**Fase 4: Bus I²C (10 minutos)**
13. SCLK → GPIO 3 (cable amarillo/verde)
14. SDIO → GPIO 2 (cable azul/blanco)

**Fase 5: Bus I²S (15 minutos)**
15. BCK → GPIO 18 (cable naranja)
16. LRCK → GPIO 19 (cable rojo)
17. DOUT → GPIO 21 (cable morado)
18. Configurar pines FMT, MD0, MD1 del PCM1808

**Fase 6: Audio y antena (15 minutos)**
19. Conectar LOUT → [Cap] → LINL
20. Conectar ROUT → [Cap] → LINR
21. Instalar antena (simple o coaxial)
22. Si es coaxial: agregar capacitor de protección

**Fase 7: Verificación pre-encendido (10 minutos)**
23. Verificar continuidad con multímetro
24. Verificar ausencia de cortocircuitos
25. Verificar polaridad de capacitores
26. Revisar checklist completo

**Tiempo total estimado: ~90 minutos**

### ✅ Checklist de Verificación Pre-Encendido

```
CONEXIONES ELÉCTRICAS:
□ Todas las tierras (GND) conectadas entre sí
□ 3.3V NO conectado accidentalmente a 5V
□ Polaridad correcta en capacitores electrolíticos
□ Sin puentes/cortocircuitos entre pines adyacentes
□ Rails de alimentación verificados con multímetro

BUS I²C:
□ SCLK conectado a GPIO 3 (SCL)
□ SDIO conectado a GPIO 2 (SDA)
□ No invertidos SDA/SCL

BUS I²S:
□ BCK conectado a GPIO 18
□ LRCK conectado a GPIO 19
□ DOUT conectado a GPIO 21
□ Configuración PCM1808 correcta (FMT=GND, MD0=VDD, MD1=GND)

AUDIO:
□ Capacitores de acoplamiento instalados (10µF)
□ Resistencias pull-down instaladas (10kΩ)
□ Conexiones LOUT/ROUT → LINL/LINR verificadas

ANTENA:
□ Antena conectada a RFIN
□ Si es coaxial: malla conectada a GND
□ Si usa protección: capacitor instalado

ALIMENTACIÓN:
□ Fuente 5V 3A disponible
□ Cable USB-C/microUSB en buen estado
```

### 🔬 Pruebas Iniciales

**Prueba 1: Detección I²C del RDA5807M**
```bash
# Habilitar I²C
sudo raspi-config
# Interface Options → I2C → Enable → Reboot

# Detectar dispositivos I²C
i2cdetect -y 1

# Salida esperada:
#      0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
# 00:          -- -- -- -- -- -- -- -- -- -- -- -- --
# 10: -- 11 -- -- -- -- -- -- -- -- -- -- -- -- -- --
#        ↑
#     RDA5807M encontrado en 0x11 ✓
```

**Prueba 2: Verificación I²S del PCM1808**
```bash
# Listar dispositivos de audio
arecord -l

# Salida esperada:
# **** List of CAPTURE Hardware Devices ****
# card 0: ... (bcm2835 ALSA)
# card 1: sndrpii2scard [snd_rpi_i2s_card], device 0: ...
#        ↑
#     PCM1808 detectado ✓

# Probar captura de 5 segundos
arecord -D hw:1,0 -f S32_LE -r 48000 -c 2 test.wav -d 5

# Verificar archivo creado
ls -lh test.wav
```

**Prueba 3: Test de Recepción FM**
```python
# test_hardware.py
from radio_controller import RadioController
import time

radio = RadioController()

print("=== TEST DE HARDWARE ===\n")

# Test 1: Sintonizar frecuencia conocida
print("Test 1: Sintonizando 95.0 MHz...")
radio.tune(95.0)
time.sleep(1)

# Test 2: Leer RSSI
rssi = radio.get_rssi()
print(f"RSSI: {rssi}/60")

if rssi > 25:
    print("✅ Recepción OK")
else:
    print("⚠️ Señal débil - revisar antena")

# Test 3: Escaneo rápido
print("\nTest 3: Escaneo rápido...")
count = 0
for freq in range(880, 1080, 5):  # Cada 0.5 MHz
    radio.tune(freq / 10.0)
    time.sleep(0.05)
    if radio.get_rssi() > 30:
        print(f"  Emisora en {freq/10.0:.1f} MHz")
        count += 1

print(f"\n✅ {count} emisoras detectadas")
```

### 🐛 Troubleshooting Hardware

| Problema | Posible Causa | Solución |
|----------|---------------|----------|
| RDA5807M no detectado en I²C | SDA/SCL invertidos | Verificar conexiones GPIO 2 y 3 |
| | Falta pull-up en I²C | Agregar resistencias 4.7kΩ a 3.3V |
| | Voltaje incorrecto | Verificar 3.3V, NO 5V |
| PCM1808 no captura audio | Pines I²S mal conectados | Verificar GPIO 18, 19, 21 |
| | Configuración incorrecta | Verificar FMT=GND, MD0=VDD, MD1=GND |
| | No hay señal del RDA5807M | Verificar capacitores de acoplamiento |
| Audio distorsionado | Capacitores con polaridad invertida | Verificar banda negativa del capacitor |
| | Volumen del RDA5807M muy alto | Reducir: `radio.set_volume(5)` |
| | Impedancia desajustada | Verificar resistencias pull-down |
| Señal débil/mucho ruido | Antena mal conectada | Verificar conexión a RFIN |
| | Interferencias | Agregar capacitor de protección |
| | Cable coaxial largo sin amplificador | Agregar amplificador de señal |
| Cortocircuito al encender | Puente en protoboard | Verificar con multímetro en modo continuidad |
| | Pin doblado tocando otro | Inspeccionar visualmente |

---

## Backend (Raspberry Pi)

### 1. app.py - API Principal

```python
from flask import Flask, Response, jsonify, request, send_from_directory
from flask_cors import CORS
from flask_socketio import SocketIO, emit
import json
import threading
import time
from radio_controller import RadioController
from streaming import StreamingManager

app = Flask(__name__)
CORS(app)  # Permitir CORS para app móvil y web
socketio = SocketIO(app, cors_allowed_origins="*")

# Inicializar controladores
radio = RadioController()
streaming = StreamingManager(radio)

# Estado global
state = {
    'frequency': 95.0,
    'volume': 10,
    'rssi': 0,
    'rds': None,
    'is_playing': False,
    'presets': []
}

def broadcast_state():
    """Enviar estado a todos los clientes conectados"""
    socketio.emit('state_update', state)

def update_state_loop():
    """Actualizar estado periódicamente"""
    while True:
        state['rssi'] = radio.get_rssi()
        state['rds'] = radio.get_rds_name()
        broadcast_state()
        time.sleep(2)

# Iniciar thread de actualización
threading.Thread(target=update_state_loop, daemon=True).start()

# ========== RUTAS DE LA API ==========

@app.route('/api/status')
def get_status():
    """Obtener estado actual"""
    return jsonify(state)

@app.route('/api/tune', methods=['POST'])
def tune():
    """Sintonizar frecuencia"""
    data = request.json
    freq = float(data.get('frequency', 95.0))
    
    if 87.0 <= freq <= 108.0:
        radio.tune(freq)
        state['frequency'] = freq
        state['is_playing'] = True
        broadcast_state()
        return jsonify({'status': 'ok', 'frequency': freq})
    
    return jsonify({'status': 'error', 'message': 'Invalid frequency'}), 400

@app.route('/api/volume', methods=['POST'])
def set_volume():
    """Ajustar volumen"""
    data = request.json
    volume = int(data.get('volume', 10))
    
    if 0 <= volume <= 15:
        radio.set_volume(volume)
        state['volume'] = volume
        broadcast_state()
        return jsonify({'status': 'ok', 'volume': volume})
    
    return jsonify({'status': 'error', 'message': 'Invalid volume'}), 400

@app.route('/api/seek', methods=['POST'])
def seek():
    """Buscar siguiente emisora"""
    data = request.json
    direction = data.get('direction', 'up')
    
    found_freq = radio.seek(direction)
    if found_freq:
        state['frequency'] = found_freq
        broadcast_state()
        return jsonify({'status': 'ok', 'frequency': found_freq})
    
    return jsonify({'status': 'error', 'message': 'No station found'}), 404

@app.route('/api/scan', methods=['GET'])
def scan_stations():
    """Escanear todas las emisoras disponibles"""
    stations = radio.scan_all()
    return jsonify({'stations': stations})

@app.route('/api/presets', methods=['GET'])
def get_presets():
    """Obtener presets guardados"""
    return jsonify({'presets': state['presets']})

@app.route('/api/presets', methods=['POST'])
def save_preset():
    """Guardar preset"""
    data = request.json
    preset = {
        'name': data.get('name'),
        'frequency': data.get('frequency')
    }
    state['presets'].append(preset)
    # Guardar en archivo
    with open('presets.json', 'w') as f:
        json.dump(state['presets'], f)
    broadcast_state()
    return jsonify({'status': 'ok', 'preset': preset})

@app.route('/api/presets/<int:index>', methods=['DELETE'])
def delete_preset(index):
    """Eliminar preset"""
    if 0 <= index < len(state['presets']):
        deleted = state['presets'].pop(index)
        with open('presets.json', 'w') as f:
            json.dump(state['presets'], f)
        broadcast_state()
        return jsonify({'status': 'ok', 'deleted': deleted})
    return jsonify({'status': 'error'}), 404

@app.route('/stream.mp3')
def stream():
    """Endpoint de streaming de audio"""
    return Response(
        streaming.generate(),
        mimetype='audio/mpeg',
        headers={
            'Cache-Control': 'no-cache',
            'Connection': 'keep-alive',
            'X-Content-Type-Options': 'nosniff'
        }
    )

@app.route('/stream.m3u')
def playlist():
    """Generar playlist M3U para compatibilidad"""
    host = request.host
    m3u = f"""#EXTM3U
#EXTINF:-1,FM Radio {state['frequency']} MHz
http://{host}/stream.mp3
"""
    return Response(m3u, mimetype='audio/x-mpegurl')

# ========== WEBSOCKET EVENTS ==========

@socketio.on('connect')
def handle_connect():
    """Cliente conectado"""
    emit('state_update', state)
    print('Cliente conectado')

@socketio.on('disconnect')
def handle_disconnect():
    """Cliente desconectado"""
    print('Cliente desconectado')

@socketio.on('tune')
def handle_tune(data):
    """Sintonizar vía WebSocket"""
    freq = float(data.get('frequency', 95.0))
    if 87.0 <= freq <= 108.0:
        radio.tune(freq)
        state['frequency'] = freq
        broadcast_state()

# ========== SERVIR CLIENTE WEB ==========

@app.route('/')
def serve_web_client():
    """Servir cliente web"""
    return send_from_directory('../web-client', 'index.html')

@app.route('/<path:path>')
def serve_static(path):
    """Servir archivos estáticos"""
    return send_from_directory('../web-client', path)

if __name__ == '__main__':
    # Cargar presets guardados
    try:
        with open('presets.json', 'r') as f:
            state['presets'] = json.load(f)
    except:
        pass
    
    # Iniciar servidor
    print("🎵 Servidor de Radio FM iniciado")
    print("📻 Web: http://0.0.0.0:5000")
    print("🎧 Stream: http://0.0.0.0:5000/stream.mp3")
    
    socketio.run(app, host='0.0.0.0', port=5000, debug=True)
```

### 2. radio_controller.py - Control del RDA5807M

```python
import smbus2
import time

class RadioController:
    """Controlador del RDA5807M"""
    
    def __init__(self, bus_num=1, address=0x11):
        self.bus = smbus2.SMBus(bus_num)
        self.address = address
        self.current_freq = 95.0
        self.init()
    
    def init(self):
        """Inicializar chip"""
        # Soft reset
        self.bus.write_i2c_block_data(self.address, 0x02, [0x00, 0x02])
        time.sleep(0.1)
        
        # Power up, enable output
        self.bus.write_i2c_block_data(self.address, 0x02, [0xC0, 0x01])
        time.sleep(0.5)
        
        # Configurar volumen inicial
        self.set_volume(10)
    
    def tune(self, freq_mhz):
        """Sintonizar frecuencia (87.0 - 108.0 MHz)"""
        if not 87.0 <= freq_mhz <= 108.0:
            return False
        
        # Calcular valor del canal
        channel = int((freq_mhz - 87.0) / 0.1)
        
        # Escribir frecuencia en registros
        high_byte = 0xC0 | ((channel >> 8) & 0x03)
        low_byte = channel & 0xFF
        
        self.bus.write_i2c_block_data(self.address, 0x03, [high_byte, low_byte])
        self.current_freq = freq_mhz
        time.sleep(0.1)
        return True
    
    def set_volume(self, volume):
        """Ajustar volumen (0-15)"""
        volume = max(0, min(15, volume))
        self.bus.write_i2c_block_data(self.address, 0x05, [0x80 | volume, 0x00])
    
    def get_rssi(self):
        """Obtener fuerza de señal (RSSI)"""
        try:
            data = self.bus.read_i2c_block_data(self.address, 0x0B, 2)
            rssi = data[1] >> 1
            return rssi
        except:
            return 0
    
    def has_signal(self, threshold=30):
        """Verificar si hay señal válida"""
        return self.get_rssi() > threshold
    
    def get_rds_name(self):
        """Obtener nombre RDS de la estación"""
        # TODO: Implementar lectura completa de RDS
        # Por ahora retorna None
        return None
    
    def seek(self, direction='up'):
        """Buscar siguiente emisora"""
        freq = self.current_freq
        step = 0.1 if direction == 'up' else -0.1
        
        for _ in range(200):  # Máximo 200 pasos
            freq += step
            
            # Wraparound
            if freq > 108.0:
                freq = 87.5
            elif freq < 87.5:
                freq = 108.0
            
            self.tune(freq)
            time.sleep(0.05)
            
            if self.has_signal():
                return round(freq, 1)
        
        return None
    
    def scan_all(self):
        """Escanear todas las frecuencias y retornar emisoras encontradas"""
        stations = []
        freq = 87.5
        
        while freq <= 108.0:
            self.tune(freq)
            time.sleep(0.05)
            
            if self.has_signal():
                rds = self.get_rds_name()
                stations.append({
                    'frequency': round(freq, 1),
                    'name': rds or f"FM {freq:.1f}",
                    'rssi': self.get_rssi()
                })
            
            freq += 0.1
        
        return stations
```

### 3. streaming.py - Gestión de Streaming

```python
import subprocess

class StreamingManager:
    """Gestor de streaming de audio"""
    
    def __init__(self, radio_controller):
        self.radio = radio_controller
        self.process = None
        self.start_ffmpeg()
    
    def start_ffmpeg(self):
        """Iniciar proceso FFmpeg para streaming"""
        cmd = [
            'ffmpeg',
            '-f', 'alsa',
            '-i', 'hw:1,0',  # Dispositivo de audio USB
            '-acodec', 'libmp3lame',
            '-b:a', '192k',
            '-ar', '44100',
            '-ac', '2',
            '-f', 'mp3',
            '-'
        ]
        
        self.process = subprocess.Popen(
            cmd,
            stdout=subprocess.PIPE,
            stderr=subprocess.DEVNULL
        )
    
    def generate(self):
        """Generar chunks de audio para streaming"""
        try:
            while True:
                chunk = self.process.stdout.read(4096)
                if not chunk:
                    break
                yield chunk
        except:
            pass
    
    def restart(self):
        """Reiniciar streaming"""
        if self.process:
            self.process.terminate()
        self.start_ffmpeg()
```

### 4. requirements.txt

```
Flask==3.0.0
Flask-CORS==4.0.0
Flask-SocketIO==5.3.0
python-socketio==5.10.0
smbus2==0.4.3
```

---

## Cliente Web

### 1. index.html

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>📻 Radio FM</title>
    <script src="https://cdn.socket.io/4.5.4/socket.io.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }
        
        .radio-container {
            background: rgba(255, 255, 255, 0.95);
            border-radius: 30px;
            padding: 40px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            max-width: 500px;
            width: 100%;
        }
        
        .radio-display {
            text-align: center;
            margin-bottom: 30px;
        }
        
        .frequency {
            font-size: 72px;
            font-weight: bold;
            color: #667eea;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
        }
        
        .station-name {
            font-size: 24px;
            color: #555;
            margin-top: 10px;
        }
        
        .signal-meter {
            margin: 20px 0;
        }
        
        .signal-bar {
            height: 30px;
            background: #e0e0e0;
            border-radius: 15px;
            overflow: hidden;
            position: relative;
        }
        
        .signal-level {
            height: 100%;
            background: linear-gradient(90deg, #4CAF50, #8BC34A);
            transition: width 0.5s ease;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-weight: bold;
        }
        
        .controls {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 15px;
            margin: 30px 0;
        }
        
        button {
            padding: 15px;
            border: none;
            border-radius: 15px;
            background: #667eea;
            color: white;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.3s;
        }
        
        button:hover {
            background: #5568d3;
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
        }
        
        button:active {
            transform: translateY(0);
        }
        
        button.large {
            grid-column: span 3;
            font-size: 18px;
            padding: 20px;
        }
        
        .tune-input {
            display: flex;
            gap: 10px;
            margin: 20px 0;
        }
        
        input[type="number"] {
            flex: 1;
            padding: 15px;
            border: 2px solid #ddd;
            border-radius: 15px;
            font-size: 18px;
            text-align: center;
        }
        
        .presets {
            margin-top: 30px;
        }
        
        .presets h3 {
            margin-bottom: 15px;
            color: #333;
        }
        
        .preset-list {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 10px;
        }
        
        .preset-btn {
            padding: 15px;
            background: #f5f5f5;
            border: 2px solid #ddd;
            border-radius: 10px;
            cursor: pointer;
            transition: all 0.3s;
            text-align: left;
        }
        
        .preset-btn:hover {
            background: #667eea;
            color: white;
            border-color: #667eea;
        }
        
        .preset-name {
            font-weight: bold;
        }
        
        .preset-freq {
            font-size: 14px;
            opacity: 0.8;
        }
        
        .audio-player {
            margin: 20px 0;
        }
        
        audio {
            width: 100%;
            border-radius: 10px;
        }
        
        .status {
            text-align: center;
            padding: 10px;
            border-radius: 10px;
            margin: 10px 0;
            font-weight: bold;
        }
        
        .status.connected {
            background: #4CAF50;
            color: white;
        }
        
        .status.disconnected {
            background: #f44336;
            color: white;
        }
    </style>
</head>
<body>
    <div class="radio-container">
        <div class="status" id="connectionStatus">Conectando...</div>
        
        <div class="radio-display">
            <div class="frequency" id="frequencyDisplay">95.0 MHz</div>
            <div class="station-name" id="stationName">FM Radio</div>
        </div>
        
        <div class="signal-meter">
            <div class="signal-bar">
                <div class="signal-level" id="signalLevel" style="width: 0%">
                    <span id="signalText">Sin señal</span>
                </div>
            </div>
        </div>
        
        <div class="audio-player">
            <audio id="audioPlayer" controls>
                <source id="audioSource" src="" type="audio/mpeg">
            </audio>
        </div>
        
        <div class="tune-input">
            <input type="number" id="freqInput" min="87.0" max="108.0" step="0.1" value="95.0">
            <button onclick="tuneManual()">📡 Sintonizar</button>
        </div>
        
        <div class="controls">
            <button onclick="seekDown()">⏮ Anterior</button>
            <button onclick="scanStations()">🔍 Escanear</button>
            <button onclick="seekUp()">⏭ Siguiente</button>
        </div>
        
        <div class="presets">
            <h3>📌 Emisoras guardadas</h3>
            <div class="preset-list" id="presetList"></div>
            <button class="large" onclick="savePreset()">💾 Guardar actual</button>
        </div>
    </div>
    
    <script src="app.js"></script>
</body>
</html>
```

### 2. app.js

```javascript
// Configuración
const API_URL = window.location.origin;
let socket;
let state = {};

// Inicializar
document.addEventListener('DOMContentLoaded', init);

function init() {
    // Conectar WebSocket
    socket = io(API_URL);
    
    socket.on('connect', () => {
        updateConnectionStatus(true);
        console.log('Conectado al servidor');
    });
    
    socket.on('disconnect', () => {
        updateConnectionStatus(false);
        console.log('Desconectado del servidor');
    });
    
    socket.on('state_update', (newState) => {
        state = newState;
        updateUI();
    });
    
    // Configurar reproductor de audio
    const audioSource = document.getElementById('audioSource');
    audioSource.src = `${API_URL}/stream.mp3?t=${Date.now()}`;
    
    // Cargar estado inicial
    loadStatus();
    loadPresets();
}

function updateConnectionStatus(connected) {
    const status = document.getElementById('connectionStatus');
    if (connected) {
        status.textContent = '🟢 Conectado';
        status.className = 'status connected';
    } else {
        status.textContent = '🔴 Desconectado';
        status.className = 'status disconnected';
    }
}

function updateUI() {
    // Actualizar frecuencia
    document.getElementById('frequencyDisplay').textContent = 
        `${state.frequency?.toFixed(1)} MHz`;
    
    // Actualizar nombre de estación (RDS)
    document.getElementById('stationName').textContent = 
        state.rds || 'FM Radio';
    
    // Actualizar señal
    const rssi = state.rssi || 0;
    const percent = Math.min((rssi / 60) * 100, 100);
    document.getElementById('signalLevel').style.width = `${percent}%`;
    document.getElementById('signalText').textContent = 
        rssi > 0 ? `Señal: ${rssi}/60` : 'Sin señal';
    
    // Actualizar input
    document.getElementById('freqInput').value = state.frequency?.toFixed(1);
}

async function loadStatus() {
    try {
        const response = await fetch(`${API_URL}/api/status`);
        state = await response.json();
        updateUI();
    } catch (error) {
        console.error('Error cargando estado:', error);
    }
}

async function tuneManual() {
    const freq = parseFloat(document.getElementById('freqInput').value);
    await tune(freq);
}

async function tune(frequency) {
    try {
        const response = await fetch(`${API_URL}/api/tune`, {
            method: 'POST',
            headers: {'Content-Type': 'application/json'},
            body: JSON.stringify({frequency})
        });
        
        if (response.ok) {
            // Recargar audio
            const audio = document.getElementById('audioPlayer');
            audio.load();
            audio.play();
        }
    } catch (error) {
        console.error('Error sintonizando:', error);
        alert('Error al sintonizar');
    }
}

async function seekUp() {
    try {
        const response = await fetch(`${API_URL}/api/seek`, {
            method: 'POST',
            headers: {'Content-Type': 'application/json'},
            body: JSON.stringify({direction: 'up'})
        });
        
        if (response.ok) {
            const audio = document.getElementById('audioPlayer');
            audio.load();
            audio.play();
        }
    } catch (error) {
        console.error('Error buscando:', error);
    }
}

async function seekDown() {
    try {
        const response = await fetch(`${API_URL}/api/seek`, {
            method: 'POST',
            headers: {'Content-Type': 'application/json'},
            body: JSON.stringify({direction: 'down'})
        });
        
        if (response.ok) {
            const audio = document.getElementById('audioPlayer');
            audio.load();
            audio.play();
        }
    } catch (error) {
        console.error('Error buscando:', error);
    }
}

async function scanStations() {
    if (!confirm('¿Escanear todas las emisoras? Esto puede tardar unos minutos.')) {
        return;
    }
    
    try {
        const response = await fetch(`${API_URL}/api/scan`);
        const data = await response.json();
        
        const stations = data.stations;
        if (stations.length > 0) {
            alert(`Se encontraron ${stations.length} emisoras`);
            // Actualizar presets con las encontradas
            await Promise.all(stations.map(station => 
                fetch(`${API_URL}/api/presets`, {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify(station)
                })
            ));
            loadPresets();
        } else {
            alert('No se encontraron emisoras');
        }
    } catch (error) {
        console.error('Error escaneando:', error);
    }
}

async function savePreset() {
    const name = prompt('Nombre de la emisora:', state.rds || `FM ${state.frequency?.toFixed(1)}`);
    if (!name) return;
    
    try {
        await fetch(`${API_URL}/api/presets`, {
            method: 'POST',
            headers: {'Content-Type': 'application/json'},
            body: JSON.stringify({
                name,
                frequency: state.frequency
            })
        });
        loadPresets();
    } catch (error) {
        console.error('Error guardando preset:', error);
    }
}

async function loadPresets() {
    try {
        const response = await fetch(`${API_URL}/api/presets`);
        const data = await response.json();
        
        const container = document.getElementById('presetList');
        container.innerHTML = '';
        
        data.presets.forEach((preset, index) => {
            const btn = document.createElement('div');
            btn.className = 'preset-btn';
            btn.innerHTML = `
                <div class="preset-name">${preset.name}</div>
                <div class="preset-freq">${preset.frequency.toFixed(1)} MHz</div>
            `;
            btn.onclick = () => tune(preset.frequency);
            container.appendChild(btn);
        });
    } catch (error) {
        console.error('Error cargando presets:', error);
    }
}
```

---

## App Móvil (React Native)

### 1. App.js

```javascript
import React, { useState, useEffect } from 'react';
import {
  SafeAreaView,
  StyleSheet,
  Text,
  View,
  TouchableOpacity,
  TextInput,
  FlatList,
  StatusBar,
} from 'react-native';
import { Audio } from 'expo-av';
import io from 'socket.io-client';

const API_URL = 'http://192.168.1.100:5000'; // Cambiar a IP de tu Raspberry Pi

export default function App() {
  const [frequency, setFrequency] = useState(95.0);
  const [rssi, setRssi] = useState(0);
  const [stationName, setStationName] = useState('FM Radio');
  const [presets, setPresets] = useState([]);
  const [sound, setSound] = useState();
  const [isPlaying, setIsPlaying] = useState(false);
  const [socket, setSocket] = useState(null);

  useEffect(() => {
    // Conectar WebSocket
    const newSocket = io(API_URL);
    setSocket(newSocket);

    newSocket.on('connect', () => {
      console.log('Conectado al servidor');
    });

    newSocket.on('state_update', (state) => {
      setFrequency(state.frequency);
      setRssi(state.rssi);
      setStationName(state.rds || 'FM Radio');
    });

    loadPresets();

    return () => {
      newSocket.disconnect();
      if (sound) {
        sound.unloadAsync();
      }
    };
  }, []);

  const loadPresets = async () => {
    try {
      const response = await fetch(`${API_URL}/api/presets`);
      const data = await response.json();
      setPresets(data.presets);
    } catch (error) {
      console.error('Error cargando presets:', error);
    }
  };

  const playAudio = async () => {
    try {
      if (sound) {
        await sound.unloadAsync();
      }

      const { sound: newSound } = await Audio.Sound.createAsync(
        { uri: `${API_URL}/stream.mp3?t=${Date.now()}` },
        { shouldPlay: true }
      );

      setSound(newSound);
      setIsPlaying(true);
    } catch (error) {
      console.error('Error reproduciendo audio:', error);
    }
  };

  const stopAudio = async () => {
    if (sound) {
      await sound.stopAsync();
      setIsPlaying(false);
    }
  };

  const tuneFrequency = async (freq) => {
    try {
      await fetch(`${API_URL}/api/tune`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ frequency: freq }),
      });

      if (isPlaying) {
        await playAudio();
      }
    } catch (error) {
      console.error('Error sintonizando:', error);
    }
  };

  const seek = async (direction) => {
    try {
      await fetch(`${API_URL}/api/seek`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ direction }),
      });

      if (isPlaying) {
        await playAudio();
      }
    } catch (error) {
      console.error('Error buscando:', error);
    }
  };

  const renderPreset = ({ item }) => (
    <TouchableOpacity
      style={styles.presetButton}
      onPress={() => tuneFrequency(item.frequency)}
    >
      <Text style={styles.presetName}>{item.name}</Text>
      <Text style={styles.presetFreq}>{item.frequency.toFixed(1)} MHz</Text>
    </TouchableOpacity>
  );

  return (
    <SafeAreaView style={styles.container}>
      <StatusBar barStyle="light-content" />
      
      <View style={styles.header}>
        <Text style={styles.title}>📻 Radio FM</Text>
      </View>

      <View style={styles.display}>
        <Text style={styles.frequency}>{frequency.toFixed(1)}</Text>
        <Text style={styles.mhz}>MHz</Text>
        <Text style={styles.stationName}>{stationName}</Text>
      </View>

      <View style={styles.signalMeter}>
        <View style={styles.signalBar}>
          <View
            style={[
              styles.signalLevel,
              { width: `${Math.min((rssi / 60) * 100, 100)}%` },
            ]}
          />
        </View>
        <Text style={styles.signalText}>Señal: {rssi}/60</Text>
      </View>

      <View style={styles.playerControls}>
        {isPlaying ? (
          <TouchableOpacity style={styles.playButton} onPress={stopAudio}>
            <Text style={styles.playButtonText}>⏸ PAUSAR</Text>
          </TouchableOpacity>
        ) : (
          <TouchableOpacity style={styles.playButton} onPress={playAudio}>
            <Text style={styles.playButtonText}>▶ REPRODUCIR</Text>
          </TouchableOpacity>
        )}
      </View>

      <View style={styles.tuneControls}>
        <TouchableOpacity style={styles.seekButton} onPress={() => seek('down')}>
          <Text style={styles.seekButtonText}>⏮</Text>
        </TouchableOpacity>

        <View style={styles.freqInput}>
          <TextInput
            style={styles.input}
            value={frequency.toString()}
            onChangeText={(text) => setFrequency(parseFloat(text) || 95.0)}
            keyboardType="numeric"
          />
          <TouchableOpacity
            style={styles.tuneButton}
            onPress={() => tuneFrequency(frequency)}
          >
            <Text style={styles.tuneButtonText}>OK</Text>
          </TouchableOpacity>
        </View>

        <TouchableOpacity style={styles.seekButton} onPress={() => seek('up')}>
          <Text style={styles.seekButtonText}>⏭</Text>
        </TouchableOpacity>
      </View>

      <View style={styles.presets}>
        <Text style={styles.presetsTitle}>Emisoras guardadas</Text>
        <FlatList
          data={presets}
          renderItem={renderPreset}
          keyExtractor={(item, index) => index.toString()}
          numColumns={2}
          contentContainerStyle={styles.presetsList}
        />
      </View>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#667eea',
  },
  header: {
    padding: 20,
    alignItems: 'center',
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    color: 'white',
  },
  display: {
    alignItems: 'center',
    padding: 30,
    backgroundColor: 'rgba(255,255,255,0.1)',
    margin: 20,
    borderRadius: 20,
  },
  frequency: {
    fontSize: 72,
    fontWeight: 'bold',
    color: 'white',
  },
  mhz: {
    fontSize: 24,
    color: 'white',
    opacity: 0.8,
  },
  stationName: {
    fontSize: 20,
    color: 'white',
    marginTop: 10,
  },
  signalMeter: {
    marginHorizontal: 20,
    marginBottom: 20,
  },
  signalBar: {
    height: 30,
    backgroundColor: 'rgba(255,255,255,0.2)',
    borderRadius: 15,
    overflow: 'hidden',
  },
  signalLevel: {
    height: '100%',
    backgroundColor: '#4CAF50',
  },
  signalText: {
    color: 'white',
    textAlign: 'center',
    marginTop: 5,
  },
  playerControls: {
    paddingHorizontal: 20,
    marginBottom: 20,
  },
  playButton: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 15,
    alignItems: 'center',
  },
  playButtonText: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#667eea',
  },
  tuneControls: {
    flexDirection: 'row',
    paddingHorizontal: 20,
    marginBottom: 20,
    alignItems: 'center',
  },
  seekButton: {
    width: 60,
    height: 60,
    backgroundColor: 'rgba(255,255,255,0.2)',
    borderRadius: 30,
    justifyContent: 'center',
    alignItems: 'center',
  },
  seekButtonText: {
    fontSize: 24,
    color: 'white',
  },
  freqInput: {
    flex: 1,
    flexDirection: 'row',
    marginHorizontal: 10,
  },
  input: {
    flex: 1,
    backgroundColor: 'white',
    padding: 15,
    borderTopLeftRadius: 15,
    borderBottomLeftRadius: 15,
    fontSize: 18,
    textAlign: 'center',
  },
  tuneButton: {
    backgroundColor: '#764ba2',
    padding: 15,
    borderTopRightRadius: 15,
    borderBottomRightRadius: 15,
    justifyContent: 'center',
    minWidth: 60,
  },
  tuneButtonText: {
    color: 'white',
    fontWeight: 'bold',
    textAlign: 'center',
  },
  presets: {
    flex: 1,
    paddingHorizontal: 20,
  },
  presetsTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: 'white',
    marginBottom: 10,
  },
  presetsList: {
    paddingBottom: 20,
  },
  presetButton: {
    flex: 1,
    backgroundColor: 'rgba(255,255,255,0.2)',
    padding: 15,
    margin: 5,
    borderRadius: 10,
  },
  presetName: {
    color: 'white',
    fontWeight: 'bold',
    fontSize: 16,
  },
  presetFreq: {
    color: 'white',
    opacity: 0.8,
    marginTop: 5,
  },
});
```

### 2. package.json

```json
{
  "name": "FMRadioApp",
  "version": "1.0.0",
  "main": "node_modules/expo/AppEntry.js",
  "scripts": {
    "start": "expo start",
    "android": "expo start --android",
    "ios": "expo start --ios"
  },
  "dependencies": {
    "expo": "~49.0.0",
    "expo-av": "~13.4.1",
    "react": "18.2.0",
    "react-native": "0.72.3",
    "socket.io-client": "^4.5.4"
  },
  "devDependencies": {
    "@babel/core": "^7.20.0"
  }
}
```

---

## Addon de Kodi

### 1. addon.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<addon id="plugin.audio.fmradio" 
       name="FM Radio Receiver" 
       version="1.0.0" 
       provider-name="TuNombre">
    <requires>
        <import addon="xbmc.python" version="3.0.0"/>
        <import addon="script.module.requests" version="2.25.1"/>
    </requires>
    <extension point="xbmc.python.pluginsource" library="default.py">
        <provides>audio</provides>
    </extension>
    <extension point="xbmc.addon.metadata">
        <summary lang="en_GB">FM Radio receiver for Raspberry Pi</summary>
        <description lang="en_GB">
            Listen to FM radio using RDA5807M chip connected to your Raspberry Pi.
            Features: frequency tuning, station search, presets, RDS support.
        </description>
        <platform>all</platform>
        <license>GPL-3.0</license>
        <assets>
            <icon>icon.png</icon>
            <fanart>fanart.jpg</fanart>
        </assets>
    </extension>
</addon>
```

### 2. default.py

```python
import sys
import xbmc
import xbmcgui
import xbmcplugin
import xbmcaddon
from urllib.parse import urlencode, parse_qsl

from resources.lib.rda5807m import RDA5807M
from resources.lib.radio import RadioManager

# Obtener handle del addon
addon_handle = int(sys.argv[1])
addon = xbmcaddon.Addon()

# Inicializar radio
radio = RadioManager()

def build_url(query):
    """Construir URL para navegación del addon"""
    return f"{sys.argv[0]}?{urlencode(query)}"

def list_stations():
    """Mostrar lista de emisoras guardadas"""
    xbmcplugin.setContent(addon_handle, 'songs')
    
    presets = radio.get_presets()
    
    for preset in presets:
        list_item = xbmcgui.ListItem(label=preset['name'])
        list_item.setInfo('music', {
            'title': preset['name'],
            'artist': f"{preset['freq']} MHz",
            'genre': 'Radio FM'
        })
        
        url = build_url({
            'action': 'play',
            'freq': preset['freq']
        })
        
        list_item.addContextMenuItems([
            ('Buscar emisoras', f'RunPlugin({build_url({"action": "seek"})})'),
            ('Sintonizar manualmente', f'RunPlugin({build_url({"action": "manual_tune"})})')
        ])
        
        xbmcplugin.addDirectoryItem(addon_handle, url, list_item, False)
    
    # Opciones adicionales
    list_item = xbmcgui.ListItem(label='[Sintonizar frecuencia...]')
    url = build_url({'action': 'manual_tune'})
    xbmcplugin.addDirectoryItem(addon_handle, url, list_item, False)
    
    list_item = xbmcgui.ListItem(label='[Buscar emisoras]')
    url = build_url({'action': 'seek'})
    xbmcplugin.addDirectoryItem(addon_handle, url, list_item, False)
    
    xbmcplugin.endOfDirectory(addon_handle)

def play_station(freq):
    """Reproducir una frecuencia"""
    try:
        radio.tune(float(freq))
        
        play_item = xbmcgui.ListItem(path=radio.get_stream_url())
        play_item.setInfo('music', {
            'title': f"FM {freq} MHz",
            'artist': 'Radio FM'
        })
        
        xbmcplugin.setResolvedUrl(addon_handle, True, listitem=play_item)
        
        xbmc.executebuiltin(f'Notification(Radio FM, Sintonizado en {freq} MHz, 3000)')
        
    except Exception as e:
        xbmc.log(f"Error reproduciendo: {str(e)}", xbmc.LOGERROR)
        xbmcgui.Dialog().notification(
            'Error',
            f'No se pudo sintonizar {freq} MHz',
            xbmcgui.NOTIFICATION_ERROR
        )

def manual_tune():
    """Diálogo para sintonizar manualmente"""
    dialog = xbmcgui.Dialog()
    freq = dialog.numeric(2, 'Ingrese frecuencia (MHz)', '95.0')
    
    if freq:
        freq = float(freq)
        if 87.0 <= freq <= 108.0:
            play_station(freq)
        else:
            dialog.notification('Error', 'Frecuencia fuera de rango', xbmcgui.NOTIFICATION_ERROR)

def seek_stations():
    """Buscar emisoras automáticamente"""
    dialog = xbmcgui.DialogProgress()
    dialog.create('Buscando emisoras', 'Escaneando frecuencias FM...')
    
    found_stations = []
    freq = 87.5
    
    while freq <= 108.0 and not dialog.iscanceled():
        percent = int(((freq - 87.5) / (108.0 - 87.5)) * 100)
        dialog.update(percent, f'Probando {freq:.1f} MHz...')
        
        radio.tune(freq)
        xbmc.sleep(100)
        
        if radio.has_signal():
            station_name = radio.get_rds_name() or f"FM {freq:.1f}"
            found_stations.append({
                'name': station_name,
                'freq': freq
            })
            dialog.update(percent, f'¡Encontrada! {station_name}')
            xbmc.sleep(500)
        
        freq += 0.1
    
    dialog.close()
    
    if found_stations:
        items = [f"{s['name']} - {s['freq']:.1f} MHz" for s in found_stations]
        selected = xbmcgui.Dialog().select('Emisoras encontradas', items)
        
        if selected >= 0:
            play_station(found_stations[selected]['freq'])
    else:
        xbmcgui.Dialog().notification('Búsqueda', 'No se encontraron emisoras', xbmcgui.NOTIFICATION_INFO)

def router(paramstring):
    """Router principal del addon"""
    params = dict(parse_qsl(paramstring))
    
    if not params:
        list_stations()
    else:
        action = params.get('action')
        
        if action == 'play':
            play_station(params['freq'])
        elif action == 'manual_tune':
            manual_tune()
        elif action == 'seek':
            seek_stations()

if __name__ == '__main__':
    router(sys.argv[2][1:])
```

### 3. resources/lib/rda5807m.py

```python
import smbus2
import time

class RDA5807M:
    """Driver para el chip RDA5807M"""
    
    def __init__(self, bus_num=1, address=0x11):
        self.bus = smbus2.SMBus(bus_num)
        self.address = address
        self.current_freq = 95.0
        
    def init(self):
        """Inicializar chip"""
        self.bus.write_i2c_block_data(self.address, 0x02, [0x00, 0x02])
        time.sleep(0.1)
        self.bus.write_i2c_block_data(self.address, 0x02, [0xC0, 0x01])
        time.sleep(0.5)
        self.set_volume(15)
        
    def tune(self, freq_mhz):
        """Sintonizar frecuencia"""
        if not 87.0 <= freq_mhz <= 108.0:
            return False
        
        channel = int((freq_mhz - 87.0) / 0.1)
        high_byte = 0xC0 | ((channel >> 8) & 0x03)
        low_byte = channel & 0xFF
        
        self.bus.write_i2c_block_data(self.address, 0x03, [high_byte, low_byte])
        self.current_freq = freq_mhz
        time.sleep(0.1)
        return True
    
    def set_volume(self, volume):
        """Ajustar volumen (0-15)"""
        volume = max(0, min(15, volume))
        self.bus.write_i2c_block_data(self.address, 0x05, [0x80 | volume, 0x00])
    
    def get_rssi(self):
        """Obtener fuerza de señal"""
        try:
            data = self.bus.read_i2c_block_data(self.address, 0x0B, 2)
            return data[1] >> 1
        except:
            return 0
    
    def has_signal(self, threshold=30):
        """Verificar si hay señal válida"""
        return self.get_rssi() > threshold
    
    def get_rds_data(self):
        """Leer datos RDS"""
        try:
            data = self.bus.read_i2c_block_data(self.address, 0x0C, 8)
            return None  # TODO: Implementar parseo RDS
        except:
            return None
```

### 4. resources/lib/radio.py

```python
import subprocess
import xbmcaddon
from .rda5807m import RDA5807M

class RadioManager:
    """Gestor de la radio FM"""
    
    def __init__(self):
        self.addon = xbmcaddon.Addon()
        self.radio = RDA5807M()
        self.radio.init()
        self.stream_port = 8765
        self.stream_process = None
        self.start_stream_server()
    
    def start_stream_server(self):
        """Iniciar servidor de streaming interno"""
        cmd = [
            'ffmpeg',
            '-f', 'alsa',
            '-i', self.addon.getSetting('audio_device') or 'hw:1,0',
            '-acodec', 'libmp3lame',
            '-b:a', '192k',
            '-f', 'mp3',
            '-listen', '1',
            f'http://127.0.0.1:{self.stream_port}/stream.mp3'
        ]
        
        self.stream_process = subprocess.Popen(
            cmd,
            stdout=subprocess.DEVNULL,
            stderr=subprocess.DEVNULL
        )
    
    def get_stream_url(self):
        """Obtener URL del stream"""
        return f'http://127.0.0.1:{self.stream_port}/stream.mp3'
    
    def tune(self, freq):
        """Sintonizar frecuencia"""
        self.radio.tune(freq)
    
    def has_signal(self):
        """Verificar señal"""
        return self.radio.has_signal()
    
    def get_rds_name(self):
        """Obtener nombre RDS"""
        return self.radio.get_rds_data()
    
    def get_presets(self):
        """Obtener emisoras guardadas"""
        presets = []
        for i in range(10):
            name = self.addon.getSetting(f'preset_{i}_name')
            freq = self.addon.getSetting(f'preset_{i}_freq')
            if name and freq:
                presets.append({
                    'name': name,
                    'freq': float(freq)
                })
        
        if not presets:
            presets = [
                {'name': 'FM 95.0', 'freq': 95.0},
                {'name': 'FM 98.5', 'freq': 98.5},
                {'name': 'FM 101.3', 'freq': 101.3},
            ]
        
        return presets
    
    def cleanup(self):
        """Limpiar recursos"""
        if self.stream_process:
            self.stream_process.terminate()
```

### 5. resources/settings.xml

```xml
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<settings>
    <category label="30000">
        <setting id="audio_device" type="text" label="30001" default="hw:1,0"/>
        <setting id="i2c_bus" type="number" label="30002" default="1"/>
        <setting id="i2c_address" type="text" label="30003" default="0x11"/>
    </category>
    
    <category label="30010">
        <setting id="preset_0_name" type="text" label="30011" default=""/>
        <setting id="preset_0_freq" type="number" label="30012" default="95.0"/>
        
        <setting id="preset_1_name" type="text" label="30011" default=""/>
        <setting id="preset_1_freq" type="number" label="30012" default="98.5"/>
    </category>
</settings>
```

---

## API Documentation

### Base URL
```
http://raspberry-pi-ip:5000
```

### REST Endpoints

#### GET /api/status
Obtener estado actual de la radio

**Response:**
```json
{
  "frequency": 95.0,
  "volume": 10,
  "rssi": 45,
  "rds": "Radio Rock",
  "is_playing": true,
  "presets": [...]
}
```

#### POST /api/tune
Sintonizar frecuencia

**Request Body:**
```json
{
  "frequency": 95.5
}
```

**Response:**
```json
{
  "status": "ok",
  "frequency": 95.5
}
```

#### POST /api/volume
Ajustar volumen

**Request Body:**
```json
{
  "volume": 12
}
```

#### POST /api/seek
Buscar siguiente emisora

**Request Body:**
```json
{
  "direction": "up"  // o "down"
}
```

#### GET /api/scan
Escanear todas las emisoras disponibles

**Response:**
```json
{
  "stations": [
    {
      "frequency": 95.0,
      "name": "Rock FM",
      "rssi": 45
    }
  ]
}
```

#### GET /api/presets
Obtener presets guardados

#### POST /api/presets
Guardar nuevo preset

**Request Body:**
```json
{
  "name": "Mi Emisora",
  "frequency": 95.5
}
```

#### DELETE /api/presets/{index}
Eliminar preset por índice

#### GET /stream.mp3
Stream de audio MP3 en tiempo real

#### GET /stream.m3u
Playlist M3U para reproductores compatibles

### WebSocket Events

#### Event: connect
Cliente conectado al servidor

#### Event: disconnect
Cliente desconectado del servidor

#### Event: state_update
Actualización de estado (emitido cada 2 segundos)

**Payload:**
```json
{
  "frequency": 95.0,
  "volume": 10,
  "rssi": 45,
  "rds": "Radio Rock",
  "is_playing": true,
  "presets": [...]
}
```

#### Event: tune
Sintonizar vía WebSocket

**Payload:**
```json
{
  "frequency": 95.5
}
```

---

## Instalación y Configuración

### 1. Backend (Raspberry Pi)

```bash
# Actualizar sistema
sudo apt-get update
sudo apt-get upgrade

# Instalar dependencias del sistema
sudo apt-get install -y python3-pip ffmpeg i2c-tools alsa-utils

# Habilitar I²C
sudo raspi-config
# Interface Options → I2C → Enable

# Habilitar I²S para PCM1808
# Editar /boot/config.txt
sudo nano /boot/config.txt

# Agregar estas líneas al final:
dtparam=i2c_arm=on
dtoverlay=i2s-mmap
dtoverlay=googlevoicehat-soundcard

# Guardar (Ctrl+O, Enter, Ctrl+X) y reiniciar
sudo reboot

# Después del reinicio, verificar I²C
i2cdetect -y 1
# Debería mostrar 0x11 (dirección del RDA5807M)

# Verificar I²S/Audio
arecord -l
# Debería listar el dispositivo PCM1808 (card 1)

# Clonar o crear el proyecto
mkdir -p ~/fmradio-ecosystem/backend
cd ~/fmradio-ecosystem/backend

# Crear archivos del backend (app.py, radio_controller.py, etc)
# Copiar código de las secciones anteriores

# Instalar dependencias Python
pip3 install -r requirements.txt

# Probar captura de audio del PCM1808
arecord -D hw:1,0 -f S32_LE -r 48000 -c 2 test.wav -d 5
aplay test.wav

# Si el audio se escucha bien, el hardware está OK

# Ejecutar servidor
python3 app.py
```

**Configuración del dispositivo de audio en streaming.py:**

Si tu PCM1808 aparece como `hw:1,0`:
```python
# En streaming.py, verificar que el dispositivo sea correcto:
cmd = [
    'ffmpeg',
    '-f', 'alsa',
    '-i', 'hw:1,0',  # ← Cambiar si tu dispositivo es diferente
    '-acodec', 'libmp3lame',
    '-b:a', '192k',
    '-ar', '44100',
    '-ac', '2',
    '-f', 'mp3',
    '-'
]
```

Para encontrar el número correcto:
```bash
# Listar dispositivos
arecord -l

# Probar captura
arecord -D hw:X,Y -f S32_LE -r 48000 -c 2 test.wav -d 3
# Donde X es el número de card e Y el número de device
```

### 2. Cliente Web

El cliente web ya está servido automáticamente por Flask en la ruta raíz `/`

Acceder desde cualquier navegador:
```
http://raspberry-pi-ip:5000
```

### 3. App Móvil

```bash
# Instalar Node.js y npm (si no están instalados)
# En tu máquina de desarrollo

# Instalar Expo CLI
npm install -g expo-cli

# Crear proyecto
cd ~/fmradio-ecosystem/mobile-app

# Copiar App.js y package.json

# Instalar dependencias
npm install

# IMPORTANTE: Editar App.js y cambiar API_URL a la IP de tu Raspberry Pi
# const API_URL = 'http://192.168.1.XXX:5000';

# Iniciar desarrollo
npm start

# En tu teléfono:
# 1. Instalar "Expo Go" desde Play Store o App Store
# 2. Escanear el QR que aparece en la terminal
# 3. La app se cargará en tu teléfono

# Para compilar APK/IPA (producción):
expo build:android
expo build:ios
```

### 4. Addon de Kodi

```bash
# En Raspberry Pi con Kodi instalado

# Crear directorio del addon
mkdir -p ~/.kodi/addons/plugin.audio.fmradio

# Copiar todos los archivos del addon
cp -r ~/fmradio-ecosystem/kodi-addon/plugin.audio.fmradio/* \
      ~/.kodi/addons/plugin.audio.fmradio/

# Reiniciar Kodi
sudo systemctl restart kodi

# O instalar desde zip:
# 1. Empaquetar addon
cd ~/fmradio-ecosystem/kodi-addon
zip -r plugin.audio.fmradio-1.0.0.zip plugin.audio.fmradio/

# 2. En Kodi:
# Settings → Add-ons → Install from zip file
# Seleccionar plugin.audio.fmradio-1.0.0.zip
```

### 5. Configurar Servicio Systemd (Opcional)

Para que el backend se inicie automáticamente al arrancar:

```bash
# Crear archivo de servicio
sudo nano /etc/systemd/system/fmradio.service
```

Contenido:
```ini
[Unit]
Description=FM Radio Streaming Service
After=network.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/fmradio-ecosystem/backend
ExecStart=/usr/bin/python3 /home/pi/fmradio-ecosystem/backend/app.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Habilitar y arrancar:
```bash
sudo systemctl daemon-reload
sudo systemctl enable fmradio
sudo systemctl start fmradio

# Ver estado
sudo systemctl status fmradio

# Ver logs
sudo journalctl -u fmradio -f
```

---

## Características del Ecosistema

✅ **Backend unificado** - API REST + WebSocket  
✅ **Cliente web responsive** - Actualizaciones en tiempo real  
✅ **App móvil nativa** - iOS y Android con React Native  
✅ **Addon de Kodi** - Integración completa con media center  
✅ **Streaming de audio** - MP3 a todos los clientes simultáneamente  
✅ **Sincronización de estado** - Todos los dispositivos ven el mismo estado  
✅ **Presets compartidos** - Emisoras guardadas disponibles en todos los clientes  
✅ **Búsqueda automática** - Escaneo de emisoras disponibles  
✅ **Soporte RDS** - Nombre de estación (cuando esté disponible)  
✅ **Medidor de señal** - Visualización en tiempo real de RSSI  
✅ **Control de volumen** - Ajuste desde cualquier cliente  
✅ **Arquitectura modular** - Fácil de extender y mantener  

---

## Troubleshooting

### El RDA5807M no se detecta en I2C
```bash
# Verificar conexiones físicas
# Verificar que I2C está habilitado
sudo raspi-config

# Probar diferentes direcciones
i2cdetect -y 1
```

### No se escucha audio
```bash
# Verificar tarjeta USB
arecord -l

# Probar captura manual
arecord -D hw:1,0 -f cd test.wav -d 5

# Verificar que FFmpeg está instalado
ffmpeg -version
```

### WebSocket no conecta desde app móvil
- Verificar que Raspberry Pi y teléfono están en la misma red
- Verificar firewall de Raspberry Pi
- Cambiar IP en la app móvil a la IP correcta de la Raspberry Pi

### Kodi addon no aparece
```bash
# Verificar que los archivos están en el lugar correcto
ls -la ~/.kodi/addons/plugin.audio.fmradio/

# Verificar logs de Kodi
tail -f ~/.kodi/temp/kodi.log
```

---

## 📊 Optimización de Señal con Cable Coaxial

Si elegiste usar la infraestructura de cable coaxial existente, aquí hay scripts útiles para optimizar tu instalación.

### Script 1: Comparación Antena Simple vs Coaxial

```python
# compare_antennas.py
"""
Compara la recepción entre antena simple y antena de TV vía coaxial
"""
from radio_controller import RadioController
import time
import json

radio = RadioController()

def test_antenna(antenna_name, test_freqs=None):
    """Probar recepción con una antena específica"""
    if test_freqs is None:
        test_freqs = [88.5, 91.0, 95.0, 98.5, 102.0, 105.5, 107.5]
    
    print(f"\n{'='*60}")
    print(f"PRUEBA: {antenna_name}")
    print(f"{'='*60}\n")
    
    results = []
    for freq in test_freqs:
        radio.tune(freq)
        time.sleep(0.5)
        rssi = radio.get_rssi()
        
        status = "✅" if rssi > 40 else "⚠️" if rssi > 25 else "❌"
        print(f"{freq:6.1f} MHz: RSSI {rssi:2d}/60  {status}")
        
        results.append({
            'frequency': freq,
            'rssi': rssi
        })
    
    avg_rssi = sum(r['rssi'] for r in results) / len(results)
    print(f"\n📊 Promedio RSSI: {avg_rssi:.1f}/60")
    
    return results

def main():
    print("\n" + "="*60)
    print("  COMPARACIÓN DE ANTENAS - Radio FM RDA5807M")
    print("="*60)
    
    input("\n1️⃣  Conecta la ANTENA SIMPLE (75cm) y presiona Enter...")
    results_simple = test_antenna("Antena Simple 75cm")
    
    input("\n2️⃣  Conecta la ANTENA DE TV (coaxial) y presiona Enter...")
    results_coax = test_antenna("Antena TV via Coaxial")
    
    # Análisis comparativo
    print(f"\n{'='*60}")
    print("📈 ANÁLISIS COMPARATIVO")
    print(f"{'='*60}\n")
    
    print(f"{'Frecuencia':<12} {'Simple':<8} {'Coaxial':<8} {'Mejora':<10}")
    print("-" * 60)
    
    improvements = []
    for i, freq in enumerate([r['frequency'] for r in results_simple]):
        simple_rssi = results_simple[i]['rssi']
        coax_rssi = results_coax[i]['rssi']
        diff = coax_rssi - simple_rssi
        
        symbol = "📈" if diff > 5 else "➡️" if abs(diff) <= 5 else "📉"
        print(f"{freq:6.1f} MHz   {simple_rssi:2d}/60    {coax_rssi:2d}/60    "
              f"{diff:+3d} dB {symbol}")
        
        if simple_rssi > 0:
            improvements.append(diff)
    
    # Estadísticas
    avg_simple = sum(r['rssi'] for r in results_simple) / len(results_simple)
    avg_coax = sum(r['rssi'] for r in results_coax) / len(results_coax)
    avg_improvement = sum(improvements) / len(improvements) if improvements else 0
    
    print(f"\n{'='*60}")
    print("📊 RESUMEN")
    print(f"{'='*60}")
    print(f"RSSI Promedio - Antena Simple:  {avg_simple:5.1f}/60")
    print(f"RSSI Promedio - Antena Coaxial: {avg_coax:5.1f}/60")
    print(f"Mejora promedio:                {avg_improvement:+5.1f} dB")
    print(f"Mejora porcentual:              {(avg_improvement/avg_simple)*100:+5.1f}%")
    
    # Recomendación
    print(f"\n{'='*60}")
    if avg_improvement > 5:
        print("✅ RECOMENDACIÓN: Usar antena de TV (mejora significativa)")
    elif avg_improvement > 0:
        print("⚠️  RECOMENDACIÓN: Antena de TV ligeramente mejor")
    else:
        print("❌ RECOMENDACIÓN: Mantener antena simple (coaxial no mejora)")
    print(f"{'='*60}\n")
    
    # Guardar resultados
    report = {
        'simple': results_simple,
        'coaxial': results_coax,
        'summary': {
            'avg_simple': avg_simple,
            'avg_coaxial': avg_coax,
            'improvement_db': avg_improvement,
            'improvement_percent': (avg_improvement/avg_simple)*100 if avg_simple > 0 else 0
        }
    }
    
    with open('antenna_comparison.json', 'w') as f:
        json.dump(report, f, indent=2)
    
    print("📄 Resultados guardados en: antenna_comparison.json\n")

if __name__ == '__main__':
    main()
```

### Script 2: Mapeo de Ubicaciones

```python
# signal_mapping.py
"""
Mapea la calidad de señal en diferentes ubicaciones de la casa
usando la infraestructura de cable coaxial
"""
from radio_controller import RadioController
import time
import json
from datetime import datetime

radio = RadioController()

def scan_location(location_name, description=""):
    """Escanear emisoras en una ubicación específica"""
    print(f"\n{'='*60}")
    print(f"📍 UBICACIÓN: {location_name}")
    if description:
        print(f"   {description}")
    print(f"{'='*60}\n")
    
    stations = []
    print("Escaneando banda FM (87.5 - 108.0 MHz)...")
    print("Progreso: ", end='', flush=True)
    
    total_steps = (1080 - 875) // 2  # Cada 0.2 MHz
    progress_marks = 20
    
    for i, freq_int in enumerate(range(875, 1081, 2)):
        freq = freq_int / 10.0
        radio.tune(freq)
        time.sleep(0.03)
        
        rssi = radio.get_rssi()
        if rssi > 30:  # Umbral de señal válida
            rds = radio.get_rds_name()
            stations.append({
                'frequency': freq,
                'rssi': rssi,
                'rds': rds
            })
        
        # Barra de progreso
        if i % (total_steps // progress_marks) == 0:
            print("█", end='', flush=True)
    
    print(" ✓")
    
    # Resultados
    if stations:
        print(f"\n✅ {len(stations)} emisoras encontradas:")
        for station in stations[:5]:  # Mostrar solo las primeras 5
            print(f"   {station['frequency']:6.1f} MHz - RSSI: {station['rssi']:2d}/60")
        if len(stations) > 5:
            print(f"   ... y {len(stations) - 5} más")
        
        avg_rssi = sum(s['rssi'] for s in stations) / len(stations)
        print(f"\n📊 RSSI Promedio: {avg_rssi:.1f}/60")
    else:
        print("\n❌ No se encontraron emisoras")
        avg_rssi = 0
    
    return {
        'name': location_name,
        'description': description,
        'timestamp': datetime.now().isoformat(),
        'stations_count': len(stations),
        'avg_rssi': avg_rssi,
        'stations': stations
    }

def main():
    print("\n" + "="*60)
    print("  MAPEO DE SEÑAL - Infraestructura Coaxial")
    print("="*60)
    print("\nEste script te ayuda a encontrar la mejor ubicación")
    print("para tu Raspberry Pi usando tu sistema de cable coaxial.\n")
    
    locations = {}
    
    # Definir ubicaciones a probar
    print("Ingresa las ubicaciones que deseas probar.")
    print("Ejemplos: 'Sala', 'Habitación principal', 'Estudio', etc.")
    print("Deja en blanco para terminar.\n")
    
    location_number = 1
    while True:
        location_name = input(f"Ubicación #{location_number} (Enter para terminar): ").strip()
        if not location_name:
            break
        
        description = input(f"Descripción opcional (ej: 'toma cerca de TV'): ").strip()
        
        input(f"\n➡️  Conecta el Raspberry Pi en '{location_name}' y presiona Enter...")
        
        result = scan_location(location_name, description)
        locations[location_name] = result
        
        location_number += 1
    
    if not locations:
        print("\n⚠️  No se probó ninguna ubicación.")
        return
    
    # Análisis comparativo
    print(f"\n{'='*60}")
    print("📊 COMPARACIÓN DE UBICACIONES")
    print(f"{'='*60}\n")
    
    print(f"{'Ubicación':<25} {'Emisoras':<10} {'RSSI Prom.':<12}")
    print("-" * 60)
    
    for name, data in locations.items():
        print(f"{name:<25} {data['stations_count']:<10} {data['avg_rssi']:5.1f}/60")
    
    # Determinar mejor ubicación
    best_location = max(locations.items(), 
                       key=lambda x: x[1]['avg_rssi'])
    
    print(f"\n{'='*60}")
    print("🏆 MEJOR UBICACIÓN")
    print(f"{'='*60}")
    print(f"Ubicación:       {best_location[0]}")
    print(f"Emisoras:        {best_location[1]['stations_count']}")
    print(f"RSSI Promedio:   {best_location[1]['avg_rssi']:.1f}/60")
    if best_location[1]['description']:
        print(f"Descripción:     {best_location[1]['description']}")
    print(f"{'='*60}\n")
    
    # Guardar resultados
    report = {
        'test_date': datetime.now().isoformat(),
        'locations': locations,
        'best_location': best_location[0]
    }
    
    filename = f"signal_map_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
    with open(filename, 'w') as f:
        json.dump(report, f, indent=2)
    
    print(f"📄 Resultados guardados en: {filename}\n")

if __name__ == '__main__':
    main()
```

### Script 3: Test de Calidad de Cable

```python
# cable_quality_test.py
"""
Evalúa la calidad del cable coaxial y detecta problemas
"""
from radio_controller import RadioController
import time

radio = RadioController()

def test_cable_quality():
    """Prueba exhaustiva de calidad del cable coaxial"""
    print("\n" + "="*60)
    print("  TEST DE CALIDAD DE CABLE COAXIAL")
    print("="*60 + "\n")
    
    # Test 1: Estabilidad de señal
    print("Test 1: Estabilidad de señal")
    print("-" * 40)
    
    test_freq = 95.0
    radio.tune(test_freq)
    
    readings = []
    print(f"Tomando 20 lecturas en {test_freq} MHz...")
    for i in range(20):
        time.sleep(0.5)
        rssi = radio.get_rssi()
        readings.append(rssi)
        print(f"  Lectura {i+1:2d}: {rssi:2d}/60")
    
    avg = sum(readings) / len(readings)
    variance = sum((x - avg) ** 2 for x in readings) / len(readings)
    std_dev = variance ** 0.5
    
    print(f"\n  Promedio:          {avg:.2f}/60")
    print(f"  Desviación estándar: {std_dev:.2f}")
    
    if std_dev < 2:
        print("  ✅ Señal muy estable")
    elif std_dev < 5:
        print("  ⚠️  Señal moderadamente estable")
    else:
        print("  ❌ Señal inestable - revisar conexiones")
    
    # Test 2: Pérdida por frecuencia
    print(f"\n\nTest 2: Pérdida por frecuencia")
    print("-" * 40)
    
    test_frequencies = [88.0, 92.0, 96.0, 100.0, 104.0, 108.0]
    freq_results = []
    
    for freq in test_frequencies:
        radio.tune(freq)
        time.sleep(0.5)
        rssi = radio.get_rssi()
        freq_results.append((freq, rssi))
        print(f"  {freq:6.1f} MHz: {rssi:2d}/60")
    
    # Análisis de uniformidad
    rssi_values = [r[1] for r in freq_results]
    freq_avg = sum(rssi_values) / len(rssi_values)
    freq_range = max(rssi_values) - min(rssi_values)
    
    print(f"\n  Rango de variación: {freq_range} dB")
    
    if freq_range < 5:
        print("  ✅ Respuesta uniforme en frecuencia")
    elif freq_range < 10:
        print("  ⚠️  Respuesta moderada - cable aceptable")
    else:
        print("  ❌ Alta variación - revisar cable/conectores")
    
    # Test 3: Detección de ruido
    print(f"\n\nTest 3: Nivel de ruido")
    print("-" * 40)
    
    # Sintonizar frecuencia sin emisora
    empty_freq = 87.6  # Típicamente vacía
    radio.tune(empty_freq)
    time.sleep(0.5)
    noise_rssi = radio.get_rssi()
    
    print(f"  Señal en frecuencia vacía ({empty_freq} MHz): {noise_rssi}/60")
    
    if noise_rssi < 15:
        print("  ✅ Nivel de ruido bajo")
    elif noise_rssi < 25:
        print("  ⚠️  Ruido moderado")
    else:
        print("  ❌ Alto nivel de ruido - revisar blindaje")
    
    # Resumen final
    print(f"\n{'='*60}")
    print("📋 RESUMEN")
    print(f"{'='*60}")
    
    score = 0
    if std_dev < 2: score += 35
    elif std_dev < 5: score += 20
    
    if freq_range < 5: score += 35
    elif freq_range < 10: score += 20
    
    if noise_rssi < 15: score += 30
    elif noise_rssi < 25: score += 15
    
    print(f"Puntuación de calidad: {score}/100")
    
    if score >= 80:
        print("✅ Excelente - Cable en óptimas condiciones")
    elif score >= 60:
        print("⚠️  Bueno - Cable funcional, considerar mejoras")
    elif score >= 40:
        print("⚠️  Regular - Revisar conectores y longitud")
    else:
        print("❌ Pobre - Reemplazar cable o usar amplificador")
    
    print(f"{'='*60}\n")
    
    # Recomendaciones
    print("💡 RECOMENDACIONES:\n")
    
    if std_dev >= 5:
        print("   • Revisar todas las conexiones (apretar conectores F)")
        print("   • Verificar que no haya cables pellizcados")
    
    if freq_range >= 10:
        print("   • Cable podría ser de baja calidad (RG-59 viejo)")
        print("   • Considerar reemplazar con RG-6 nuevo")
    
    if noise_rssi >= 25:
        print("   • Verificar blindaje del cable (malla intacta)")
        print("   • Alejar de fuentes de interferencia (routers, etc.)")
        print("   • Considerar agregar filtro FM")
    
    if score < 60:
        print("   • Si cable >30m, considerar amplificador de señal")
        print("   • Revisar divisores (usar 5-2400 MHz, no 5-1000 MHz)")
    
    print()

if __name__ == '__main__':
    test_cable_quality()
```

### Interpretación de Resultados

**RSSI (Received Signal Strength Indicator):**
- **55-60:** Señal excelente - Sin ruido, perfecta
- **45-54:** Señal muy buena - Mínimo ruido
- **35-44:** Señal buena - Usable con calidad
- **25-34:** Señal débil - Algunos problemas
- **15-24:** Señal muy débil - Mucho ruido
- **0-14:** Sin señal utilizable

**Mejoras esperadas con cable coaxial vs antena simple:**
- **+10-15 dB:** Excelente (cable corto, antena buena)
- **+5-10 dB:** Muy bueno (instalación normal)
- **+2-5 dB:** Bueno (cable largo o divisor con pérdida)
- **0-2 dB:** Marginal (revisar instalación)
- **Negativo:** Problema (cable defectuoso o excesiva longitud)

### Consejos de Optimización

Si los resultados no son óptimos:

1. **Mejorar conexiones:**
   ```
   - Apretar todos los conectores F
   - Limpiar conectores con alcohol isopropílico
   - Reemplazar conectores corroídos
   ```

2. **Reducir pérdidas:**
   ```
   - Usar el cable más corto posible
   - Minimizar número de divisores
   - Usar divisores de calidad (5-2400 MHz)
   ```

3. **Amplificación (si cable >30m):**
   ```
   Antena → Amplificador (20dB) → Distribuidor → Tomas
                ↑
            Instalar cerca de antena
            Precio: $20-40
   ```

4. **Filtrado de interferencias:**
   ```
   Toma → Filtro FM (88-108 MHz) → RDA5807M
          Rechaza señales fuera de banda
          Precio: $15-25
   ```

---

## 🎯 Próximas Mejoras

- [ ] Implementación completa de RDS/RBDS
- [ ] Equalizer de audio
- [ ] Grabación programada de emisoras
- [ ] Soporte para múltiples RDA5807M (múltiples sintonizadores)
- [ ] Integración con servicios de música (Spotify, etc.)
- [ ] Reconocimiento de canciones (Shazam-like)
- [ ] Panel de administración web avanzado
- [ ] Notificaciones push en app móvil
- [ ] Widget para Android
- [ ] Soporte para AirPlay/Chromecast

---

## Licencia

Este proyecto está bajo licencia MIT. Siéntete libre de usar, modificar y distribuir.

## Créditos

Desarrollado como proyecto educativo para aprender sobre:
- Radio FM y DSP
- Streaming de audio
- Desarrollo full-stack
- Integración de hardware con software
- Arquitecturas de microservicios

---

## Contacto

Para preguntas, sugerencias o reportar bugs, abre un issue en el repositorio del proyecto.

¡Disfruta tu radio FM! 📻🎵
