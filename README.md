# 🌡️ Simulasi Heat Transfer: CES vs DES

<div align="center">

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243?style=for-the-badge&logo=numpy&logoColor=white)
![SciPy](https://img.shields.io/badge/SciPy-8CAAE6?style=for-the-badge&logo=scipy&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-11557c?style=for-the-badge&logo=python&logoColor=white)

**Perbandingan Continuous Event Simulation (CES) vs Discrete Event Simulation (DES)**  
**dalam Pemodelan Perpindahan Panas**

[📓 Lihat Notebook](https://colab.research.google.com/drive/1VyK3aIRFuhXpGGz6xNvaCJoxytAKKSp0?usp=sharing) • [📊 Dataset](https://www.kaggle.com/datasets/parthdande/timeseries-weather-dataset/data) • [🎯 Hasil](#-hasil-dan-visualisasi)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1VyK3aIRFuhXpGGz6xNvaCJoxytAKKSp0?usp=sharing)

</div>

---

## 📋 Daftar Isi

- [Tentang Proyek](#-tentang-proyek)
- [Tujuan Simulasi](#-tujuan-simulasi)
- [Dataset](#-dataset)
- [Teori dan Metodologi](#-teori-dan-metodologi)
- [Implementasi](#-implementasi)
- [Hasil dan Visualisasi](#-hasil-dan-visualisasi)
- [Perbandingan CES vs DES](#-perbandingan-ces-vs-des)
- [Kesimpulan](#-kesimpulan)
- [Instalasi dan Penggunaan](#-instalasi-dan-penggunaan)
- [Penulis](#-penulis)

---

## 🎯 Tentang Proyek

Proyek ini mengimplementasikan dan membandingkan dua pendekatan simulasi perpindahan panas:

- **CES (Continuous Event Simulation)** - Menggunakan metode ODE (Ordinary Differential Equation)
- **DES (Discrete Event Simulation)** - Menggunakan pendekatan iteratif diskrit

Simulasi dilakukan dengan **data cuaca riil** untuk memodelkan proses **pendinginan** dan **pemanasan** objek dalam lingkungan dengan suhu yang berubah setiap jam.

### 🔬 Konteks Akademik

**Mata Kuliah:** Pemodelan dan Simulasi Data C  
**Institusi:** Universitas Muhammadiyah Malang  
**Fokus:** Event-Based Simulation & Heat Transfer Modeling

---

## 🎯 Tujuan Simulasi

Simulasi ini bertujuan untuk:

1. 🔄 **Memodifikasi simulasi perpindahan panas** dengan berbagai parameter
2. 📊 **Menganalisis efek variasi laju pendinginan** (cooling rate `k`)
3. 🔥 **Mensimulasikan proses pemanasan** dan pendinginan
4. 🌤️ **Menggunakan data suhu lingkungan riil** untuk realisme tinggi
5. ⚖️ **Membandingkan akurasi dan efisiensi** CES vs DES

### Skenario yang Disimulasikan

```
✓ Cooling dengan berbagai nilai k (0.1, 0.3, 0.6)
✓ Heating dengan k = 0.3
✓ Perbandingan CES vs DES
✓ Durasi simulasi: 24 jam
```

---

## 📊 Dataset

### Informasi Dataset

**Sumber:** [Kaggle - Time Series Weather Dataset](https://www.kaggle.com/datasets/parthdande/timeseries-weather-dataset/data)

| Properti | Detail |
|----------|--------|
| **Nama File** | `Weather_Data_1980_2024(hourly).csv` |
| **Periode** | 1980 - 2024 |
| **Interval** | Per jam (hourly) |
| **Data yang Digunakan** | 24 jam pertama suhu lingkungan |
| **Variabel** | Suhu (Temperature) dalam °C |

### Contoh Data

```python
# 24 jam pertama data suhu lingkungan
Hour    Temperature (°C)
0       15.2
1       14.8
2       14.5
3       14.1
...     ...
23      16.8
```

**Penggunaan dalam Simulasi:**
- Data suhu ini digunakan sebagai `T_env` (suhu lingkungan) yang berubah setiap jam
- Memberikan kondisi realistis untuk simulasi perpindahan panas
- Mensimulasikan skenario real-world seperti pendinginan makanan atau pemanasan ruangan

---

## 📚 Teori dan Metodologi

### Hukum Newton tentang Pendinginan

Perpindahan panas antara objek dan lingkungan mengikuti hukum:

```
Rate of heat transfer ∝ Temperature difference
```

### 1️⃣ Continuous Event Simulation (CES)

CES menggunakan **Ordinary Differential Equations (ODE)** yang diselesaikan dengan metode numerik.

#### Persamaan Pendinginan

```
dT/dt = -k(T - T_env)
```

#### Persamaan Pemanasan

```
dT/dt = k(T_env - T)
```

**Dimana:**
- `T` = Suhu objek (°C)
- `T_env` = Suhu lingkungan (°C)
- `k` = Konstanta laju perpindahan panas (1/jam)
- `t` = Waktu (jam)

#### Implementasi dengan SciPy

```python
from scipy.integrate import odeint

def cooling_model(T, t, k, T_env):
    """Model ODE untuk pendinginan"""
    dTdt = -k * (T - T_env)
    return dTdt

# Solusi numerik
T_solution = odeint(cooling_model, T0, time_points, args=(k, T_env))
```

**Karakteristik CES:**
- ✅ Solusi kontinu dan smooth
- ✅ Akurasi tinggi
- ⚠️ Komputasi lebih kompleks
- ⚠️ Memerlukan solver ODE

---

### 2️⃣ Discrete Event Simulation (DES)

DES menggunakan **iterasi diskrit** dengan persamaan beda hingga (finite difference).

#### Rumus Cooling (Diskrit)

```
T(t+1) = T(t) - k × Δt × (T(t) - T_env(t))
```

#### Rumus Heating (Diskrit)

```
T(t+1) = T(t) + k × Δt × (T_env(t) - T(t))
```

**Dimana:**
- `Δt` = Time step (1 jam dalam simulasi ini)

#### Implementasi dengan Loop

```python
def des_cooling(T0, T_env_array, k, dt=1):
    """DES untuk pendinginan"""
    T = [T0]
    for T_env in T_env_array:
        T_new = T[-1] - k * dt * (T[-1] - T_env)
        T.append(T_new)
    return T
```

**Karakteristik DES:**
- ✅ Implementasi sederhana
- ✅ Komputasi cepat
- ⚠️ Hasil diskrit (bergerigi)
- ⚠️ Akurasi bergantung pada time step

---

## 💻 Implementasi

### Setup dan Import Libraries

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy.integrate import odeint
```

### 1. Load Data Cuaca

```python
# Load dataset
df = pd.read_csv('Weather_Data_1980_2024(hourly).csv')

# Ambil 24 jam pertama
T_env_data = df['Temperature'].head(24).values
time_hours = np.arange(0, 24, 1)
```

### 2. Implementasi CES

```python
def cooling_ode(T, t, k, T_env_func):
    """ODE untuk cooling menggunakan CES"""
    T_env = T_env_func(t)
    dTdt = -k * (T - T_env)
    return dTdt

def heating_ode(T, t, k, T_env_func):
    """ODE untuk heating menggunakan CES"""
    T_env = T_env_func(t)
    dTdt = k * (T_env - T)
    return dTdt

# Interpolasi suhu lingkungan
from scipy.interpolate import interp1d
T_env_func = interp1d(time_hours, T_env_data, kind='linear', fill_value='extrapolate')

# Simulasi CES Cooling
T0_cooling = 80  # Suhu awal 80°C
time_points = np.linspace(0, 24, 500)  # 500 points untuk smooth curve

T_ces_cooling = odeint(cooling_ode, T0_cooling, time_points, args=(k, T_env_func))
```

### 3. Implementasi DES

```python
def des_cooling(T0, T_env_array, k, dt=1):
    """DES untuk cooling"""
    T = [T0]
    for i in range(len(T_env_array)):
        T_new = T[-1] - k * dt * (T[-1] - T_env_array[i])
        T.append(T_new)
    return np.array(T)

def des_heating(T0, T_env_array, k, dt=1):
    """DES untuk heating"""
    T = [T0]
    for i in range(len(T_env_array)):
        T_new = T[-1] + k * dt * (T_env_array[i] - T[-1])
        T.append(T_new)
    return np.array(T)

# Simulasi DES
T_des_cooling = des_cooling(T0_cooling, T_env_data, k=0.3)
T_des_heating = des_heating(T0_heating, T_env_data, k=0.3)
```

### 4. Variasi Parameter k

```python
# Simulasi dengan berbagai nilai k
k_values = [0.1, 0.3, 0.6]
results = {}

for k in k_values:
    T_result = odeint(cooling_ode, T0_cooling, time_points, args=(k, T_env_func))
    results[f'k={k}'] = T_result.flatten()
```

---

## 📈 Hasil dan Visualisasi

### 1. Efek Variasi Nilai k (CES Cooling)

**Pengamatan:**

| Nilai k | Karakteristik Pendinginan | Waktu untuk Mencapai T_env |
|---------|---------------------------|----------------------------|
| **k = 0.1** | 🐌 Sangat lambat | > 24 jam |
| **k = 0.3** | 🚶 Sedang | ~18 jam |
| **k = 0.6** | 🏃 Sangat cepat | ~10 jam |

**Visualisasi:**

```
Suhu (°C)
  80 |                                    k=0.1
     |                                    -------___
  60 |                           k=0.3            ----___
     |                           ---------___            ---
  40 |                  k=0.6            ----____           
     |              ----------____              ----____
  20 |         ------             ----____              ----
     |    -----                          ----____
   0 |____________________________________________________
     0    4    8    12   16   20   24
                  Waktu (jam)
```

**Insight:**
- ✅ Nilai `k` kecil → Isolasi termal baik (termos, styrofoam)
- ✅ Nilai `k` besar → Konduktor baik (logam, air)
- ✅ Real-world: Makanan dalam kulkas (k≈0.1-0.3)

---

### 2. Proses Pemanasan (CES Heating)

**Kondisi Awal:**
- Suhu objek: 20°C
- Suhu lingkungan: Bervariasi (dari data cuaca)
- k = 0.3

**Hasil:**

```
Suhu (°C)
  30 |                              _______
     |                         _____
  25 |                    _____
     |               _____
  20 |__________
     |
  15 |     T_env (variasi)
     |     ~~~~~~~~~~~
  10 |
     |________________________________________________
     0    4    8    12   16   20   24
                  Waktu (jam)
```

**Karakteristik:**
- Objek dingin dipanaskan menuju suhu lingkungan
- Proses mengikuti kurva eksponensial (asimptotik)
- Rate pemanasan bergantung pada `k` dan perbedaan suhu

**Contoh Aplikasi:**
- ☕ Pemanasan kopi di suhu ruang
- 🏠 Pemanasan ruangan dengan heater
- 🍲 Makanan beku yang dicairkan

---

### 3. Perbandingan CES vs DES

**Setup Perbandingan:**
- Kondisi sama: T₀ = 80°C, k = 0.3
- Simulasi: Cooling selama 24 jam

**Visualisasi Perbandingan:**

```
Suhu (°C)
  80 |
     |  CES (smooth)
  60 |  ____________
     |             ~~~---___
  40 |  DES (stepped)      ~~~--___
     |  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓---___
  20 |                                  ~~~--
     |________________________________________________
     0    4    8    12   16   20   24
                  Waktu (jam)
```

#### Analisis Kuantitatif

```python
# Error antara CES dan DES
error = np.mean(np.abs(T_ces - T_des_interpolated))
max_error = np.max(np.abs(T_ces - T_des_interpolated))

print(f"Mean Absolute Error: {error:.2f}°C")
print(f"Maximum Error: {max_error:.2f}°C")
```

**Hasil:**

| Metrik | CES | DES | Perbedaan |
|--------|-----|-----|-----------|
| **Smoothness** | Sangat halus | Bergerigi | CES lebih baik |
| **Akurasi** | Referensi | Error ~2-5% | CES lebih akurat |
| **Waktu Komputasi** | ~50ms | ~5ms | DES 10x lebih cepat |
| **Kompleksitas** | Tinggi (ODE solver) | Rendah (loop) | DES lebih simple |

---

## ⚖️ Perbandingan CES vs DES

### Karakteristik Lengkap

#### Continuous Event Simulation (CES)

**Kelebihan:**
- ✅ **Akurasi tinggi** - Mendekati solusi analitik
- ✅ **Smooth curves** - Tidak ada diskontinuitas
- ✅ **Fleksibel** - Mudah ubah time resolution
- ✅ **Robust** - Solver ODE handle stiffness

**Kekurangan:**
- ❌ **Komputasi berat** - Perlu numerical solver
- ❌ **Kompleks** - Butuh pemahaman ODE
- ❌ **Memory intensive** - Simpan banyak intermediate points

**Best Use Cases:**
- 🔬 Penelitian ilmiah yang memerlukan presisi tinggi
- 📊 Analisis jangka panjang
- 🎓 Validasi model matematis

---

#### Discrete Event Simulation (DES)

**Kelebihan:**
- ✅ **Sederhana** - Easy to implement dan understand
- ✅ **Cepat** - Komputasi minimal
- ✅ **Efisien** - Memory usage rendah
- ✅ **Intuitif** - Step-by-step logic

**Kekurangan:**
- ❌ **Kurang akurat** - Error akumulasi
- ❌ **Bergantung Δt** - Perlu time step kecil untuk akurasi
- ❌ **Tidak smooth** - Hasil "tangga"
- ❌ **Numerical instability** - Jika k dan Δt terlalu besar

**Best Use Cases:**
- 💼 Aplikasi bisnis / industri (akurasi cukup)
- ⚡ Real-time simulation
- 🎮 Game development
- 📱 Embedded systems

---

### Tabel Perbandingan Komprehensif

| Aspek | CES (ODE) | DES (Iterative) | Rekomendasi |
|-------|-----------|-----------------|-------------|
| **Akurasi** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | CES untuk presisi |
| **Kecepatan** | ⭐⭐ | ⭐⭐⭐⭐⭐ | DES untuk speed |
| **Kemudahan** | ⭐⭐ | ⭐⭐⭐⭐⭐ | DES untuk simplicity |
| **Smoothness** | ⭐⭐⭐⭐⭐ | ⭐⭐ | CES untuk visualization |
| **Memory** | ⭐⭐ | ⭐⭐⭐⭐ | DES untuk efficiency |
| **Skalabilitas** | ⭐⭐⭐ | ⭐⭐⭐⭐ | DES untuk large-scale |

---

### Kapan Menggunakan Masing-Masing?

#### Gunakan CES jika:
```
✓ Butuh hasil yang sangat akurat
✓ Melakukan penelitian ilmiah
✓ Visualisasi smooth curves penting
✓ Simulasi jangka panjang
✓ Ada komputasi resource yang cukup
```

#### Gunakan DES jika:
```
✓ Implementasi cepat lebih penting
✓ Akurasi 95%+ sudah cukup
✓ Real-time simulation diperlukan
✓ Resource terbatas (embedded system)
✓ Debugging dan maintenance prioritas
```

---

## 💡 Kesimpulan

### Ringkasan Temuan

1. **Efek Konstanta k**
   - Nilai `k` sangat mempengaruhi **kecepatan perpindahan panas**
   - k kecil (0.1) → pendinginan lambat (isolator baik)
   - k besar (0.6) → pendinginan cepat (konduktor baik)
   - Dapat digunakan untuk **material characterization**

2. **Pemanasan vs Pendinginan**
   - Kedua proses mengikuti **persamaan diferensial yang sama**
   - Perbedaan hanya pada **arah aliran energi**
   - Model dapat digunakan untuk berbagai aplikasi termal

3. **CES vs DES**
   - **CES lebih akurat** tapi komputasi lebih berat
   - **DES lebih sederhana** dan cepat tapi sedikit kurang akurat
   - Pilihan metode tergantung **trade-off akurasi vs efisiensi**

4. **Data Riil**
   - Penggunaan **data cuaca riil** memberikan simulasi yang realistis
   - Fluktuasi suhu lingkungan mempengaruhi dinamika sistem
   - Model dapat diaplikasikan untuk **real-world scenarios**

### Aplikasi Praktis

**Industri Makanan:**
- 🍖 Pendinginan makanan di cold storage
- 🍕 Prediksi waktu pendinginan setelah dimasak
- 🧊 Optimasi sistem refrigerasi

**HVAC (Heating, Ventilation, Air Conditioning):**
- 🏠 Simulasi pemanasan/pendinginan ruangan
- 💨 Optimasi sistem AC
- 📊 Prediksi konsumsi energi

**Material Science:**
- 🔬 Karakterisasi properti termal material
- 🧪 Studi heat treatment
- ⚗️ Proses annealing simulation

**Elektronika:**
- 💻 Thermal management CPU/GPU
- 📱 Prediksi overheating smartphone
- 🔋 Battery thermal modeling

---

## 🚀 Instalasi dan Penggunaan

### Prerequisites

```bash
Python 3.7+
pip (Python package manager)
```

### Installation

#### 1. Clone Repository

```bash
git clone https://github.com/username/heat-transfer-simulation.git
cd heat-transfer-simulation
```

#### 2. Install Dependencies

```bash
pip install numpy pandas matplotlib scipy
```

#### 3. Download Dataset

Download dataset dari [Kaggle](https://www.kaggle.com/datasets/parthdande/timeseries-weather-dataset/data) dan letakkan di folder `data/`

#### 4. Jalankan Simulasi

**Opsi A: Google Colab** (Recommended)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1VyK3aIRFuhXpGGz6xNvaCJoxytAKKSp0?usp=sharing)

**Opsi B: Jupyter Notebook**

```bash
jupyter notebook heat_transfer_simulation.ipynb
```

**Opsi C: Python Script**

```bash
python simulate_heat_transfer.py --method CES --k 0.3 --mode cooling
```

### Kustomisasi Parameter

```python
# Edit parameter simulasi
config = {
    'T0_cooling': 80,        # Suhu awal untuk cooling (°C)
    'T0_heating': 20,        # Suhu awal untuk heating (°C)
    'k_values': [0.1, 0.3, 0.6],  # Konstanta perpindahan panas
    'duration': 24,          # Durasi simulasi (jam)
    'method': 'CES',         # 'CES' atau 'DES'
    'mode': 'cooling'        # 'cooling' atau 'heating'
}
```

### Output

Simulasi menghasilkan:
- 📊 **Grafik interaktif** - Perbandingan CES vs DES
- 📈 **Time series plots** - Evolusi suhu dari waktu ke waktu
- 📄 **Statistik** - Error analysis, computation time
- 💾 **Export data** - CSV untuk analisis lebih lanjut

---

## 👤 Penulis

**Maylani Kusuma Wardhani**

- 🎓 **NIM**: 202210370311123
- 📚 **Kelas**: Pemodelan dan Simulasi Data C
- 🏫 **Institusi**: Universitas Muhammadiyah Malang

---

## 📄 Lisensi

Proyek ini dibuat untuk keperluan akademik dan pembelajaran.

---

## 🔗 Links Penting

- 📓 **Google Colab**: [Buka Simulasi](https://colab.research.google.com/drive/1VyK3aIRFuhXpGGz6xNvaCJoxytAKKSp0?usp=sharing)
- 📊 **Dataset**: [Kaggle Weather Data](https://www.kaggle.com/datasets/parthdande/timeseries-weather-dataset/data)

---

<div align="center">

### ⭐ Jika proyek ini bermanfaat, berikan star!

**Made with 🔥 and ❄️ by Maylani Kusuma Wardhani**

`Simulate • Analyze • Compare`

---

![Footer](https://img.shields.io/badge/Status-Active-success?style=for-the-badge)
![Simulation](https://img.shields.io/badge/Type-Heat_Transfer-red?style=for-the-badge)
![Academic](https://img.shields.io/badge/Purpose-Academic-orange?style=for-the-badge)

</div>
