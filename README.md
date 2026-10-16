# Лаборатори №6: Тест тохиолдол боловсруулах — Category-Partition ба PICT

* **Оюутны нэр:** А. Төгөлдөр
* **Оюутны код:** B242270001
* **Хичээл:** F.CSA313 — Программ хангамжийн чанарын баталгаа ба тест (2026)

---

## Хэсэг A: Category-Partition гараар (Шилжүүлгийн функц)

### A1. Сонголт ба төлөөлөх утгууд

Бүх тестэд дараах **суурь орчинг** (далд сонголт) ашиглана: ханш 1 USD = 3,000 MNT; `acc_101` үлдэгдэл 10,000 MNT; өдрийн лимит 5,000 MNT (тестийн эхэнд ашигласан дүн 0); `acc_102` идэвхтэй; `acc_frozen` царцаасан.

| Параметр | Сонголтууд | Төлөөлөх утгууд |
| :--- | :--- | :--- |
| **from** | Valid, Frozen, Missing | `acc_101`, `acc_frozen`, `""` |
| **to** | Valid, Missing, SameAsFrom | `acc_102`, `""`, `acc_101` (from-той ижил) |
| **amount** | Normal, Zero, Negative, OverBalance, OverLimit | `500`, `0`, `-50`, `50000` (үлдэгдэл 10,000-аас их), `6000` (лимит 5,000-аас их, үлдэгдлээс бага) |
| **currency** | Same, Different | `"MNT"` (дансны валюттай адил), `"USD"` (өөр, хөрвүүлэлт шаардана) |
| **Далд орчин** | Өдрийн лимит хэр дүүрсэн, дансны үлдэгдэл | Лимит хоосон (0 ашигласан) гэж тогтооно. OverLimit нь үлдэгдлээс бага байх ёстой, эс бөгөөс ERROR_INSUFFICIENT түрүүлнэ. |

### A2. Хязгаарлалтууд ба "N → M"

| Параметр | Утга | Тэмдэглэгээ | Үндэслэл |
| :--- | :--- | :---: | :--- |
| from | Valid | — | Хэвийн утга. |
| from | Frozen | [ERROR] | Тодорхойлолтоор ERROR_FROZEN буцна. |
| from | Missing | [ERROR], [IF] | ERROR_NO_FROM буцна; from байхгүй үед amount-ийг шалгах утгагүй тул Amount = Normal болгоно. |
| to | Missing | [ERROR], [IF] | ERROR_NO_TO буцна; Amount = Normal болгоно. |
| to | SameAsFrom | [ERROR], [IF] | ERROR_SAME_ACCOUNT буцна; from байхгүй бол "from-той ижил" гэх ойлголт үүсэхгүй тул Missing-тэй хамт орохгүй. |
| amount | Zero | [ERROR] | amount ≤ 0 тул ERROR_BAD_AMOUNT. |
| amount | Negative | [ERROR], [SINGLE] | Zero-той ижил ERROR_BAD_AMOUNT өгдөг тул нэг л удаа шалгахад хангалттай. |
| amount | OverBalance | [ERROR] | Хөрвүүлсний дараах дүн үлдэгдлээс их тул ERROR_INSUFFICIENT. |
| amount | OverLimit | [ERROR] | Хөрвүүлсний дараах дүн өдрийн лимитээс их тул ERROR_LIMIT. |
| currency | Different | [SINGLE] | Хөрвүүлэлтийн логикийг нэг удаа шалгахад хангалттай. |

**Тооцоо:**

* Бүрэн олонлог: N = 3 × 3 × 5 × 2 = **90**
* [ERROR] утга бүр өөрийн кейстэй, бусад параметр нь хэвийн (Valid / Normal / Same):
  from 2 (Frozen, Missing) + to 2 (Missing, SameAsFrom) + amount 4 (Zero, Negative, OverBalance, OverLimit) = **8**
* Алдаагүй: Currency = Same (happy path) 1 + Currency = Different ([SINGLE]) 1 = **2**
* **N → M: 90 → 10.** Үүнд давхар нөхцөлтэй 3 кейс нэмж A3 хүснэгтэд нийт **13 спецификаци** болсон (10 + 3).

### A3. Спецификацийн хүснэгт ба давуу эрэмбэ

> **Давуу эрэмбэ:** `ERROR_NO_FROM` → `ERROR_FROZEN` → `ERROR_NO_TO` → `ERROR_SAME_ACCOUNT` → `ERROR_BAD_AMOUNT` → `ERROR_INSUFFICIENT` → `ERROR_LIMIT`
>
> **Үндэслэл:** Эхлээд данснуудыг (from → to) шалгаж, дараа нь дүнг шалгана. Учир нь үлдэгдэл, лимитийн шалгалт хүчинтэй from данс шаардана. Дүнгийн дотор утга буруу (≤ 0) байвал үлдэгдэл, лимитийг тооцох утгагүй тул BAD_AMOUNT → INSUFFICIENT → LIMIT дарааллаар явна.

| № | from | to | amount | currency | Тэмдэглэгээ | Oracle | Үндэслэл |
|:--|:--|:--|:--|:--|:--|:--|:--|
| 1 | Valid `acc_101` | Valid `acc_102` | Normal `500` | Same `MNT` | — | `201 OK` | Happy path |
| 2 | Missing `""` | Valid `acc_102` | Normal `500` | Same `MNT` | [ERROR][IF] | `200 ERROR_NO_FROM` | from байхгүй |
| 3 | Frozen `acc_frozen` | Valid `acc_102` | Normal `500` | Same `MNT` | [ERROR] | `200 ERROR_FROZEN` | from царцаасан |
| 4 | Valid `acc_101` | Missing `""` | Normal `500` | Same `MNT` | [ERROR][IF] | `200 ERROR_NO_TO` | to байхгүй |
| 5 | Valid `acc_101` | SameAsFrom `acc_101` | Normal `500` | Same `MNT` | [ERROR][IF] | `200 ERROR_SAME_ACCOUNT` | to = from |
| 6 | Valid `acc_101` | Valid `acc_102` | Zero `0` | Same `MNT` | [ERROR] | `200 ERROR_BAD_AMOUNT` | amount ≤ 0 |
| 7 | Valid `acc_101` | Valid `acc_102` | OverBalance `50000` | Same `MNT` | [ERROR] | `200 ERROR_INSUFFICIENT` | 50,000 > 10,000 |
| 8 | Valid `acc_101` | Valid `acc_102` | OverLimit `6000` | Same `MNT` | [ERROR] | `200 ERROR_LIMIT` | 6,000 ≤ 10,000 боловч > 5,000 лимит |
| 9 | Valid `acc_101` | Valid `acc_102` | Normal `1` | Different `USD` | [SINGLE] | `201 OK` | 1 USD = 3,000 MNT, лимит дотор |
| 10 | Frozen `acc_frozen` | Valid `acc_102` | Zero `0` | Same `MNT` | Давхар | `200 ERROR_FROZEN` | FROZEN ба BAD_AMOUNT зэрэг; from нь amount-аас өмнө |
| 11 | Valid `acc_101` | Missing `""` | OverBalance `50000` | Same `MNT` | Давхар | `200 ERROR_NO_TO` | NO_TO ба INSUFFICIENT зэрэг; to нь amount-аас өмнө |
| 12 | Valid `acc_101` | SameAsFrom `acc_101` | Zero `0` | Same `MNT` | Давхар | `200 ERROR_SAME_ACCOUNT` | SAME_ACCOUNT ба BAD_AMOUNT зэрэг; to нь amount-аас өмнө |
| 13 | Valid `acc_101` | Valid `acc_102` | Negative `-50` | Different `USD` | [SINGLE][ERROR] | `200 ERROR_BAD_AMOUNT` | Сөрөг дүн (Zero-той ижил анги) |

---

## Хэсэг B: PICT — Pairwise covering array

### Model файлууд

`browser.model`: Content, Cookies (Allow/Restrict/Block), Popups, Addons, Attacks, Forgeries (Yes/No).

`transfer-nc.model` (хязгаарлалтгүй):
```
From:     Valid, Frozen, Missing
To:       Valid, Missing, SameAsFrom
Amount:   Normal, Zero, OverBalance, OverLimit
Currency: Same, Different
```

`transfer.model` (хязгаарлалттай): дээрхтэй ижил утгууд, нэмэх нь
```
IF [From] = "Missing" THEN [Amount] = "Normal";
IF [To] = "Missing" THEN [Amount] = "Normal";
IF [From] = "Missing" THEN [To] <> "SameAsFrom";
```

`transfer-neg.model` ([ERROR] утгууд `~`-тэй):
```
From:     Valid, ~Frozen, ~Missing
To:       Valid, ~Missing, ~SameAsFrom
Amount:   Normal, ~Zero, ~OverBalance, ~OverLimit
Currency: Same, Different

IF [From] = "Missing" THEN [Amount] = "Normal";
IF [To] = "Missing" THEN [Amount] = "Normal";
```
(Хязгаарлалтад утгыг `~`-гүй бичнэ. `transfer`-ийн `From = "SameAsFrom"` гэсэн буруу мөрийг дээрх гурав дахь хязгаарлалтаар сольсон.) Бүх model-д `pict model 2>&1 >/dev/null` ажиллуулахад анхааруулга гарахгүй.

### B1. Мөрийн тооны харьцуулалт

* **2-way (browser):** PICT **10** мөр гаргасан, лекцийн гар шийдэл 9 мөр.
* **3-way (browser):** бүрэн олонлог 3 × 3 × 2 × 2 × 2 × 2 = **144**, PICT **20** мөр (≈ 14%).
* **Greedy:** PICT нь мөр бүрт үлдсэн хосуудаас хамгийн олныг нь хамруулах хамгийн ойрын сонголтыг хийдэг (greedy), нийт хамгийн цөөн мөрийг заавал олдоггүй. Тиймээс бүх хос хамрагдсан ч гараар оновчтой олсон 9 мөрөөс нэг мөр илүү (10) гарсан.

| Загвар | Түвшин | Нийт мөр | Happy path мөр | Нэг мөр дэх алдааны тоо | Файл |
|:--|:--:|:--:|:--:|:--:|:--|
| browser | 2-way | 10 | — | — | `results/browser-2way.txt` |
| browser | 3-way | 20 | — | — | `results/browser-3way.txt` |
| transfer-nc | 2-way | 13 | 0 | 1–3 (13 мөрийн 10-д ≥2) | `results/transfer-nc.txt` |
| transfer | 2-way | 11 | 0 | 1–3 (11 мөрийн 8-д ≥2) | `results/transfer.txt` |
| transfer-neg | 2-way | 16 | 2 | яг 1 (14 мөр), 0 (2 happy мөр) | `results/transfer-neg.txt` |

Алдааны тоо = From ≠ Valid, To ≠ Valid, Amount ≠ Normal гэсэн утгуудын тоо (мөр бүрээр гараар тоолсон).

**Ажиглалт:**

* `transfer-nc` 13 мөр, `transfer` 11 мөр: хязгаарлалт 2 мөр хассан.
* `transfer-nc` ба `transfer`-д happy path (Valid, Valid, Normal) мөр **гараагүй**. Мөр бүрт хамгийн ихдээ 3 алдаа зэрэг орсон тул error masking ихтэй, олон алдаатай мөрийн oracle нь A3-ын давуу эрэмбээс хамаарна.
* `transfer-neg` нь 2 happy path (мөр 1, 2) агуулж, бусад 14 мөр бүрд яг нэг [ERROR] утга бий.
* **`~` ашиглавал A2-ын хязгаарлалт хэрэгтэй юу?** Нэг мөрөнд хамгийн ихдээ нэг `~` утга орох тул `~Missing` нь өөр алдаатай хамт ордоггүй. Иймд Missing-ийн хоёр [IF] хязгаарлалт `transfer-neg`-д өөрчлөлт оруулахгүй, зөвхөн нөөц болж үлдсэн. Харин `transfer.model`-д хязгаарлалт шаардлагатай (13 → 11 мөр).

### B2. Бодит тест кейсүүд (PICT гаралтаас)

Суурь орчин: A1-д заасан (ханш 1 USD = 3,000 MNT, `acc_101` 10,000 MNT, лимит 5,000 MNT).

| № | Эх мөр | Setup | Input body | Oracle |
|:--|:--|:--|:--|:--|
| 1 | `transfer-neg.txt` мөр 1: Valid, Valid, Normal, Same (happy) | `acc_101` 10,000 MNT, `acc_102` идэвхтэй | `{"from":"acc_101","to":"acc_102","amount":500,"currency":"MNT"}` | `201 {"result":"OK"}` |
| 2 | `transfer-neg.txt` мөр 2: Valid, Valid, Normal, Different (happy) | Дээрхтэй ижил | `{"from":"acc_101","to":"acc_102","amount":1,"currency":"USD"}` | `201 {"result":"OK"}` (1 USD = 3,000 MNT) |
| 3 | `transfer-neg.txt` мөр 11: Valid, Valid, ~OverLimit, Different (нэг алдаатай) | Дээрхтэй ижил | `{"from":"acc_101","to":"acc_102","amount":2,"currency":"USD"}` | `200 {"result":"ERROR_LIMIT"}` (6,000 MNT > 5,000, ≤ 10,000) |
| 4 | `transfer.txt` мөр 10: Frozen, SameAsFrom, OverLimit, Same (олон алдаатай) | `acc_frozen` царцаасан, үлдэгдэл 10,000 MNT | `{"from":"acc_frozen","to":"acc_frozen","amount":6000,"currency":"MNT"}` | `200 {"result":"ERROR_FROZEN"}` |

Кейс 4: FROZEN, SAME_ACCOUNT, LIMIT гурван алдаа зэрэг байна. A3-ын давуу эрэмбэд FROZEN нь SAME_ACCOUNT, LIMIT-ээс түрүүлдэг тул ERROR_FROZEN буцна.

### B3. Гараар (A3) ба PICT-ийн харьцуулалт

| | Category-Partition (гар) | PICT |
|:--|:--|:--|
| Давуу тал | Алдааны семантик, давуу эрэмбэ, oracle-ийг бүрэн хянана; happy path ба SINGLE мөр тусгайлан орно | Хос бүрийн хамралтыг баталгаажуулна; хурдан, дахин ажиллуулахад ижил үр дүн |
| Сул тал | Хос хамралтыг баталгаажуулахгүй, хүнээс хамаарна | Oracle байхгүй; хязгаарлалтгүй бол happy path гардаггүй (13 ба 11 мөрөнд 0), олон алдаа зэрэг орно |

---

## Дүгнэлт

Гараар хийсэн category-partition нь алдааны семантик ба давуу эрэмбийг барьдаг: 90 боломжоос 10 үндсэн кейс, 3 давхар нөхцөлтэй нийт 13 спецификаци гаргаж, мөр бүрийн oracle-ийг эрэмбээр нь тодорхойлсон. PICT нь хос бүрийн хамралтыг баталгаажуулдаг ч oracle өгдөггүй. Browser загвар дээр 2-way 10 мөр гарч, гараар олсон 9 мөрөөс нэгээр илүү байсан нь greedy алгоритм хамгийн цөөнийг заавал олдоггүйг харуулсан. 3-way нь 144 комбинацийг 20 мөрөнд (≈ 14%) багтаасан. Хязгаарлалтгүй `transfer-nc` 13, хязгаарлалттай `transfer` 11 мөр гарсан бөгөөд хоёулангийнх нь happy path мөр 0, нэг мөрөнд хамгийн ихдээ 3 алдаа зэрэг орсон тул error masking үүссэн. `transfer-neg` нь 16 мөр, үүнээс 2 нь happy path, бусад 14 нь яг нэг алдаатай тул [ERROR] утгыг тусад нь шалгадаг category-partition-тэй дүйцсэн. `~` хэрэглэсэн үед Missing-ийн хязгаарлалт үр дүнд нөлөөлөхгүй болсон боловч `transfer.model`-д хязгаарлалт хэрэгтэй хэвээр. Олон алдаатай мөрийн oracle-ийг давуу эрэмбээс гаргасан нь (жишээ нь Frozen + SameAsFrom + OverLimit → ERROR_FROZEN) PICT-ийн гаралтыг бодит тест болгох тулд гар аргын дүрэм заавал хэрэгтэйг харуулсан. Хамгийн их бодол шаардсан алхам нь давуу эрэмбийг тогтоох, хязгаарлалтыг PICT синтаксаар зөв бичих (`~`-гүй, байхгүй утга ашиглахгүй) явдал байв. Иймд хоёр аргыг хослуулах нь хамгийн сайн: PICT хосын хамралтыг, гар арга oracle болон алдааны семантикийг хариуцна.
