# ModuleJET — Licensing Platform Master Plan

**Thương hiệu:** ModuleJET — modulejet.com  
**Email:** support@modulejet.com  
**Mục đích:** Platform licensing + distribution cho tất cả WHMCS modules của ModuleJET  
**Sản phẩm đầu tiên:** Pricing Calculator  
**Ngày lập:** 11/03/2026 — **v1.1**

---

## 1. Tầm Nhìn

ModuleJET bán WHMCS addon modules thương mại. Platform cần:

- Bán nhiều products trên cùng hệ thống
- Mỗi product: 3 tiers × 3 cycles = tối đa 8 SKUs
- Billing, client portal, support, downloads dùng chung
- Thêm product mới < 1 ngày (tạo products + upload files)
- License verification code tái sử dụng 100%

---

## 2. Kiến Trúc Platform

```
┌─────────────────────────────────────────────────────────────────┐
│                     KHÁCH HÀNG (N instances)                     │
│                                                                  │
│  WHMCS #1: mj_pricing_calculator ──┐                            │
│  WHMCS #2: mj_invoice_manager ─────┤  license check (HTTPS)     │
│  WHMCS #3: mj_server_manager ──────┘                            │
└─────────────────────┬───────────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                  MODULEJET.COM (1 WHMCS Instance)                │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │              Software Licensing Addon                       │ │
│  │  - Key generation (MJ-{PROD}-{TIER}-...)                  │ │
│  │  - Remote verification API                                 │ │
│  │  - License Manager + Public verification                   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌────────────────┐ │
│  │ Pricing Calc     │  │ Invoice Manager  │  │ Server Manager │ │
│  │ 8 SKUs           │  │ 8 SKUs           │  │ 8 SKUs         │ │
│  └─────────────────┘  └─────────────────┘  └────────────────┘ │
│                                                                  │
│  Payment (PayPal+Stripe) │ Client Portal │ Downloads │ Support  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. License Key Convention (Global)

### 3.1 Format

```
MJ-{PRODUCT}-{TIER}-{32_char_random}
```

| Segment | Values |
|---|---|
| `MJ` | Brand prefix (cố định) |
| `{PRODUCT}` | 3 uppercase letters (cố định 3 ký tự) |
| `{TIER}` | `STD` / `PRO` / `OSS` |

### 3.2 Product Registry

| Code | Sản phẩm | Status |
|---|---|---|
| `PRC` | Pricing Calculator | First product |
| `INV` | Invoice Manager | Planned |
| `SRV` | Server Manager | Planned |
| `EML` | Email Manager | Idea |
| `DMN` | Domain Manager | Idea |
| `BKP` | Backup Manager | Idea |

### 3.3 Tier Definitions (áp dụng cho MỌI product)

| Tier | Key | Encoding | Domain | Sửa code | Redistribute | White-label | Support |
|---|---|---|---|---|---|---|---|
| Standard | `STD` | ionCube | 1 domain | ❌ | ❌ | ❌ | Email 48h |
| Professional | `PRO` | ionCube | Unlimited | ❌ | ❌ | ❌ | Priority 24h |
| Open Source | `OSS` | Plain PHP | Unlimited | ✅ | ❌ | ✅ | Priority 24h |

> **"Open Source" naming:** Code không mã hóa, khách đọc/sửa được. License vẫn **proprietary** — cấm phân phối lại. Đây là tên thương mại (marketing name), không liên quan đến giấy phép GPL/MIT.

### 3.4 Billing Cycles (áp dụng cho MỌI product)

| Cycle | Thanh toán | Support/Updates |
|---|---|---|
| Monthly | Hàng tháng | Khi còn active |
| Annually | Hàng năm | Khi còn active |
| Lifetime | Một lần | **Trọn đời** |

Open Source không có Monthly (giá trị quá cao cho monthly billing).

### 3.5 SKU Matrix chuẩn per product (8 SKUs)

| # | SKU Pattern | Billing | Tier |
|---|---|---|---|
| 1 | `{Product} Monthly Standard` | Monthly | STD |
| 2 | `{Product} Annually Standard` | Annually | STD |
| 3 | `{Product} Lifetime Standard` | One Time | STD |
| 4 | `{Product} Monthly Professional` | Monthly | PRO |
| 5 | `{Product} Annually Professional` | Annually | PRO |
| 6 | `{Product} Lifetime Professional` | One Time | PRO |
| 7 | `{Product} Annually Open Source` | Annually | OSS |
| 8 | `{Product} Lifetime Open Source` | One Time | OSS |

---

## 4. Pricing Framework

### 4.1 Formula (base rate × multiplier)

| Tier | Monthly | Annually | Lifetime |
|---|---|---|---|
| Standard | `1× base` | `8× base` | `Annually × 2.5` = `20× base` |
| Professional | `3× base` | `24× base` | `Annually × 2.5` = `60× base` |
| Open Source | — | `36× base` | `Annually × 2.5` = `90× base` |

**Quy tắc chung:** Lifetime = Annually × 2.5 (cho mọi tier, mọi product).

### 4.2 Ví dụ: base = $4.99

| Tier | Monthly | Annually | Lifetime |
|---|---|---|---|
| Standard | $4.99 | $39.99 | $99.99 |
| Professional | $14.99 | $119.99 | $299.99 |
| Open Source | — | $179.99 | $449.99 |

> Giá round đến `.99` cho đẹp. Mỗi product có thể chọn base khác nhau tùy complexity.

### 4.3 Support/Updates Policy (global)

| Billing | Policy |
|---|---|
| Monthly / Annually | Bao gồm khi subscription active |
| Lifetime | **Trọn đời** — không cần addon riêng |

---

## 5. WHMCS Instance trên modulejet.com

### 5.1 One-time Setup (làm 1 lần)

| Hạng mục | Chi tiết |
|---|---|
| WHMCS License | Self-hosted hoặc Cloud |
| Software Licensing Addon | $99.95 one-time |
| Payment Gateways | PayPal + Stripe |
| Licensing Config | Reissues: 3, Prune: 90 days, Public verification: ✅ |
| Support Departments | "Billing & Licensing" + "Pre-Sales" (chung) |
| Email Templates | Shared, merge fields per product |
| Legal Pages | TOS, Privacy, Refund (30 ngày) |

### 5.2 Per-Product Setup (lặp lại mỗi product mới)

```
□ Đăng ký product code vào registry
□ Tạo Product Group: "ModuleJET — {Product Name}"
□ Tạo 8 products theo SKU matrix
□ Key prefix: MJ-{CODE}-STD- / MJ-{CODE}-PRO- / MJ-{CODE}-OSS-
□ Domain rules: STD = no domain conflict, PRO/OSS = allow
□ Pricing theo formula
□ Upgrade paths
□ Upload downloads (encoded + open source ZIPs)
□ Support department: "{Product Name} Support"
□ KB category + articles
□ Product landing page
□ E2E test
```

**Time per product: < 1 ngày** (infrastructure đã sẵn).

### 5.3 Product Groups trong WHMCS

```
├── ModuleJET — Pricing Calculator
│   ├── PC Monthly Standard ($4.99/mo)
│   ├── PC Annually Standard ($39.99/yr)
│   ├── PC Lifetime Standard ($99.99)
│   ├── PC Monthly Professional ($14.99/mo)
│   ├── PC Annually Professional ($119.99/yr)
│   ├── PC Lifetime Professional ($299.99)
│   ├── PC Annually Open Source ($179.99/yr)
│   └── PC Lifetime Open Source ($449.99)
│
├── ModuleJET — Invoice Manager          ← thêm khi ra product
│   └── ... (8 SKUs, base có thể khác)
│
└── ModuleJET — Server Manager
    └── ... (8 SKUs)
```

### 5.4 Module Settings per product

**Chung:**
- Module: `License Software`, Key Length: `32`, Allow Reissue: ✅
- Support/Updates Addon: **để trống**

**STD:** Key Prefix `MJ-{CODE}-STD-`, Allow Domain Conflict: ❌  
**PRO:** Key Prefix `MJ-{CODE}-PRO-`, Allow Domain Conflict: ✅  
**OSS:** Key Prefix `MJ-{CODE}-OSS-`, Allow Domain Conflict: ✅

### 5.5 Downloads

| File | Products |
|---|---|
| `mj-{product}-encoded-v{ver}.zip` | STD + PRO |
| `mj-{product}-opensource-v{ver}.zip` | OSS |

### 5.6 Upgrade Paths (chuẩn)

```
Within tier:  Monthly → Annually → Lifetime
Cross-tier:   STD → PRO → OSS
WHMCS auto-prorates.
```

---

## 6. Shared LicenseChecker

### 6.1 Class (template cho mọi module)

```php
<?php
namespace MJ\{ProductName};

class LicenseChecker
{
    private string $licenseKey;
    private string $localKey;
    private string $whmcsUrl          = 'https://modulejet.com/whmcs/';
    private string $licensingSecretKey = '{per-product-secret}';
    private int    $localKeyDays       = 15;
    private int    $allowCheckFailDays = 5;

    public function __construct(string $licenseKey, string $localKey = '') { ... }

    /** Based on WHMCS check_sample_code.php */
    public function check(): array { /* returns status, localkey, message */ }
    
    public function isValid(): bool { return $this->check()['status'] === 'Active'; }

    public function getTier(): string
    {
        if (preg_match('/^MJ-[A-Z]{3}-(STD|PRO|OSS)-/', $this->licenseKey, $m)) {
            return match($m[1]) {
                'STD' => 'standard',
                'PRO' => 'professional',
                'OSS' => 'opensource',
                default => 'unknown',
            };
        }
        return 'unknown';
    }

    public function getProduct(): string
    {
        preg_match('/^MJ-([A-Z]{3})-/', $this->licenseKey, $m);
        return $m[1] ?? 'UNKNOWN';
    }

    public function isCorrectProduct(string $expectedCode): bool
    {
        return $this->getProduct() === strtoupper($expectedCode);
    }
}
```

### 6.2 Integration pattern (mọi module)

```php
$checker = new \MJ\{Product}\LicenseChecker($key, $localKey);
if (!$checker->isCorrectProduct('{CODE}')) return notice('wrong_product');
$result = $checker->check();
if (!empty($result['localkey'])) saveLocalKey($result['localkey']);
if ($result['status'] !== 'Active') return notice($result['status']);
// → inject features
```

### 6.3 Caching

- Local key: `tbladdonmodules` setting `localKey`
- Remote check: mỗi 15 ngày
- Grace: 5 ngày nếu server unreachable
- Failure: warning banner, **không crash WHMCS**

---

## 7. Build System

### 7.1 Universal build.sh

```bash
#!/bin/bash
# Usage: ./build.sh <module_dir> <version>
# Example: ./build.sh mj_pricing_calculator 1.2.0

MODULE="$1"; VERSION="$2"
SRC="./products/${MODULE}/src/${MODULE}"

# Encoded
OUT="./build/encoded/${MODULE}"
rm -rf "./build/encoded" && mkdir -p "$OUT"
cp -r "$SRC/"* "$OUT/"
find "$SRC" -name "*.php" ! -path "*/templates/*" ! -path "*/lang/*" ! -path "*/vendor/*" \
  -print0 | xargs -0 ioncube_encoder --php81 --optimize max --obfuscate all --into "$OUT"
cd "./build/encoded" && zip -r "../${MODULE}-encoded-v${VERSION}.zip" "${MODULE}/" && cd ../..

# Open Source
OUT="./build/opensource/${MODULE}"
rm -rf "./build/opensource" && mkdir -p "$OUT"
cp -r "$SRC/"* "$OUT/"
cp ./shared/COMMERCIAL.txt "$OUT/LICENSE"
cd "./build/opensource" && zip -r "../${MODULE}-opensource-v${VERSION}.zip" "${MODULE}/" && cd ../..
```

### 7.2 ionCube Settings

Target PHP 8.1+, Obfuscation Level 4, No expiry, No callback.

### 7.3 Encode chỉ: main `.php`, `hooks.php`, `lib/*.php`. Giữ plain: `assets/`, `templates/`, `lang/`, metadata.

### 7.4 Test Matrix

| PHP | WHMCS | ionCube Loader |
|---|---|---|
| 8.1 | 8.x | 13.0.2+ |
| 8.2 | 8.x / 9.x | 13.0.2+ |
| 8.3 | 9.x | 14.4.0+ |

---

## 8. Monorepo Structure

```
modulejet/
├── products/
│   ├── mj_pricing_calculator/
│   │   └── src/mj_pricing_calculator/   ← module source
│   ├── mj_invoice_manager/             ← future
│   └── mj_server_manager/              ← future
│
├── shared/
│   ├── LicenseChecker.template.php      ← master, copy per module
│   ├── license-notice.css               ← CSS snippet
│   └── COMMERCIAL.txt                   ← proprietary license
│
├── scripts/
│   ├── build.sh                         ← universal
│   ├── build-all.sh
│   └── test-matrix.sh
│
├── build/                               ← gitignored
├── docs/
│   ├── platform/                        ← this doc, dev-standard, registry
│   └── products/                        ← per-product docs
└── README.md
```

---

## 9. Website modulejet.com

```
modulejet.com/
├── / (Home — product grid, grows with catalog)
├── /products/{name}/ (per-product: features, pricing, docs, CTA)
├── /docs/{name}/ (per-product: install, config, FAQ)
├── /clientarea/ (WHMCS: all licenses, downloads, tickets, invoices)
├── /support/
├── /verify/ (public license check)
└── /legal/ (TOS, privacy, refund, acceptable-use)
```

Per-product page template: Hero → Pain/Solution → Features → Pricing table → FAQ → CTA → Compatibility.

---

## 10. Support System

| Department | Scope |
|---|---|
| {Product Name} Support | Per product, tạo khi launch |
| Billing & Licensing | Chung |
| Pre-Sales Questions | Chung |

SLA: Standard 48h, Professional/Open Source 24h.

KB: per-product categories (Install, Config, FAQ, Troubleshoot) + General (ionCube, License Activation, Reissue, Upgrade, Billing).

---

## 11. Timeline

### Phase 0 — Platform Foundation (~12 ngày, làm 1 lần)

| # | Task | Est. |
|---|---|---|
| 0.1 | WHMCS install + SSL trên modulejet.com | 1 ngày |
| 0.2 | Software Licensing Addon mua + config | 1 ngày |
| 0.3 | PayPal + Stripe setup | 1 ngày |
| 0.4 | Email templates | 1 ngày |
| 0.5 | Support departments (Billing, Pre-Sales) | 0.5 ngày |
| 0.6 | General KB articles (5 bài) | 1 ngày |
| 0.7 | Legal pages (TOS, Privacy, Refund) | 1.5 ngày |
| 0.8 | ionCube Encoder setup | 0.5 ngày |
| 0.9 | Universal build.sh + test matrix | 1 ngày |
| 0.10 | Shared LicenseChecker.template.php | 1 ngày |
| 0.11 | Website: home, about, legal, product template | 3 ngày |

### Phase 1 — First Product: Pricing Calculator (~11 ngày)

| # | Task | Est. |
|---|---|---|
| 1.1 | Rebrand hvn_ → mj_ | 1.5 ngày |
| 1.2 | Migration tbl_hvn → tbl_mj | 0.5 ngày |
| 1.3 | LicenseChecker integration + UI | 2.5 ngày |
| 1.4 | 8 WHMCS products + upgrades + downloads | 1 ngày |
| 1.5 | Support dept + KB articles | 1 ngày |
| 1.6 | Product landing page | 1 ngày |
| 1.7 | Build + test encoded (PHP 8.1/8.2/8.3) | 2 ngày |
| 1.8 | E2E test + launch | 1.5 ngày |

### Phase N — Mỗi Product Tiếp Theo

| Task | Est. |
|---|---|
| Module development | Tùy complexity |
| LicenseChecker + WHMCS products + builds | ~3.5 ngày overhead |

---

## 12. Operations & KPIs

**KPIs:** MRR per product, activation rate >95%, support response STD<48h PRO<24h, refund <10%.

**Recurring:** Weekly license log review, monthly KB updates, per-release compatibility testing, daily DB backup.

**Alerts:** Payment error >5% → investigate, license failures spike → check server, ticket queue >20 → prioritize.

---

## 13. Risk Management

| Rủi ro | Giải pháp |
|---|---|
| Server down | Local key 15 ngày + grace 5 ngày |
| ionCube Loader missing | KB article + module installer check |
| Encoded cracked | Focus support/updates value |
| Open Source redistributed | Proprietary license + DMCA |
| New PHP breaks ionCube | Test early, re-encode promptly |
| Wrong product key used | `isCorrectProduct()` check |

---

## 14. Checklist

### One-Time Platform
- [ ] WHMCS + SSL on modulejet.com
- [ ] Software Licensing Addon
- [ ] Payment gateways (PayPal + Stripe)
- [ ] Email templates
- [ ] Support departments (Billing, Pre-Sales)
- [ ] General KB articles
- [ ] Legal pages (TOS, Privacy, Refund)
- [ ] ionCube Encoder
- [ ] build.sh working
- [ ] LicenseChecker.template.php
- [ ] Website home + template

### Per-Product Launch
- [ ] Product code registered
- [ ] Module + LicenseChecker integrated
- [ ] 8 WHMCS products (MJ-{CODE}-STD/PRO/OSS, code = 3 letters)
- [ ] Upgrade paths
- [ ] Encoded + open source ZIPs built + tested
- [ ] Downloads uploaded
- [ ] Support department + KB articles
- [ ] Product page live
- [ ] E2E test passed

---

*ModuleJET Platform v1.1 — March 2026*  
*Pricing: Lifetime = Annually × 2.5 | PRO Monthly = STD × 3 | PRO Annually = base × 24 | OSS Annually = base × 36*
