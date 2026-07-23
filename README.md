# fantastic-spork
To fantastyczne rozszerzenie projektu, Anonse! Dodanie obsługi **trzech niezależnych ścieżek audio** (wewnętrzny mikrofon + zewnętrzny mikrofon + wejście ze słuchawek Bluetooth/mikrofon w nich) oraz **transkrypcji tekstu w czasie rzeczywistym** z analizą jakości przez LM (Large Language Model / Local LLM) czyni z tej aplikacji profesjonalne narzędzie studyjne.

Poniżej przedstawiam kompletny schemat architektury, strukturę plików oraz propozycje technologii Open Source, które pozwolą zbudować **"VocalForge AI Studio"** na Twoim Redmi Note 17 Pro 5G.

---

### 🚀 Koncepcja Rozszerzona: "VocalForge AI Studio"

**Główne założenia:**
1.  **Multi-Track Recording:** Jednoczesne nagrywanie 3 kanałów:
    *   Kanał 1: Mikrofon wbudowany telefonu.
    *   Kanał 2: Zewnętrzny mikrofon (przez USB-C OTG lub Jack 3.5mm).
    *   Kanał 3: Słuchawki Bluetooth (mikrofon w zestawie słuchawkowym) – *uwaga: tutaj kluczowe będzie użycie protokołu aptX Voice lub podobnego dla niskich opóźnień, albo wymuszenie trybu "Hands-Free" jeśli jakość jest priorytetem nad latencją.*
2.  **AI Audio Engine:**
    *   **Denoising:** Usuwanie szumów tła w czasie rzeczywistym (np. wentylator, ulica).
    *   **Pitch Correction:** Auto-Tune dla każdego kanału osobno.
    *   **Quality Analysis (LM):** Lokalny model językowy (np. TinyLlama lub specjalistyczny model audio-whisper) ocenia jakość dźwięku, wykrywa fałsze i generuje raport tekstowy.
3.  **Transkrypcja Live:** Zamiana mowy na tekst (Speech-to-Text) zapisywany jako metadane do pliku projektu.
4.  **Open Source & Free:** Całość na licencji MIT/Apache 2.0, bez płatnych API.

---

### ️ Stack Technologiczny (Open Source)

| Moduł | Technologia / Biblioteka | Dlaczego? |
| :--- | :--- | :--- |
| **Język Core** | Kotlin + C++ (JNI) | Kotlin dla UI/Logiki, C++ dla przetwarzania audio w czasie rzeczywistym (niskie opóźnienia). |
| **Silnik Audio** | **Oboe** (Google) + **PortAudio** | Oboe zapewnia najniższą latencję na Androidzie. PortAudio pomaga zarządzać wieloma urządzeniami wejściowymi. |
| **ML Audio (Denoise/Pitch)** | **RNNoise** (szumy) + **SoundTouch** (pitch) | RNNoise to świetny open-source'owy algorytm usuwania szumów. SoundTouch do zmiany tonacji. |
| **LM / Transkrypcja** | **Whisper.cpp** (port C++) lub **TensorFlow Lite** | Whisper.cpp działa lokalnie na telefonie, transkrybuje głos na tekst i może analizować kontekst. |
| **Analiza Jakości** | Custom Model (TFLite) + Logika heurystyczna | Model trenowany na danych audio do oceniania "czystości" dźwięku. |
| **Baza Danych** | **Room** (SQLite) | Przechowywanie metadanych sesji, ścieżek i wyników analizy LM. |
| **Chmura / Git** | **OkHttp** + **GitHub API** + **Google Drive SDK** | Bezpośrednia komunikacja z serwisami. |
| **UI** | **Jetpack Compose** | Nowoczesny, responsywny interfejs. |

---

### 📂 Pełna Struktura Plików Projektu (Android Studio)

Oto jak powinien wyglądać drzewo katalogów, aby projekt był czytelny i skalowalny:

```text
VocalForgeAI/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/anonse/vocalforge/
│   │   │   │   ├── di/                  # Dependency Injection (Hilt/Koin)
│   │   │   │   │   ── AppModule.kt
│   │   │   │   ├── data/
│   │   │   │   │   ├── local/           # Room Database
│   │   │   │   │   │   ├── AudioSessionDao.kt
│   │   │   │   │   │   └── AppDatabase.kt
│   │   │   │   │   ├── remote/          # GitHub/Drive Clients
│   │   │   │   │   │   ├── GitHubService.kt
│   │   │   │   │   │   └── DriveManager.kt
│   │   │   │   │   └── repository/      # Repozytoria danych
│   │   │   │   ├── domain/
│   │   │   │   │   ├── model/           # Modele danych (Session, Track, QualityReport)
│   │   │   │   │   ── usecase/         # Logika biznesowa (StartRecording, AnalyzeQuality)
│   │   │   │   ├── presentation/
│   │   │   │   │   ├── ui/              # Jetpack Compose Screens
│   │   │   │   │   │   ├── MainScreen.kt
│   │   │   │   │   │   ├── MixerScreen.kt  # Widok 3 ścieżek z suwakami
│   │   │   │   │   │   └── SettingsScreen.kt
│   │   │   │   │   ├── viewModel/       # ViewModel'e
│   │   │   │   │   └── components/      # Reusable UI components (WaveformView)
│   │   │   │   ├── core/
│   │   │   │   │   ├── audio/           # KLUCZOWY MODUŁ AUDIO
│   │   │   │   │   │   ├── AudioEngine.kt      # Zarządzanie Oboe/C++
│   │   │   │   │   │   ├── MultiTrackRecorder.kt # Logika 3 kanałów
│   │   │   │   │   │   ├── EffectsProcessor.kt # Auto-tune, Denoise (C++ wrapper)
│   │   │   │   │   │   └── DeviceManager.kt    # Wykrywanie mic USB/BT
│   │   │   │   │   ├── ml/              # Moduły ML
│   │   │   │   │   │   ├── WhisperTranscriber.kt # STT
│   │   │   │   │   │   ├── QualityAnalyzer.kt  # Ocena jakości przez LM
│   │   │   │   │   │   ── ModelLoader.kt      # Ładowanie .tflite/.bin
│   │   │   │   │   ├── utils/           # Narzędzia (FileUtils, Permissions)
│   │   │   │   │   └── network/         # Konfiguracja sieci
│   │   │   │   ├── VocalForgeApp.kt     # Główna klasa aplikacji
│   │   │   │   └── MainActivity.kt
│   │   │   ├── cpp/                     # Kod natywny C++
│   │   │   │   ├── audio_engine.cpp     # Implementacja Oboe
│   │   │   │   ├── dsp_effects.cpp      # Algorytmy pitch/noise reduction
│   │   │   │   └── CMakeLists.txt
│   │   │   ├── res/                     # Zasoby (layouty, ikony, modele ML)
│   │   │   │   ├── raw/
│   │   │   │   │   ├── whisper_model.bin
│   │   │   │   │   └── denoise_model.tflite
│   │   │   │   └── values/
│   │   │   ├── AndroidManifest.xml
│   │   │   └── assets/                  # Dodatkowe pliki konfiguracyjne
│   │   └── test/                        # Testy jednostkowe
│   └── build.gradle.kts                 # Konfiguracja zależności
├── build.gradle.kts                     # Konfiguracja projektu
├── settings.gradle.kts
└── README.md                            # Dokumentacja Open Source
```

---

### ⚙️ Szczegółowy Plan Implementacji Kluczowych Modułów

#### 1. Moduł Multi-Track Audio (C++ / Oboe)
To serce aplikacji. Musisz stworzyć klasę `MultiTrackRecorder`, która inicjalizuje trzy strumienie danych:
*   **Stream 1:** `AudioInputSource::Microphone` (Wbudowany).
*   **Stream 2:** `AudioInputSource::UsbDevice` (Wykrywanie podpiętego mikrofonu przez USB OTG).
*   **Stream 3:** `AudioInputSource::BluetoothSco` (Słuchawki BT).
    *   *Wyzwanie:* Bluetooth ma większe opóźnienie. W kodzie C++ musisz zastosować buforowanie adaptacyjne, aby zsynchronizować wszystkie trzy ścieżki przed zapisem do pliku WAV (multichannel WAV).

#### 2. Moduł ML: Jakość Dźwięku i Transkrypcja
*   **Transkrypcja:** Użyj **Whisper.cpp**. Załaduj skompilowany model (np. `tiny` lub `base` dla szybkości na mobile) do pamięci RAM. Przetwarzaj fragmenty audio (bufory 2-3 sekundowe) w tle, nie blokując nagrywania. Wynik (tekst) jest dopisywany do obiektu sesji w bazie danych.
*   **Analiza Jakości:**
    *   Stwórz prosty model TFLite, który przyjmuje spektrogram audio i zwraca ocenę (0.0 - 1.0) dla parametrów: *Szum*, *Przesterowania (Clipping)*, *Stabilność Pitchu*.
    *   Alternatywnie: Użyj heurystyki w C++ (analiza RMS dla głośności, detekcja zer crossings dla szumu), co jest szybsze i lżejsze.
    *   **LM Feedback:** Jeśli wykryjesz niską jakość, LM (lokalny chatbot) generuje sugestię: *"Wykryto szum tła w ścieżce 2. Zalecane włączenie filtra RNNoise"* lub *"Ścieżka 1 ma przesterowania, zmniejsz gain o 3dB"*.

#### 3. Edycja i Usuwanie Szumów (Real-time)
*   W pipeline'u C++ (`dsp_effects.cpp`) zastosuj filtr **RNNoise** przed zapisem do dysku.
*   Dla Auto-Tune: Zaimplementuj algorytm wykrywania pitchu (np. YIN), a następnie przesuń próbkę używając interpolacji (WSOLA), aby dopasować ją do najbliższej nuty w wybranej skali.

#### 4. Integracja GitHub i Google Drive
*   Po zakończeniu sesji, aplikacja pakuje:
    1.  Plik audio (.wav multichannel).
    2.  Plik transkrypcji (.txt/.json).
    3.  Raport jakości (.json).
    4.  Plik projektu (.vfp - własny format JSON z ustawieniami).
*   Użytkownik wybiera cel eksportu:
    *   **GitHub:** Tworzy commit z tagiem wersji, pushuje do repo.
    *   **Drive:** Wrzuca do folderu "VocalForge Backups".

---

###  Propozycje Dodatkowych Funkcjonalności (Value Added)

1.  **"AI Coach Mode":** Aplikacja nie tylko nagrywa, ale w trakcie śpiewania wyświetla na ekranie (nad falą dźwiękową) komunikaty od LM: *"Trzymaj dłużej tę nutę!", "Weź głębszy oddech"*, bazując na analizie w czasie rzeczywistym.
2.  **Separacja Ścieżek:** Jeśli użytkownik wgra gotowy podkład, użyj modelu **Demucs** (dostępny w wersji mobilnej), aby oddzielić wokal od muzyki, pozwalając na nagranie nowego wokalu na czysty instrumental.
3.  **Wersjonowanie Projektów:** Dzięki integracji z GitHubem, każda wersja nagrania jest automatycznie wersjonowana. Możesz cofnąć się do wersji z wczoraj, jeśli dzisiejsza edycja nie wyszła.
4.  **Tryb Offline-First:** Wszystko działa bez internetu. Synchronizacja z chmurą następuje tylko wtedy, gdy użytkownik wyraźnie tego chce.

### 🏁 Pierwsze Kroki do Realizacji (Action Plan)

1.  **Konfiguracja Środowiska:** Zainstaluj Android Studio, NDK (Native Development Kit) i CMake. To konieczne do kodu C++.
2.  **Prototyp Audio:** Napisz prostą apkę w C++ z użyciem Oboe, która nagra dźwięk z dwóch źródeł naraz (wbudowany + USB) i zapisze go jako jeden plik. To zweryfikuje możliwości Twojego Redmi.
3.  **Integracja Whisper.cpp:** Skompiluj Whisper.cpp pod architekturę ARM64 (procesor Twojego telefonu) i załaduj model do apki. Sprawdź czas transkrypcji 1 minuty audio.
4.  **UI Mixer:** Stwórz w Jetpack Compose widok z trzema suwakami głośności i przyciskami "Mute/Solo" dla każdej ścieżki.

Anonse, ten plan jest kompletny i wykorzystuje w 100% potencjał open source. Czy chcesz, abym przygotował teraz **kod C++ dla silnika audio (Oboe)** obsługującego wiele wejść, czy może wolisz zobaczyć **konfigurację Gradle** z wszystkimi niezbędnymi bibliotekami?
