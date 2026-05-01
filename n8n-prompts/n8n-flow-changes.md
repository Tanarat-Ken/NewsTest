# n8n Flow Changes Required — LineItems Architecture

## สรุปการเปลี่ยนแปลงที่ต้องทำใน n8n

### เหตุผล
OCR output เปลี่ยนจาก single row (Description/Qty/Price รวม) → LineItems array (รายแถว)
ต้องให้ n8n วน loop แล้ว insert Supabase **1 row ต่อ 1 line item**

---

## Supabase — Column ที่ต้องเพิ่ม

| Column ใหม่ | Type | คำอธิบาย |
|---|---|---|
| `LineNum` | integer | ลำดับ line ใน invoice (1, 2, 3…) |
| `UnitPrice` | numeric | ราคาต่อหน่วย (ต่างจาก LineAmount) |
| `LineAmount` | numeric | มูลค่าของ line นี้ (Qty × UnitPrice) |
| `POLineNum` | text | LINE_NUM ของ PO ที่ match |
| `PODescription` | text | ITEM_DESCRIPTION ของ PO ที่ match |
| `LineMatchStatus` | text | MATCHED / PARTIAL / NO_MATCH |
| `DocumentType` | text | ใบกำกับภาษี / ใบแจ้งหนี้ / ไม่ระบุ / ใบลดหนี้ |

> Column `Price` เดิม → ใช้เก็บ `LineAmount` แทน (repurpose, ไม่ต้องเพิ่มใหม่)
> Column `Description` เดิม → ใช้เก็บ line description (repurpose ได้เลย)
> Column `Qty` เดิม → ใช้เก็บ line qty (repurpose ได้เลย)

---

## n8n Node Changes

### 1. `Edit Fields` (หลัง AI Agent2)
ปัจจุบัน: parse `$json.output` เป็น JSON object เดียว
เปลี่ยนเป็น: parse แล้ว keep LineItems array ไว้

```javascript
// ไม่ต้องเปลี่ยนมาก — JSON.parse แล้ว LineItems จะติดมาเองใน $json
```

### 2. เพิ่ม node ใหม่: "Split LineItems" (Code node)
วางระหว่าง `Edit Fields1` → `Create a row1`

```javascript
// แยก LineItems ออกเป็น individual items พร้อม header fields
const parent = $input.first().json;
const lines = parent.LineItems ?? [];

return lines.map(line => ({
  json: {
    // Header fields (shared across all lines)
    Vendor:              parent.Vendor,
    PO:                  parent.PO,
    Invoice:             parent.Invoice,
    "Invoice Date":      parent["Invoice Date"],
    DocumentType:        parent.DocumentType,
    Discount:            parent.Discount,
    Vat:                 parent.Vat,
    "Total Amount":      parent["Total Amount"],
    Stamp_Signature:     parent.Stamp_Signature,
    Stamp_date:          parent.Stamp_date,
    Stamp_Date_Collect:  parent.Stamp_Date_Collect,
    Stamp_Received:      parent.Stamp_Received,
    "Stamp_ยอดสะสม":     parent["Stamp_ยอดสะสม"],
    "Stamp_ยอดคงค้าง":   parent["Stamp_ยอดคงค้าง"],
    Matched:             parent.Matched,
    ReceivingStatus:     parent.ReceivingStatus,
    Reason:              parent.Reason,
    Assessment:          parent.Assessment,
    ConfidenceLevel:     parent.ConfidenceLevel,
    DeliveryTimingStatus:parent.DeliveryTimingStatus,
    URL:                 parent.URL,
    // Line-level fields
    LineNum:             line.LineNum,
    Description:         line.Description,
    Qty:                 line.Qty,
    UnitPrice:           line.UnitPrice,
    LineAmount:          line.LineAmount,
    POLineNum:           line.POLineNum ?? "",
    PODescription:       line.PODescription ?? "",
    LineMatchStatus:     line.LineMatchStatus ?? "NO_MATCH"
  }
}));
```

### 3. `Create a row1` — เพิ่ม field mappings

```
LineNum          → $json.LineNum
UnitPrice        → $json.UnitPrice
LineAmount       → $json.LineAmount
POLineNum        → $json.POLineNum
PODescription    → $json.PODescription
LineMatchStatus  → $json.LineMatchStatus
DocumentType     → $json.DocumentType
Price            → $json.LineAmount        (repurpose เดิม)
Description      → $json.Description      (repurpose เดิม)
Qty              → $json.Qty              (repurpose เดิม)
```

---

## Flex Message (Code in JavaScript4)
ต้องปรับ: ปัจจุบัน read `row.Qty` / `row.Price` จาก single row
หลังปรับ: ต้องรวบ LineItems จากทุก row ที่มี Invoice เดียวกัน
หรือ: แสดงแค่ invoice-level summary (Total Amount, ReceivingStatus) ไม่ต้องแสดงราย line ใน flex

---

## ลำดับการปรับ (แนะนำ)
1. เพิ่ม column ใหม่ใน Supabase (migration)
2. อัปเดต Prompt 1 (OCR)
3. อัปเดต Prompt 2 (Agent2)
4. เพิ่ม "Split LineItems" Code node ใน n8n
5. ปรับ `Create a row1` field mappings
6. Test กับ invoice จริง 1-2 ใบ
