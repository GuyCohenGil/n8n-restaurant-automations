<div dir="rtl">

# בסיס Airtable להזמנות (01a–01c)

workflows ‏01a, ‏01b ו-01c קוראים וכותבים לבסיס Airtable אחד עם חמש טבלאות. הצמתים מפנים לטבלאות לפי השם, אז שמות הטבלאות והשדות צריכים להיות בדיוק כמו כאן.

## התקנה

1. יוצרים בסיס חדש ב-Airtable, ובו חמש טבלאות לפי הפירוט למטה.
2. ב-n8n מוסיפים credential מסוג Airtable OAuth2 (או Personal Access Token עם הרשאות `data.records:read` ו-`data.records:write` לבסיס הזה).
3. בכל צומת Airtable בוחרים את ה-credential ומחליפים את `YOUR_AIRTABLE_BASE_ID` ב-ID של הבסיס (מתחיל ב-`app`, מופיע בכתובת).

## איך מחשבים כמות להזמנה

- **ימי כיסוי:** כמה ימים יש מהיום עד יום ההזמנה הבא של אותו ספק (לפי `Order Days`, במעגל של שבוע).
- **כמות מוצעת:** צריכה יומית × ימי כיסוי + מלאי ביטחון − מה שנספר, מחולק ב-`Pack Size` ומעוגל למעלה. אם יוצא 0 או פחות, הפריט לא נכנס להזמנה.
- **שורה ידנית** (אירוע, ספיישל) נכנסת עם `Source = Manual` וסיבה. שורות עם סיבה לא נכנסות לחישוב הצריכה היומית.

## טבלאות

### Suppliers

| שדה | סוג | ערכים / הערה |
| --- | --- | --- |
| Supplier | Single line text (ראשי) | שם הספק, כפי שנשלח ב-webhook |
| Category | Single select | ירקות ופירות, בשר ועוף, דגים, מוצרי חלב, יבשים |
| Order Days | Multiple select | ראשון, שני, שלישי, רביעי, חמישי, שישי, שבת |
| Lead Days | Number | ימים מההזמנה עד האספקה. 1 = מגיע מחר |
| Cutoff | Single line text | שעת סגירת הזמנות, `HH:MM` |
| Channel | Single select | WhatsApp, Email, Phone |
| Contact Phone | Phone | |
| Contact Email | Email | |
| Active | Checkbox | |
| Items | Link to Items | |
| Orders | Link to Orders | |

### Items

| שדה | סוג | ערכים / הערה |
| --- | --- | --- |
| Item | Single line text (ראשי) | השם שהצוות כותב בספירה |
| Storage | Single select | מקרר, מקפיא, מחסן יבש, חדר ירקות |
| Count Unit | Single select | ק"ג, ליטר, יחידה |
| Pack Name | Single line text | יחידת הקנייה מהספק, למשל ארגז |
| Pack Size | Number | כמה יחידות ספירה ביחידת קנייה אחת |
| Daily Usage | Number | צריכה יומית ממוצעת ביחידות ספירה |
| Safety Stock | Number | מלאי ביטחון ביחידות ספירה |
| Sort | Number | סדר בתוך מקום האחסון, לפי מסלול הספירה |
| Active | Checkbox | |
| Supplier | Link to Suppliers | ספק ראשי |
| Supplier Name | Lookup (Supplier ← Supplier) | משמש לסינון הפריטים לפי ספק |
| Counts | Link to Counts | |
| Order Lines | Link to Order Lines | |

### Counts

| שדה | סוג |
| --- | --- |
| Count | Single line text (ראשי) |
| Date | Date |
| Qty | Number |
| Counted By | Single line text |
| Item | Link to Items |

### Orders

| שדה | סוג | ערכים / הערה |
| --- | --- | --- |
| Order | Single line text (ראשי) | למשל "ירקות השרון – 16/10" |
| Order Date | Date | |
| Delivery Date | Date | |
| Status | Single select | Draft, Approved, Sent, Received, Cancelled |
| Message | Long text | נוסח ההודעה לספק |
| Supplier | Link to Suppliers | |
| Order Lines | Link to Order Lines | |

### Order Lines

| שדה | סוג | ערכים / הערה |
| --- | --- | --- |
| Line | Single line text (ראשי) | |
| On Hand | Number | מה שנספר |
| Coverage Days | Number | ימים עד המשלוח הבא |
| Suggested Packs | Number | החישוב של המערכת |
| Final Packs | Number | מה שהמנהל אישר |
| Source | Single select | Count, Manual |
| Reason | Single select | אירוע, ספיישל, אחר. חובה בשורה ידנית |
| Order | Link to Orders | |
| Item | Link to Items | |

> בצמתי Airtable הערכים של Single select נכתבים כטקסט עם `typecast`, אז שם הערך צריך להיות זהה לרשימה כאן.

</div>
