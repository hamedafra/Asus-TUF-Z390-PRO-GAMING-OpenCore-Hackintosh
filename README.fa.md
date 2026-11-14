<div dir="rtl">

# هکینتاش OpenCore برای Asus TUF Z390-PRO GAMING

فولدر EFI اوپن‌کور برای **macOS 26 Tahoe**، آخرین نسخه‌ی macOS برای مک‌های اینتلی.

[English](README.md)

| قطعه | مدل |
|---|---|
| مادربرد | Asus TUF Z390-PRO GAMING |
| پردازنده | Intel Core i9-9900K (گرافیک UHD 630 فقط برای پردازش) |
| کارت گرافیک | ASUS ROG STRIX RX 5700 XT O8G GAMING |
| اترنت | Intel I219-V |
| وای‌فای | TP-Link Archer T9E (Broadcom BCM4360) |
| بلوتوث | Asus BT400 (Broadcom BCM20702) |

| | |
|---|---|
| OpenCore | 1.0.8 |
| SMBIOS | iMac20,1 |
| سیستم‌عامل | macOS 26 Tahoe |

> **وضعیت:** این EFI از روی EFI قدیمی OpenCore 0.5.7 دوباره ساخته شده و بررسی `ocvalidate` خود OpenCore را بدون
> خطا رد می‌کند، ولی **هنوز روی خود سیستم بوت نشده است**. قبل از جایگزین کردن EFI روی دیسک، یک فلش با EFI سالم
> کنار دستتان داشته باشید.

EFI قدیمی دوره‌ی Catalina (OpenCore 0.5.7 و iMac19,1) در تگ [`opencore-0.5.7`](../../tree/opencore-0.5.7) نگه
داشته شده است.

## چه چیزی روی Tahoe کار می‌کند و چه چیزی نه

| بخش | وضعیت |
|---|---|
| RX 5700 XT | درایور خود AMD در macOS (نسخه‌ی 26 هنوز Navi 10 را پشتیبانی می‌کند) |
| UHD 630 | بدون خروجی تصویر، برای Quick Sync (رمزگذاری و پخش ویدیو) |
| اترنت | IntelMausi |
| بلوتوث | BrcmPatchRAM3 و BrcmFirmwareData و BlueToolFixup |
| USB | USBMap.kext، ساخته‌شده از نقشه‌ی پورت SSDT-UIAC قدیمی (همان ۱۵ پورت) |
| صدای HDMI و DisplayPort، کارت صدای USB | کار می‌کند |
| **صدای آنالوگ (جک‌های پشت کیس)** | **کار نمی‌کند.** macOS 26 درایور AppleHDA را حذف کرده و AppleALC چیزی برای پچ کردن ندارد. از صدای HDMI یا DP، کارت صدای USB یا VoodooHDA استفاده کنید |
| **وای‌فای (Archer T9E)** | **به‌صورت پیش‌فرض کار نمی‌کند.** از macOS 14 درایور BCM4360 حذف شده و فقط با root patch برنامه‌ی OpenCore Legacy Patcher برمی‌گردد که باید SIP را پایین بیاورد |

## قبل از اولین بوت

1. **SMBIOS:** مقدارهای داخل کانفیگ نمونه هستند. با [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) برای
   `iMac20,1` مقدارهای خودتان را بسازید و در `PlatformInfo > Generic` بگذارید: `SystemSerialNumber`، `MLB`،
   `SystemUUID` و `ROM` (آدرس MAC اترنت بدون دونقطه). سریال را در صفحه‌ی Check Coverage اپل امتحان کنید؛ باید
   *نامعتبر* باشد.
2. فولدر `EFI` را در پارتیشن EFI کپی کنید (اول روی فلش امتحان کنید).

## تنظیمات بایوس

| تنظیم | مقدار |
|---|---|
| Fast Boot و Secure Boot و CSM | Disabled |
| VT-d | Disabled |
| CFG Lock | Disabled (چون `AppleXcpmCfgLock` خاموش است، این لازم است) |
| Above 4G Decoding | Enabled |
| XHCI Hand-off | Enabled |
| Primary Display | PEG (کارت RX 5700 XT) |
| iGPU Multi-Monitor | Enabled |
| DVMT Pre-Allocated | 64M |
| OS Type | Other OS |

## نکته‌های کانفیگ

- **WhateverGreen استفاده نشده است.** طبق راهنمای Dortania برای Tahoe، این kext روی macOS 26 با کارت‌های AMD
  مشکل دارد. اصلاح `agdpmod=pikera` که کارت‌های Navi لازم دارند، به‌جای آن یک پچ در `Kernel > Patch` است که در
  AppleGraphicsDevicePolicy عبارت `board-id` را به `board-ix` تغییر می‌دهد.
- **Booter Quirks** طبق راهنمای Coffee Lake برای Z390 تنظیم شده‌اند: `DevirtualiseMmio`، `ProtectUefiServices`،
  `RebuildAppleMemoryMap` و `SyncRuntimePermissions` روشن و `EnableWriteUnprotector` خاموش. اگر بوت درست بعد از
  OpenCore متوقف شد (بایوس قدیمی بدون جدول MAT)، `EnableWriteUnprotector` را روشن و `RebuildAppleMemoryMap` و
  `SyncRuntimePermissions` را خاموش کنید. آپدیت بایوس راه بهتری است.
- **آپدیت سیستم:** `SecureBootModel` روی `Disabled` است و RestrictEvents با `revpatch=sbvmm` اجرا می‌شود، همان‌طور
  که Dortania برای آپدیت روی macOS 14.4 به بعد پیشنهاد می‌کند.
- **ریست شدن بایوس بعد از ریستارت:** پچ قدیمی RTC حالا quirk خود OpenCore به اسم `DisableRtcChecksum` است.
- **بلوتوث روی macOS 12 به بعد:** کلیدهای `bluetoothExternalDongleFailed` و `bluetoothInternalControllerInfo` طبق
  README برنامه‌ی BrcmPatchRAM در NVRAM تنظیم شده‌اند.
- منوی بوت (OpenCanopy) پنج ثانیه نشان داده می‌شود. برای مخفی کردنش `Misc > Boot > ShowPicker` را `false` کنید و
  موقع بوت کلید Option را نگه دارید.

## نسخه‌ها

| قطعه | نسخه |
|---|---|
| OpenCore | 1.0.8 |
| OcBinaryData (HfsPlus.efi و فایل‌های منوی بوت) | master، ۱۴ نوامبر ۲۰۲۶ |
| Lilu | 1.7.2 |
| VirtualSMC و SMCProcessor و SMCSuperIO | 1.3.8 |
| IntelMausi | 1.0.8 |
| BrcmPatchRAM3 و BrcmFirmwareData و BlueToolFixup | 2.7.2 |
| RestrictEvents | 1.1.6 |

## تغییرها نسبت به EFI نسخه‌ی 0.5.7

- OpenCore از 0.5.7 به 1.0.8، و کانفیگ از روی `Sample.plist` نسخه‌ی 1.0.8 دوباره ساخته شده.
- SMBIOS از iMac19,1 به iMac20,1 (macOS 26 مدل iMac19,1 را پشتیبانی نمی‌کند).
- حالا همه‌چیز داخل فولدر `EFI/` با `BOOT` و `OC` است و آماده‌ی کپی.
- `USBInjectAll` و `SSDT-UIAC` جای خود را به `USBMap.kext` با همان پورت‌ها داده‌اند.
- `BrcmBluetoothInjector` جای خود را به `BlueToolFixup` داده.
- حذف‌شده‌ها: WhateverGreen، AppleALC (در 26 دیگر AppleHDA نیست)، ApfsDriverLoader، VBoxHfs، AppleUsbKbDxe،
  AudioDxe و درایورهای NVMe و XHCI و HII که استفاده نمی‌شدند.
- اضافه‌شده‌ها: RestrictEvents، ResetNvramEntry، OpenShell.
- UHD 630: از framebuffer نمایشی `0x3E980003` به حالت بدون خروجی `0x3E910003`.

## قدردانی

[Acidanthera](https://github.com/acidanthera) برای OpenCore و kextها،
[راهنمای Dortania](https://dortania.github.io/OpenCore-Install-Guide/) و
[corpnewt](https://github.com/corpnewt) برای GenSMBIOS.

</div>
