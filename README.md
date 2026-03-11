# ModuleJET — Kế Hoạch Triển Khai Software Licensing

**Module:** Pricing Calculator (sản phẩm đầu tiên của ModuleJET)  
**Thương hiệu:** ModuleJET — modulejet.com  
**Email hỗ trợ:** support@modulejet.com  
**Licensing Platform:** WHMCS Software Licensing Addon  
**Ngày lập:** 11/03/2026  
**Trạng thái:** Draft v2

---

## 1. Tổng Quan

### 1.1 Mục tiêu

Triển khai hệ thống licensing cho module Pricing Calculator sử dụng WHMCS Software Licensing Addon, cho phép ModuleJET bán license qua website modulejet.com với WHMCS làm nền tảng billing + licensing.

### 1.2 Mô hình sản phẩm hiện tại

Module hiện tại: `hvn_pricing_calculator` v1.1.0 (production, đã hoàn thành 3 phases). Cần rebrand sang **ModuleJET Pricing Calculator** và tích hợp license verification.

### 1.3 Kiến trúc tổng quan

```
┌─────────────────────────────────────────────────────────┐
│                    KHÁCH HÀNG                            │
│  (WHMCS Admin cài module Pricing Calculator)            │
│                                                          │
│  Module gọi license check → modulejet.com/whmcs         │
└──────────────────────┬──────────────────────────────────┘
                       │ HTTPS (license verify)
                       ▼
┌─────────────────────────────────────────────────────────┐
│              MODULEJET.COM (WHMCS Instance)              │
│                                                          │
│  ┌─────────────────┐  ┌──────────────────────────────┐  │
│  │ Software         │  │ Product Catalog               │  │
│  │ Licensing Addon  │  │ 8 license products            │  │
│  │ (license keys,   │  │ (không cần Support/Updates    │  │
│  │  verify API)     │  │  addon riêng cho Lifetime)    │  │
│  └─────────────────┘  └──────────────────────────────┘  │
│                                                          │
│  ┌─────────────────┐  ┌──────────────────────────────┐  │
│  │ Client Portal    │  │ Download Manager              │  │
│  │ (order, pay,     │  │ (encoded + unencoded          │  │
│  │  manage license) │  │  packages)                    │  │
│  └─────────────────┘  └──────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

## 2. License Tiers & Pricing Structure

### 2.1 Key Prefix Convention (chuẩn hóa cho multi-product)

Để mở rộng cho các sản phẩm tương lai, key prefix theo format:

```
MJET-{PRODUCT}-{TIER}-
```

| Segment | Giá trị | Mô tả |
|---|---|---|
| `MJET` | Cố định | Thương hiệu ModuleJET |
| `{PRODUCT}` | `PC` = Pricing Calculator, `XX` = sản phẩm sau | Mã sản phẩm (2–4 ký tự) |
| `{TIER}` | `STD` / `PRO` / `UEN` | Standard / Professional / Unencoded |

Ví dụ key: `MJET-PC-STD-A1B2C3D4E5F6G7H8...`

Khi có sản phẩm mới (ví dụ "Invoice Manager"), chỉ cần tạo prefix `MJET-IM-STD-`, `MJET-IM-PRO-`, v.v.

### 2.2 Bảng sản phẩm

| # | Tên sản phẩm | Billing | Encoding | Domain | Key Prefix | Support/Updates |
|---|---|---|---|---|---|---|
| 1 | Monthly Standard License | Monthly | ionCube | 1 domain | `MJET-PC-STD-` | Khi còn active subscription |
| 2 | Annual Standard License | Annually | ionCube | 1 domain | `MJET-PC-STD-` | Khi còn active subscription |
| 3 | Lifetime Standard License | One Time | ionCube | 1 domain | `MJET-PC-STD-` | **Lifetime** (bao gồm) |
| 4 | Monthly Professional License | Monthly | ionCube | Unlimited | `MJET-PC-PRO-` | Khi còn active subscription |
| 5 | Annual Professional License | Annually | ionCube | Unlimited | `MJET-PC-PRO-` | Khi còn active subscription |
| 6 | Lifetime Professional License | One Time | ionCube | Unlimited | `MJET-PC-PRO-` | **Lifetime** (bao gồm) |
| 7 | Annual Unencoded License | Annually | Không mã hóa | Unlimited | `MJET-PC-UEN-` | Khi còn active subscription |
| 8 | Lifetime Unencoded License | One Time | Không mã hóa | Unlimited | `MJET-PC-UEN-` | **Lifetime** (bao gồm) |

> **Ghi chú về "Unencoded":** Đây KHÔNG phải open source. License vẫn là thương mại (proprietary). Khách mua quyền sử dụng source code không mã hóa, có thể đọc/tùy chỉnh nhưng **không được phân phối lại**. Tên "Unencoded" thay vì "Open Source" để tránh nhầm lẫn với giấy phép OSS (GPL, MIT...).

### 2.3 So sánh tính năng theo tier

| Tính năng | Standard | Professional | Unencoded |
|---|---|---|---|
| Pricing Calculator Toolbar | ✅ | ✅ | ✅ |
| Config Options Manager | ✅ | ✅ | ✅ |
| Presets & Settings | ✅ | ✅ | ✅ |
| Domain giới hạn | 1 domain | Unlimited | Unlimited |
| Source code | ionCube encoded | ionCube encoded | Plain PHP (đọc/sửa được) |
| Quyền sửa code | ❌ | ❌ | ✅ (chỉ cho domain của mình) |
| Quyền phân phối lại | ❌ | ❌ | ❌ |
| Hỗ trợ kỹ thuật | Email (48h) | Priority email (24h) | Priority email (24h) |
| Branding removal | ❌ | ✅ | ✅ |
| White-label | ❌ | ❌ | ✅ (có thể rebrand) |

### 2.4 Pricing

| Tier | Monthly | Annually | Lifetime |
|---|---|---|---|
| Standard | $4.95/mo | $39.95/yr (~33% off) | $99.95 |
| Professional | $9.95/mo | $79.95/yr (~33% off) | $199.95 |
| Unencoded | — | $119.95/yr | $299.95 |

**Giải thích giá Unencoded cao hơn:** Khách nhận source code đọc được + quyền tùy chỉnh + white-label. Đây là giá trị premium so với bản encoded.

### 2.5 Support/Updates Policy

| Billing | Support/Updates | Chi tiết |
|---|---|---|
| Monthly | Bao gồm khi subscription active | Hủy = mất quyền download bản mới |
| Annually | Bao gồm khi subscription active | Hết hạn = mất quyền download bản mới |
| Lifetime | **Bao gồm trọn đời** | Không cần renew, không cần addon riêng |

**Triển khai trong WHMCS:**
- Monthly/Annual: KHÔNG cần tạo Support/Updates addon riêng → WHMCS tự quản lý qua product status (Active/Suspended/Terminated)
- Lifetime: KHÔNG set Support/Updates Addon trong module settings → khách luôn có quyền download
- Download restrictions: enforce qua WHMCS product status — chỉ Active mới download được

---

## 3. Cấu Hình WHMCS Software Licensing Addon

### 3.1 Cài đặt Licensing Addon trên modulejet.com

**Bước 1 — Mua & cài addon:**
- Mua WHMCS Software Licensing Addon ($99.95 one-time) từ WHMCS Marketplace
- Upload `/modules/addons/licensing/` vào WHMCS installation trên modulejet.com
- Activate tại Configuration → System Settings → Addon Modules

**Bước 2 — Cấu hình addon:**
- Maximum Allowed Reissues: `3` (cho phép chuyển domain 3 lần)
- Auto Logs Prune: `90 days`
- Public License Verification Tool: ✅ Enable
- Gán quyền admin cho roles cần thiết

### 3.2 Tạo Product Group

```
Product Group: "ModuleJET — Pricing Calculator"
├── Standard Licenses
│   ├── PC Monthly Standard ($4.95/mo)
│   ├── PC Annual Standard ($39.95/yr)
│   └── PC Lifetime Standard ($99.95 one-time)
├── Professional Licenses
│   ├── PC Monthly Professional ($9.95/mo)
│   ├── PC Annual Professional ($79.95/yr)
│   └── PC Lifetime Professional ($199.95 one-time)
└── Unencoded Licenses
    ├── PC Annual Unencoded ($119.95/yr)
    └── PC Lifetime Unencoded ($299.95 one-time)
```

### 3.3 Cấu hình từng product (Module Settings tab)

**Chung cho tất cả 8 products:**
- Module Name: `License Software`
- Key Length: `32`
- Allow Reissue: ✅ Yes
- Support/Updates Addon: **Để trống** (không cần addon riêng)

**Standard License (products #1–3):**
- Key Prefix: `MJET-PC-STD-`
- Allow Domain Conflict: ❌ No (enforce 1 domain)
- Allow IP Conflict: ✅ Yes
- Allow Directory Conflict: ✅ Yes

**Professional License (products #4–6):**
- Key Prefix: `MJET-PC-PRO-`
- Allow Domain Conflict: ✅ Yes (unlimited domain)
- Allow IP Conflict: ✅ Yes
- Allow Directory Conflict: ✅ Yes

**Unencoded License (products #7–8):**
- Key Prefix: `MJET-PC-UEN-`
- Allow Domain Conflict: ✅ Yes (unlimited domain)
- Allow IP Conflict: ✅ Yes
- Allow Directory Conflict: ✅ Yes

### 3.4 Download Configuration

Tạo 2 download files gắn vào products:

| Download | Gắn vào products | Require Active |
|---|---|---|
| `modulejet-pricing-calculator-encoded-v1.2.0.zip` | Standard #1–3, Professional #4–6 | ✅ |
| `modulejet-pricing-calculator-unencoded-v1.2.0.zip` | Unencoded #7–8 | ✅ |

WHMCS tự enforce: chỉ product Active mới cho download. Lifetime = always Active.

---

## 4. Thay Đổi Codebase Module

### 4.1 Rebrand: hvn_pricing_calculator → modulejet_pricing_calculator

| Hạng mục | Cũ | Mới |
|---|---|---|
| Directory name | `hvn_pricing_calculator/` | `modulejet_pricing_calculator/` |
| Main PHP file | `hvn_pricing_calculator.php` | `modulejet_pricing_calculator.php` |
| Function prefix | `hvn_pricing_calculator_` | `modulejet_pricing_calculator_` |
| DB table | `tbl_hvn_pricing_presets` | `tbl_mjet_pricing_presets` |
| CSS class prefix | `hvn-` | `mjet-` |
| JS function prefix | `hvn_pricing_` / `hvnPricing` | `mjet_pricing_` / `mjetPricing` |
| Alpine components | `hvnPricingToolbar()`, `hvnConfigManager()`, `hvnConfigToolbar()` | `mjetPricingToolbar()`, `mjetConfigManager()`, `mjetConfigToolbar()` |
| Constants | `HVN_PRICING_DIR` | `MJET_PRICING_DIR` |
| Author | HVN GROUP | ModuleJET |
| Author URL | hvn.vn | modulejet.com |
| Support email | support@hvn.vn | support@modulejet.com |
| whmcs.json name | `hvn-pricing-calculator` | `modulejet-pricing-calculator` |
| JS filename | `hvn-pricing.js` | `mjet-pricing.js` |
| CSS filename | `hvn-pricing.css` | `mjet-pricing.css` |

**Migration logic:** Trong `_activate()`, detect bảng cũ `tbl_hvn_pricing_presets` → copy data sang `tbl_mjet_pricing_presets` → giữ bảng cũ (không drop, để user có thể rollback).

### 4.2 Tích hợp License Verification

#### 4.2.1 Flow hoạt động

```
Module activate/load
  └→ Đọc license key từ module settings (admin nhập khi config)
  └→ Đọc local key từ DB (tbladdonmodules, setting 'localKey')
  └→ Gọi check_license() verify với modulejet.com
      ├─ Active     → Module hoạt động bình thường
      ├─ Invalid    → Hiển thị warning, disable toolbar injection
      ├─ Suspended  → Thông báo license bị suspend, link client area
      └─ Expired    → Thông báo hết hạn, link renew
```

#### 4.2.2 Thêm config fields

```php
// Trong modulejet_pricing_calculator_config()
'fields' => [
    'licenseKey' => [
        'FriendlyName' => 'License Key',
        'Type'         => 'text',
        'Size'         => '50',
        'Default'      => '',
        'Description'  => 'Enter your ModuleJET license key. '
                        . '<a href="https://modulejet.com" target="_blank">Purchase</a>',
    ],
    // ... existing fields giữ nguyên ...
],
```

#### 4.2.3 License check class

Tạo file `lib/LicenseChecker.php` — dựa trên `check_sample_code.php` của WHMCS:

```php
<?php
namespace ModuleJET\PricingCalculator;

class LicenseChecker
{
    private string $licenseKey;
    private string $localKey;
    private string $whmcsUrl = 'https://modulejet.com/whmcs/';
    private string $licensingSecretKey = ''; // từ WHMCS licensing config
    
    private int $localKeyDays = 15;       // Local key valid 15 ngày
    private int $allowCheckFailDays = 5;  // Grace period 5 ngày nếu remote fail

    public function check(): array
    {
        // Return: ['status' => 'Active|Invalid|Expired|Suspended',
        //          'message' => '...', 'addons' => '...', 
        //          'configoptions' => '...', 'localkey' => '...']
    }
    
    public function isValid(): bool
    {
        $result = $this->check();
        return $result['status'] === 'Active';
    }
    
    public function getTier(): string
    {
        // Parse key prefix: MJET-PC-STD- | MJET-PC-PRO- | MJET-PC-UEN-
        if (str_starts_with($this->licenseKey, 'MJET-PC-STD-')) return 'standard';
        if (str_starts_with($this->licenseKey, 'MJET-PC-PRO-')) return 'professional';
        if (str_starts_with($this->licenseKey, 'MJET-PC-UEN-')) return 'unencoded';
        return 'unknown';
    }
    
    public function getProduct(): string
    {
        // Parse: MJET-{PRODUCT}-... → 'PC', 'IM', etc.
        preg_match('/^MJET-([A-Z]{2,4})-/', $this->licenseKey, $m);
        return $m[1] ?? 'UNKNOWN';
    }
}
```

#### 4.2.4 Integration points

**hooks.php — license gate trước toolbar injection:**

```php
add_hook('AdminAreaFooterOutput', 1, function($vars) {
    // ... existing page detection ...
    
    $settings = mjet_pricing_getSettings();
    $licenseKey = $settings['licenseKey'] ?? '';
    
    if (empty($licenseKey)) {
        return mjet_renderLicenseNotice('no_key');
    }
    
    $checker = new LicenseChecker($licenseKey, $settings['localKey'] ?? '');
    $result = $checker->check();
    
    // Store refreshed local key
    if (!empty($result['localkey'])) {
        mjet_saveLocalKey($result['localkey']);
    }
    
    if ($result['status'] !== 'Active') {
        return mjet_renderLicenseNotice($result['status'], $result);
    }
    
    // ... existing toolbar injection (unchanged) ...
});
```

**Dashboard — license status card:**

```php
// modulejet_pricing_calculator_output() → renderDashboard()
// Thêm card hiển thị: License Key (masked), Status, Tier, Expiry
```

#### 4.2.5 Caching strategy

- Local key lưu trong `tbladdonmodules` (setting `localKey` của module)
- Remote check chỉ mỗi 15 ngày (hoặc khi local key hết hạn)
- Grace period 5 ngày nếu server modulejet.com unreachable
- Sau 5 ngày không verify → hiển thị warning nhưng **không crash WHMCS**

### 4.3 Hai bản phân phối

#### Bản Encoded (Standard + Professional)

```
modulejet_pricing_calculator/
├── modulejet_pricing_calculator.php     ← ionCube encoded
├── hooks.php                            ← ionCube encoded
├── lib/
│   └── LicenseChecker.php              ← ionCube encoded
├── assets/
│   ├── js/
│   │   ├── mjet-pricing.js             ← Plain (obfuscate tùy chọn)
│   │   └── vendor/
│   │       └── alpine.min.js           ← Plain (3rd party, giữ nguyên)
│   └── css/
│       └── mjet-pricing.css            ← Plain
├── templates/
│   ├── admin-dashboard.php             ← Plain
│   ├── admin-presets.php               ← Plain
│   └── config-options-manager.php      ← Plain
├── lang/
│   ├── english.php                     ← Plain
│   └── vietnamese.php                  ← Plain
├── whmcs.json                           ← Plain
├── logo.png                             ← Plain
├── LICENSE                              ← Proprietary
└── README.md                            ← Plain
```

**ionCube Encoding:**
- Target: PHP 8.1+
- Files encode: `modulejet_pricing_calculator.php`, `hooks.php`, `lib/LicenseChecker.php`
- Files KHÔNG encode: `assets/`, `templates/`, `lang/`, `whmcs.json`, `logo.png`

#### Bản Unencoded

Cùng cấu trúc, tất cả file `.php` ở dạng plain text. License vẫn proprietary (thương mại), license check vẫn có nhưng khách có thể đọc/sửa source.

---

## 5. Quy Trình Mua & Kích Hoạt License

### 5.1 Customer Journey

```
1. Khách truy cập modulejet.com → xem trang Pricing Calculator
2. So sánh plans (Standard / Professional / Unencoded)
3. Chọn plan + billing cycle → Add to Cart → Checkout
4. Thanh toán (PayPal, Stripe)
5. WHMCS tự động generate license key (MJET-PC-XXX-...)
6. Khách nhận email:
   - License key
   - Link download module (encoded hoặc unencoded tùy plan)
   - Hướng dẫn cài đặt
7. Khách upload module vào WHMCS → Activate
8. Nhập license key vào Configure → Save
9. Module verify license → hoạt động
```

### 5.2 Client Area Self-Service

- View license key + trạng thái (Active/Suspended/Expired)
- Reissue license khi đổi domain (max 3 lần)
- Download module (require product Active)
- Upgrade license tier hoặc billing cycle
- Manage billing, renew

### 5.3 Upgrade Paths

| Từ | Đến | Cách |
|---|---|---|
| Standard Monthly → Annual/Lifetime | Same tier, longer cycle | WHMCS Product Upgrade |
| Standard → Professional | Higher tier | WHMCS Product Upgrade (prorate) |
| Professional Monthly → Annual/Lifetime | Same tier, longer cycle | WHMCS Product Upgrade |
| Any Encoded → Unencoded | Different distribution | WHMCS Product Upgrade (prorate) |
| Unencoded Annual → Lifetime | Same tier, longer cycle | WHMCS Product Upgrade |

---

## 6. Website modulejet.com

### 6.1 Pages

```
modulejet.com/
├── Home (landing page — sản phẩm đầu tiên = Pricing Calculator)
├── Products/
│   └── Pricing Calculator/
│       ├── Features & Screenshots
│       ├── Pricing (comparison table 3 tiers)
│       ├── Documentation
│       └── Changelog
├── Client Area (WHMCS)
│   ├── My Licenses, Downloads, Support Tickets, Invoices
├── Knowledge Base (installation, FAQ, troubleshooting)
├── Support (WHMCS ticket system)
├── Terms of Service
├── Privacy Policy
└── Refund Policy (gợi ý 30 ngày money-back)
```

### 6.2 WHMCS Setup

- Payment Gateways: PayPal + Stripe (tối thiểu)
- Email Templates: customize welcome email (license key + download link)
- Support Department: "Pricing Calculator Support"
- Knowledgebase: Installation guide, FAQ, Troubleshooting

---

## 7. ionCube Encoding

### 7.1 Build script — Encoded version

```bash
#!/bin/bash
# build-encoded.sh

SRC="./src/modulejet_pricing_calculator"
OUT="./build/encoded/modulejet_pricing_calculator"
IONCUBE="/path/to/ioncube_encoder"
VERSION="1.2.0"

rm -rf "./build/encoded"
mkdir -p "$OUT"

# Copy toàn bộ trước
cp -r "$SRC/"* "$OUT/"

# Encode các file PHP cần bảo vệ (ghi đè lên plain)
$IONCUBE \
  --php81 \
  --optimize max \
  --obfuscate all \
  --obfuscation-key "modulejet-secret" \
  --into "$OUT" \
  "$SRC/modulejet_pricing_calculator.php" \
  "$SRC/hooks.php" \
  "$SRC/lib/LicenseChecker.php"

# Package
cd "./build/encoded"
zip -r "../modulejet-pricing-calculator-encoded-v${VERSION}.zip" \
  modulejet_pricing_calculator/
```

### 7.2 Build script — Unencoded version

```bash
#!/bin/bash
# build-unencoded.sh

SRC="./src/modulejet_pricing_calculator"
OUT="./build/unencoded/modulejet_pricing_calculator"
VERSION="1.2.0"

rm -rf "./build/unencoded"
mkdir -p "$OUT"

cp -r "$SRC/"* "$OUT/"

# Replace LICENSE file với proprietary commercial
cp ./licenses/COMMERCIAL.txt "$OUT/LICENSE"

cd "./build/unencoded"
zip -r "../modulejet-pricing-calculator-unencoded-v${VERSION}.zip" \
  modulejet_pricing_calculator/
```

### 7.3 Test matrix

| PHP | WHMCS | ionCube Loader | Status |
|---|---|---|---|
| 8.1 | 8.x | 13.0.2+ | Must pass |
| 8.2 | 8.x / 9.x | 13.0.2+ | Must pass |
| 8.3 | 9.x | 14.4.0+ | Must pass |

---

## 8. Kế Hoạch Triển Khai (Timeline)

### Phase 1 — Rebrand & License Integration (Tuần 1–2)

| # | Task | Ưu tiên | Est. |
|---|---|---|---|
| 1.1 | Rebrand codebase: tất cả files hvn_ → modulejet_/mjet_ | Critical | 1 ngày |
| 1.2 | Rename JS: hvn-pricing.js → mjet-pricing.js, update all references | Critical | 0.5 ngày |
| 1.3 | Rename CSS: hvn-pricing.css → mjet-pricing.css, hvn- classes → mjet- | Critical | 0.5 ngày |
| 1.4 | Migration logic: detect tbl_hvn → copy to tbl_mjet | Critical | 0.5 ngày |
| 1.5 | Tạo `lib/LicenseChecker.php` (từ WHMCS check_sample_code.php) | Critical | 1 ngày |
| 1.6 | Thêm `licenseKey` field vào `_config()` | Critical | 0.5 ngày |
| 1.7 | License gate trong hooks.php (check trước khi inject toolbar) | Critical | 1 ngày |
| 1.8 | License status card trên dashboard | High | 0.5 ngày |
| 1.9 | License warning/expired/invalid UI | High | 0.5 ngày |
| 1.10 | Local key caching (lưu vào tbladdonmodules) | High | 0.5 ngày |
| 1.11 | Graceful degradation (không crash khi invalid) | High | 0.5 ngày |
| 1.12 | Update whmcs.json, lang files, README | Medium | 0.5 ngày |
| 1.13 | Test license check tất cả status (Active/Invalid/Expired/Suspended) | Critical | 1 ngày |

### Phase 2 — WHMCS Setup trên modulejet.com (Tuần 2–3)

| # | Task | Ưu tiên | Est. |
|---|---|---|---|
| 2.1 | Cài đặt WHMCS trên modulejet.com | Critical | 1 ngày |
| 2.2 | Mua & activate Software Licensing Addon | Critical | 0.5 ngày |
| 2.3 | Tạo 8 products theo spec (prefix, domain rules, billing) | Critical | 1 ngày |
| 2.4 | Cấu hình upgrade paths giữa các products | High | 0.5 ngày |
| 2.5 | Setup PayPal + Stripe | Critical | 0.5 ngày |
| 2.6 | Customize email templates | High | 1 ngày |
| 2.7 | Setup download manager (2 packages: encoded + unencoded) | High | 0.5 ngày |
| 2.8 | Tạo knowledgebase (install guide, FAQ, troubleshoot) | Medium | 1 ngày |
| 2.9 | Setup support department + ticket templates | Medium | 0.5 ngày |

### Phase 3 — ionCube Encoding & Build (Tuần 3)

| # | Task | Ưu tiên | Est. |
|---|---|---|---|
| 3.1 | Mua/setup ionCube Encoder | Critical | — |
| 3.2 | Tạo build-encoded.sh script | Critical | 0.5 ngày |
| 3.3 | Tạo build-unencoded.sh script | High | 0.5 ngày |
| 3.4 | Test encoded module trên PHP 8.1/8.2/8.3 | Critical | 1 ngày |
| 3.5 | Test trên WHMCS 8.x và 9.x | Critical | 1 ngày |
| 3.6 | Version tagging + CHANGELOG.md | Medium | 0.5 ngày |

### Phase 4 — Website & Launch (Tuần 4)

| # | Task | Ưu tiên | Est. |
|---|---|---|---|
| 4.1 | Landing page modulejet.com | High | 2 ngày |
| 4.2 | Pricing comparison table (3 tiers) | High | 0.5 ngày |
| 4.3 | Documentation pages | High | 1 ngày |
| 4.4 | Terms of Service + Privacy Policy + Refund Policy | Critical | 1 ngày |
| 4.5 | End-to-end test: order → pay → download → install → activate | Critical | 1 ngày |
| 4.6 | Soft launch → Public launch | — | — |

**Tổng: ~4 tuần (20 working days)**

---

## 9. Cấu Trúc Thư Mục Dự Án

### 9.1 Source (development)

Phản ánh đúng cấu trúc module hiện tại sau rebrand:

```
modulejet-pricing-calculator/          ← Project root (repo)
│
├── src/
│   └── modulejet_pricing_calculator/  ← Module source code
│       │
│       ├── modulejet_pricing_calculator.php
│       │     # Main addon file
│       │     #   _config(), _activate(), _deactivate(), _upgrade()
│       │     #   _output() — AJAX handler + page router
│       │     #   getConfigOptionsData(), saveConfigOptionsPricing()
│       │     #   renderDashboard(), renderPresets()
│       │
│       ├── hooks.php
│       │     # Hook registrations + license gate
│       │     #   AdminAreaHeaderOutput  → load CSS (normal pages)
│       │     #   AdminAreaHeadOutput    → load CSS + JS (popup pages)
│       │     #   AdminAreaFooterOutput  → license check + inject toolbar
│       │     #   AdminProductConfigFieldsSave → save config options pricing
│       │
│       ├── lib/
│       │   └── LicenseChecker.php     ← MỚI (license verification)
│       │
│       ├── assets/
│       │   ├── js/
│       │   │   ├── mjet-pricing.js    # Main JS (IIFE)
│       │   │   │     # Utils, Undo, Calc, PricingDOM
│       │   │   │     # Alpine: mjetPricingToolbar(), mjetConfigManager()
│       │   │   │     # buildToolbarHtml(), buildEmbeddedToolbar(), boot()
│       │   │   └── vendor/
│       │   │       └── alpine.min.js  # Alpine.js 3.14.8 local fallback
│       │   └── css/
│       │       └── mjet-pricing.css   # Ant Design–inspired pure CSS
│       │
│       ├── templates/
│       │   ├── admin-dashboard.php    # Dashboard: guide, license status
│       │   ├── admin-presets.php      # Preset management UI
│       │   └── config-options-manager.php  # Inline config options
│       │
│       ├── lang/
│       │   ├── english.php
│       │   └── vietnamese.php
│       │
│       ├── whmcs.json                 # WHMCS Marketplace metadata
│       ├── logo.png
│       └── LICENSE
│
├── build/                             ← Build output (gitignored)
│   ├── modulejet-pricing-calculator-encoded-v1.2.0.zip
│   └── modulejet-pricing-calculator-unencoded-v1.2.0.zip
│
├── scripts/
│   ├── build-encoded.sh
│   └── build-unencoded.sh
│
├── licenses/
│   └── COMMERCIAL.txt                 # Proprietary license text
│
├── docs/
│   ├── installation.md
│   ├── configuration.md
│   ├── licensing-faq.md
│   └── CHANGELOG.md
│
└── README.md                          # Repo README
```

### 9.2 Distribution (delivered to customer)

Customer nhận chính xác nội dung trong `src/modulejet_pricing_calculator/` — upload trực tiếp vào `/modules/addons/` của WHMCS.

---

## 10. Checklist Trước Khi Launch

### Technical
- [ ] Rebrand hoàn tất: không còn reference nào đến "hvn" trong code
- [ ] License check hoạt động với cả 3 key prefixes (STD, PRO, UEN)
- [ ] Module hoạt động bình thường khi license Active
- [ ] Warning rõ ràng khi license Invalid/Expired/Suspended
- [ ] Graceful degrade — không crash WHMCS khi license check fail
- [ ] ionCube encoded files chạy trên PHP 8.1 / 8.2 / 8.3
- [ ] Tương thích WHMCS 8.0+ và 9.x
- [ ] Local key caching (không remote check mỗi page load)
- [ ] Reissue hoạt động từ client area
- [ ] Download enforce đúng (Active product = download OK)
- [ ] Upgrade paths hoạt động (Standard→Pro, Monthly→Annual→Lifetime)
- [ ] Build scripts tạo đúng 2 bản
- [ ] Migration logic: cài mới + upgrade từ bản HVN

### Business
- [ ] Payment gateways test OK
- [ ] Welcome email có license key + download link
- [ ] Client area hiển thị license info
- [ ] Knowledgebase: Installation, FAQ, Troubleshooting
- [ ] Terms of Service (bao gồm: Unencoded ≠ open source, cấm redistribute)
- [ ] Privacy Policy
- [ ] Refund Policy (30 ngày money-back)
- [ ] Support ticket system hoạt động
- [ ] Public license verification page hoạt động

---

## 11. Rủi Ro & Giải Pháp

| Rủi ro | Mức độ | Giải pháp |
|---|---|---|
| ionCube Encoder giá cao | Medium | Dùng online encoding service trước, mua full khi scale |
| Khách thiếu ionCube Loader | Medium | Hướng dẫn chi tiết trong KB, check version trong module |
| License server down | High | Local key 15 ngày + grace 5 ngày |
| Bản encoded bị crack | Low | Chấp nhận, focus vào giá trị support/updates |
| Bản unencoded bị redistribute | Medium | Proprietary license + DMCA enforcement + community trust |
| PHP version mới | Medium | Test mỗi PHP release, update encoding targets |

---

## 12. Mở Rộng Cho Sản Phẩm Tương Lai

Khi ModuleJET ra sản phẩm mới (ví dụ: "Invoice Manager"):

1. **Key prefix:** Tạo prefix mới `MJET-IM-STD-`, `MJET-IM-PRO-`, `MJET-IM-UEN-`
2. **Products:** Tạo product group mới "ModuleJET — Invoice Manager" với cùng 8 tier/cycle
3. **LicenseChecker:** Tái sử dụng class, chỉ cần thay `getProduct()` check → `'IM'`
4. **Build scripts:** Clone và sửa tên module
5. **Infrastructure:** Tận dụng toàn bộ WHMCS instance, payment gateways, client portal đã có

---

*Tài liệu này là kế hoạch triển khai cho sản phẩm đầu tiên của ModuleJET. Infrastructure xây dựng ở đây (WHMCS, payment, portal, key convention) sẽ tái sử dụng cho tất cả sản phẩm sau.*
