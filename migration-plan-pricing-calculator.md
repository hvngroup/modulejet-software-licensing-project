# ModuleJET — Pricing Calculator Licensing Plan

**Sản phẩm đầu tiên của ModuleJET**  
**Ngày:** 11/03/2026 — **v4.1**  
**Tham chiếu:** Platform Master Plan v1.1, Dev Standard v2.2.0

---

## 1. Tổng Quan

Rebrand `hvn_pricing_calculator` v1.1.0 → `mj_pricing_calculator`, tích hợp license verification, phân phối qua modulejet.com.

---

## 2. License Products

### 2.1 SKUs (base = $4.99)

| # | Tên | Billing | Tier | Key Prefix | Giá |
|---|---|---|---|---|---|
| 1 | PRC Monthly Standard | Monthly | STD | `MJ-PRC-STD-` | $4.99/mo |
| 2 | PRC Annually Standard | Annually | STD | `MJ-PRC-STD-` | $39.99/yr |
| 3 | PRC Lifetime Standard | One Time | STD | `MJ-PRC-STD-` | $99.99 |
| 4 | PRC Monthly Professional | Monthly | PRO | `MJ-PRC-PRO-` | $14.99/mo |
| 5 | PRC Annually Professional | Annually | PRO | `MJ-PRC-PRO-` | $119.99/yr |
| 6 | PRC Lifetime Professional | One Time | PRO | `MJ-PRC-PRO-` | $299.99 |
| 7 | PRC Annually Open Source | Annually | OSS | `MJ-PRC-OSS-` | $179.99/yr |
| 8 | PRC Lifetime Open Source | One Time | OSS | `MJ-PRC-OSS-` | $449.99 |

**Pricing formula:**
- STD Monthly = base ($4.99)
- STD Annually = base × 8
- PRO Monthly = base × 3
- PRO Annually = base × 24
- OSS Annually = base × 36
- Lifetime = Annually × 2.5 (mọi tier)

### 2.2 Tính năng theo tier

| Tính năng | Standard | Professional | Open Source |
|---|---|---|---|
| Pricing Calculator Toolbar | ✅ | ✅ | ✅ |
| Config Options Manager | ✅ | ✅ | ✅ |
| Presets & Settings | ✅ | ✅ | ✅ |
| Domain | 1 domain | Unlimited | Unlimited |
| Source code | ionCube | ionCube | Plain PHP |
| Sửa code | ❌ | ❌ | ✅ |
| Redistribute | ❌ | ❌ | ❌ |
| Branding removal | ❌ | ✅ | ✅ |
| White-label | ❌ | ❌ | ✅ |
| Support | 48h | 24h | 24h |

> **Open Source = proprietary license.** Code không mã hóa nhưng cấm phân phối lại.

### 2.3 Support/Updates

| Cycle | Policy |
|---|---|
| Monthly / Annually | Bao gồm khi active |
| Lifetime | **Trọn đời** |

---

## 3. WHMCS Config

### 3.1 Product Settings

**Chung:** Module `License Software`, Key Length `32`, Allow Reissue ✅, Support/Updates Addon **trống**.

**STD:** Prefix `MJ-PRC-STD-`, Domain Conflict ❌  
**PRO:** Prefix `MJ-PRC-PRO-`, Domain Conflict ✅  
**OSS:** Prefix `MJ-PRC-OSS-`, Domain Conflict ✅

### 3.2 Downloads

| File | Products |
|---|---|
| `mj-pricing-encoded-v1.2.0.zip` | STD + PRO |
| `mj-pricing-opensource-v1.2.0.zip` | OSS |

### 3.3 Upgrades

```
STD Monthly → STD Annually → STD Lifetime
PRO Monthly → PRO Annually → PRO Lifetime
OSS Annually → OSS Lifetime
STD → PRO → OSS (cross-tier, prorate)
```

---

## 4. Codebase Changes

### 4.1 Rebrand: hvn_ → mj_

| Cũ | Mới |
|---|---|
| `hvn_pricing_calculator/` | `mj_pricing_calculator/` |
| `hvn_pricing_calculator.php` | `mj_pricing_calculator.php` |
| `hvn_pricing_calculator_config()` | `mj_pricing_calculator_config()` |
| `hvn_pricing_` helpers | `mj_pricing_` helpers |
| `tbl_hvn_pricing_presets` | `tbl_mj_pricing_presets` |
| `hvn-` CSS classes | `mj-` CSS classes |
| `hvn-pricing.js` / `.css` | `mj-pricing.js` / `.css` |
| `window.hvnPricingConfig` | `window.mjPricingConfig` |
| `hvnPricingToolbar()` | `mjPricingToolbar()` |
| `HVN_PRICING_DIR` | `MJ_PRICING_DIR` |
| `hvn-pricing-calculator` (whmcs.json) | `mj-pricing-calculator` |
| HVN GROUP / hvn.vn | ModuleJET / modulejet.com |

Migration: `_activate()` detects `tbl_hvn_` → copies to `tbl_mj_` → keeps old table.

### 4.2 LicenseChecker

File: `lib/LicenseChecker.php`

```php
<?php
namespace MJ\PricingCalculator;

class LicenseChecker
{
    // ... (see Platform Plan v1.1 section 6.1)
    // Regex: /^MJ-[A-Z]{3}-(STD|PRO|OSS)-/
    // getTier(): STD→standard, PRO→professional, OSS→opensource
    // getProduct(): /^MJ-([A-Z]{3})-/
    // isCorrectProduct('PRC')
}
```

Config field `licenseKey` added. License gate in `hooks.php` before toolbar injection. Dashboard shows license status card.

### 4.3 Distribution

**Encoded (STD + PRO):** `mj_pricing_calculator.php`, `hooks.php`, `lib/LicenseChecker.php` → ionCube. Rest plain.

**Open Source (OSS):** All files plain PHP. Proprietary LICENSE file.

---

## 5. File Structure

```
mj_pricing_calculator/
├── mj_pricing_calculator.php    # Main: config, activate, output, AJAX
├── hooks.php                    # Hooks + license gate
├── lib/
│   └── LicenseChecker.php      # License verification
├── assets/
│   ├── js/
│   │   ├── mj-pricing.js       # Single IIFE
│   │   └── vendor/
│   │       └── alpine.min.js   # Alpine.js 3.14.8
│   └── css/
│       └── mj-pricing.css      # Design system
├── templates/
│   ├── admin-dashboard.php      # Dashboard + license card
│   ├── admin-presets.php        # Preset CRUD
│   └── config-options-manager.php
├── lang/
│   ├── english.php
│   └── vietnamese.php
├── whmcs.json
├── logo.png
└── LICENSE
```

---

## 6. Timeline

| Phase | Tasks | Est. |
|---|---|---|
| **Rebrand** | hvn_ → mj_ (all files, DB migration) | 2 ngày |
| **License** | LicenseChecker, gate, UI, caching | 2.5 ngày |
| **WHMCS** | 8 products, upgrades, downloads | 1 ngày |
| **Content** | Support dept, KB, product page | 2 ngày |
| **Build** | Encoded + open source + test matrix | 2 ngày |
| **Launch** | E2E test, soft launch, public launch | 1.5 ngày |
| **Total** | | **~11 ngày** |

(Assumes Platform Phase 0 already completed)

---

## 7. Checklist

- [ ] Zero `hvn` references in code
- [ ] License check works (Active/Invalid/Expired/Suspended/no key/wrong product)
- [ ] Graceful degrade — no WHMCS crash
- [ ] ionCube OK: PHP 8.1/8.2/8.3 + WHMCS 8.x/9.x
- [ ] DB migration from HVN works
- [ ] 8 products created, pricing correct (`MJ-PRC-{STD|PRO|OSS}`)
- [ ] Upgrade paths work
- [ ] Downloads enforce Active status
- [ ] E2E: order → pay → download → install → activate

---

*Pricing Calculator Licensing Plan v4.1 — `MJ-PRC-{STD|PRO|OSS}`*  
*Tham chiếu: Platform Master Plan v1.1, Dev Standard v2.2.0*
