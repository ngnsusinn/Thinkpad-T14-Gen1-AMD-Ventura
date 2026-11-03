# ThinkPad T14 Gen 1 AMD — OpenCore Hackintosh (macOS Ventura)

> **English TL;DR:** OpenCore EFI for Lenovo ThinkPad T14 Gen 1 AMD (Ryzen 4000 / Renoir) running macOS Ventura. GPU acceleration works (NootedRed, Metal 3), WiFi + Bluetooth (Intel AX210) work after fixes. **Webcam and microphone do NOT work** (AMD xHCI isochronous crash — see below). Bluetooth on the ATK Dragonfly A9 mouse does not work (device firmware issue).

OpenCore EFI cho **Lenovo ThinkPad T14 Gen 1 AMD** (Ryzen 4000 / Renoir) chạy **macOS Ventura 13.7.8**.

## 🖥️ Cấu hình phần cứng

| Linh kiện | Model |
|---|---|
| CPU / iGPU | AMD Ryzen 4000 (Renoir) — Radeon Graphics (0x1636) |
| RAM | 32 GB |
| SSD | Micron MTFDKCD512TFK 512 GB (NVMe) |
| WiFi / Bluetooth | Intel AX210 |
| Audio codec | Realtek ALC257 |
| Webcam | Chicony 0x04F2:B6D0 (UVC) |
| Touchpad | Synaptics (SMBus, VoodooRMI) |
| Ethernet | Realtek RTL8111 |
| Fingerprint | Synaptics 0x06cb:00bd |
| USB | AMD xHCI (map qua UTBMap) |

## 📋 Tổng quan trạng thái

| ✅ Hoạt động | ❌ Không hoạt động | ⏳ Chưa test |
|---|---|---|
| Boot macOS ổn định | Webcam | Sleep/Wake |
| GPU (Metal 3, full acceleration) | Microphone | iMessage / FaceTime |
| WiFi (AX210) | Chuột ATK Dragonfly A9 (qua Bluetooth) | Continuity / AirDrop / Sidecar |
| Bluetooth (AX210) | Fingerprint | Touchscreen (Raydium) |
| Audio output (loa trong) | Chrome hardware acceleration | USB-C / DisplayPort alt |
| Bàn phím + touchpad | | Cổng SD (nếu có) |
| Phím sáng màn hình | | |
| Pin, NVMe, Ethernet | | |

---

## ✅ Hoạt động (đã test)

- **Boot macOS Ventura 13.7.8** — ổn định
- **GPU**: NootedRed 0.8.10 — **Metal 3**, VRAM 2 GB, full acceleration. Chạy 2 màn hình (panel trong + LG UltraGear 1080p/120Hz)
- **WiFi**: Intel AX210 qua AirportItlwm 2.3.0 — hoạt động tốt
- **Bluetooth**: Intel AX210 — **đã fix** (xem phần Fix bên dưới). Đã kết nối thành công:
  - Bàn phím MCHOSE G75 V2 (BLE)
  - Tai nghe Soundcore R60i NC (A2DP)
- **Audio output**: ALC257 — loa trong hoạt động (`alcid=101`)
- **Bàn phím / touchpad**: VoodooPS2Controller + VoodooRMI/SMBus
- **Điều chỉnh sáng màn hình**: BrightnessKeys + SSDT-PNLF
- **Pin**: SMCBatteryManager
- **NVMe, Ethernet (RTL8111), USB** (đã map UTBMap)

## ❌ Không hoạt động (đã test thất bại)

### 🎥 Webcam (Chicony 0x04F2:B6D0)
- **Triệu chứng**: Camera được nhận diện lúc boot, nhưng khi mở (Photo Booth/FaceTime):
  1. Đèn LED webcam sáng
  2. Màn hình đen → sau vài giây báo **"there is no camera connected"**
  3. Camera **biến mất khỏi USB bus** — phải reboot mới quay lại
- **Nguyên nhân**: Controller **AMD xHCI (pci1022:1632) crash khi xử lý isochronous** (video stream). Log kernel:
  ```
  AppleUSBXHCICommandRing::executeCommand: command[11] timeout
  hardware became unavailable
  PRT4: terminateDevice: destroying (Integrated Camera): port is powering off
  RESA@00000000: hardwareException — attempting recovery
  ```
- **Hướng xử lý**:
  - ⭐ **Update BIOS** (AMD AGESA đã vá nhiều lỗi USB) — khả năng cao nhất
  - Webcam USB ngoài (chạy trên controller khác)
  - Chưa có kext fix triệt để cho macOS Ventura

### 🎤 Microphone
- **Triệu chứng**: "Built-in Input / Internal Microphone" được nhận diện nhưng **không thu được âm thanh** (đã test với `alcid=18` và `alcid=101`)
- **Nghi vấn**: mic có thể nằm **trong module camera** (nhiều Lenovo làm vậy) → camera crash = mic chết. Chưa xác nhận được do camera không mở được
- **Hướng xử lý**: phụ thuộc hướng fix webcam; nếu mic là codec riêng, cần layout-id đúng cho model này (chưa có trong AppleALC)

### 🖱️ Chuột ATK Dragonfly A9 (qua Bluetooth)
- **Triệu chứng**: Xuất hiện trong scan ("ATK Mouse BT"), bấm Connect thấy kết nối, nhưng **GATT service discovery fail** → "No database exists for device" spam → timeout → ngắt
- **Nguyên nhân**: **Firmware BLE của chuột không tuân thủ chuẩn GATT** của macOS (các thiết bị BT khác chạy tốt)
- **Hướng xử lý**: Update firmware chuột qua **ATK Hub** (Windows), hoặc dùng dongle 2.4GHz

### ☝️ Fingerprint (Synaptics 0x06cb:00bd)
- Không hoạt động trên macOS (phổ biến trên hackintosh — chưa có driver)

### 🌐 Chrome hardware acceleration
- Bật "Use hardware acceleration" trong Chrome → **màn trắng/xanh, nhoè, đơ**
- Nguyên nhân: giới hạn của NootedRed với nhân Chromium ([NootedRed issue #122](https://github.com/ChefKissInc/NootedRed/issues/122))
- **Giải pháp**: tắt HW acceleration trong Chrome, hoặc dùng Safari/Firefox

## ⏳ Chưa test

- Sleep / Wake
- iMessage / FaceTime (cần SMBIOS hợp lệ)
- Continuity / AirDrop / Sidecar (giới hạn với Intel BT)
- Touchscreen (Raydium — máy có màn touch)
- USB-C / DisplayPort alternate mode
- Cổng SD card reader (nếu có)

---

## 🔧 Các fix quan trọng đã áp dụng

### Bluetooth AX210 (lỗi "BCM_4350C2 / Address NULL" trên Ventura)
1. Thêm 2 key NVRAM (GUID `7C436110-AB2A-4BBB-A880-FE41995C9F82`):
   - `bluetoothInternalControllerInfo` = 16 bytes `0x00`
   - `bluetoothExternalDongleFailed` = 1 byte `0x00`
2. Thêm boot-arg **`-btlfxnvramcheck`** (BlueToolFixup 2.7.2: skip "Internal Bluetooth Controller NVRAM check" — replicate hành vi bản fork zxystd đã fix Ventura 13.4+)
3. Xoá kext Broadcom không cần thiết: `BrcmBluetoothInjector`, `BrcmPatchRAM3`

### Audio
- `alcid=101` (ALC257 — Lenovo AMD Renoir; layout 18 là của Legion, sai mic pin)

### Khác
- `ScanPolicy = 0` — quét tất cả thiết bị/phân vùng theo khuyến nghị chuẩn của Dortania (tránh lỗi `OC: Invalid ScanPolicy` khi thiếu bit lock)
- `HideAuxiliary = true` — ẩn tool (bấm Space để hiện CleanNvram/ResetSystem)
- `Debug Target = 3` — tắt ghi log file mỗi lần boot
- Xoá tool không cần: BootKicker, ListPartitions, MmapDump, OpenShell
- `ReleaseUsbOwnership = true` — giải phóng quyền điều khiển xHCI từ firmware BIOS theo chuẩn Dortania/Olarila cho laptop
- Đã bật patch ACPI `HPET _CRS to XCRS` đồng bộ với `SSDT-HPET.aml`
- Loại bỏ các thành phần thừa: `SSDT-PLUG-ALT.aml` (chỉ dành cho Intel gen 12+), `ThunderboltReset.kext` (không có Thunderbolt), `SMCProcessorAMD.kext` (trùng lặp với SMCAMDProcessor), `VoodooI2CSynaptics.kext` & các plugin input gây xung đột SMBus

## 🧩 Kexts & phiên bản

| Kext | Version |
|---|---|
| Lilu | 1.7.2 |
| VirtualSMC | 1.3.7 |
| NootedRed | 0.8.10 |
| AirportItlwm | 2.3.0 |
| AppleALC | 1.9.7 |
| IntelBluetoothFirmware | 2.4.0 |
| IntelBTPatcher | 2.4.0 |
| BlueToolFixup | 2.7.2 |
| RestrictEvents | 1.1.6 |
| NVMeFix, RealtekRTL8111, USBToolBox, UTBMap, VoodooPS2Controller, VoodooRMI, VoodooSMBus, VoodooI2C, VoodooI2CHID, BrightnessKeys, ECEnabler, FeatureUnlock, SMCBatteryManager, SMCAMDProcessor, SMCRadeonGPU, AMDRyzenCPUPowerManagement, AmdTscSync, AppleMCEReporterDisabler, HibernationFixup, YogaSMCAlter | — |

## ⚙️ Boot-args

```
keepsyms=1 alcid=101 serverperfmode=1 -btlfxnvramcheck -v revcpu=1 revcpuname="AMD Ryzen 5 PRO 4650U with Radeon Graphics" revpatch=cpuname,sbvmm
```

## ⚠️ Lưu ý

- **SMBIOS trong `config.plist` là giá trị đang dùng** — nếu dùng EFI này công khai, nên **generate SMBIOS mới** (GenSMBIOS) trước khi đăng nhập iMessage/Apple ID
- EFI này dành cho máy cụ thể của tác giả; không đảm bảo hoạt động nguyên bản trên máy khác
- Tham khảo: [Dortania OpenCore Guide](https://dortania.github.io/OpenCore-Install-Guide/), [OpenIntelWireless](https://github.com/OpenIntelWireless), [ChefKissInc/NootedRed](https://github.com/ChefKissInc/NootedRed)

---
*OpenCore ~1.0.0 (build Mar 2024) · macOS Ventura 13.7.8 · Cập nhật 2026-08-26*
