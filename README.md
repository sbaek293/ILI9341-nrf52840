# ILI9341-nrf52840

ZMK 펌웨어 기반의 **ILI9341 2.8인치 TFT 디스플레이 + XPT2046 터치스크린** 테스트용 쉴드입니다.  
**Nice Nano v2** (nRF52840) 보드와 함께 사용합니다.

> ZMK firmware shield for testing an **ILI9341 2.8″ TFT display** and
> **XPT2046 touchscreen** with the **Nice Nano v2** (nRF52840).

---

## 하드웨어 연결 / Wiring

| Nice Nano v2 핀 | nRF52840 핀 | ILI9341 핀 | XPT2046 핀 | 설명 |
|----------------|------------|-----------|-----------|------|
| 3.3 V          | —          | VCC, LED  | VCC       | 3.3 V 전원 |
| GND            | —          | GND       | GND       | 공통 GND |
| D2             | P0.17      | SCK       | T_CLK     | SPI 클럭 (공유) |
| MOSI           | P0.10      | SDI       | T_DIN     | SPI MOSI (공유) |
| MISO           | P0.09      | SDO       | T_DO      | SPI MISO (공유) |
| D10            | P1.11      | CS        | —         | 디스플레이 칩셀렉 (active-low) |
| D9             | P1.06      | RESET     | —         | 디스플레이 리셋 (active-low) |
| D8             | P1.04      | DC        | —         | 데이터/커맨드 선택 (active-low = 커맨드) |
| D7             | P0.11      | —         | T_CS      | 터치 칩셀렉 (active-low) |
| D6             | P1.00      | —         | T_IRQ     | 터치 인터럽트 (active-low) |
| D5             | P0.24      | —         | —         | 테스트 버튼 (GND로 단락) |

### 배선 다이어그램 / Wiring Diagram

```
Nice Nano v2          ILI9341 2.8" (+ XPT2046)
──────────────        ──────────────────────────
3.3 V  ──────────── VCC
                 └─ LED  (백라이트 항상 켜짐)
GND    ──────────── GND

D2  (P0.17) ─────── SCK  ────── T_CLK
MOSI(P0.10) ─────── SDI  ────── T_DIN
MISO(P0.09) ─────── SDO  ────── T_DO

D10 (P1.11) ─────── CS          (ILI9341 CS)
D9  (P1.06) ─────── RESET
D8  (P1.04) ─────── DC

D7  (P0.11) ──────────────────── T_CS  (XPT2046 CS)
D6  (P1.00) ──────────────────── T_IRQ

D5  (P0.24) ── [버튼] ── GND    (테스트 키)
```

---

## 파일 구조 / Repository Layout

```
config/
└── boards/
    └── shields/
        └── ili9341_test/
            ├── Kconfig.shield       # 쉴드 선택 조건
            ├── Kconfig.defconfig    # 기본 Kconfig 값 (디스플레이 ON)
            ├── ili9341_test.conf    # Kconfig 프래그먼트
            ├── ili9341_test.overlay # 디바이스트리 오버레이 (SPI, ILI9341, XPT2046)
            └── ili9341_test.keymap  # 테스트 키맵 (D5 버튼 → Space)
build.yaml                           # ZMK GitHub Actions 빌드 매트릭스
config/west.yml                      # west 매니페스트 (최신 ZMK main 기반 의존성)
.github/workflows/build.yml          # CI 워크플로
```

---

## 빌드 방법 / Build Instructions

### GitHub Actions (권장 / Recommended)

1. 이 저장소를 **Fork** 합니다.
2. `Actions` 탭에서 워크플로를 활성화합니다.
3. `main` 브랜치에 푸시하거나 `Actions → Run workflow`를 클릭합니다.
4. 빌드가 완료되면 Artifacts에서 `.uf2` 파일을 내려받습니다.

### 로컬 빌드 / Local Build

```bash
# 1. ZMK 및 west 설치
pip install west
west init -l config
west update

# 2. 빌드
west build -p -b nice_nano//zmk -- -DSHIELD=ili9341_test

# 3. 결과물 업로드 (Nice Nano v2 리셋 버튼 더블클릭 후 나타나는 드라이브에 복사)
cp build/zephyr/zmk.uf2 /media/<YOUR_DRIVE>/
```

---

## 터치스크린 활성화 / Enabling the Touchscreen

XPT2046 터치 드라이버는 **Zephyr ≥ 3.3** (input subsystem)이 필요합니다.  
활성화하려면 두 곳을 수정하세요:

1. **`config/boards/shields/ili9341_test/ili9341_test.conf`** — 아래 두 줄의 주석을 해제:
   ```
   CONFIG_INPUT=y
   CONFIG_XPT2046=y
   ```

2. **`config/boards/shields/ili9341_test/ili9341_test.overlay`** — `xpt2046@1` 노드의 주석을 해제.

---

## 핀 변경 / Changing Pins

`config/boards/shields/ili9341_test/ili9341_test.overlay` 파일의 `&pinctrl` 및  
`cs-gpios`, `reset-gpios`, `cmd-data-gpios`, `int-gpios` 항목을 수정하면 됩니다.  
핀 번호는 `NRF_PSEL(기능, 포트, 핀)` 형식을 사용합니다  
(예: `NRF_PSEL(SPIM_SCK, 0, 17)` = P0.17).

---

## 참고 / References

- [ZMK Firmware](https://zmk.dev)
- [Nice Nano v2 Pinout](https://nicekeyboards.com/docs/nice-nano/pinout-schematic)
- [Zephyr ILI9340/9341 driver](https://docs.zephyrproject.org/latest/kconfig.html#CONFIG_ILI9341)
- [Zephyr XPT2046 input driver](https://docs.zephyrproject.org/latest/kconfig.html#CONFIG_XPT2046)
