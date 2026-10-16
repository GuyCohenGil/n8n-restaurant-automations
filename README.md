<div dir="rtl">

# אוטומציות למסעדות ב-n8n

תהליכי עבודה שמסעדה מריצה כל יום, בנויים ב-n8n, ועוד workflow משותף לטיפול בשגיאות. תהליך ההזמנות (01) מחובר ל-Airtable ומחולק לשלושה workflows. כל workflow נבדק עם קלט תקין ועם קלט שבור.

נבנו על ידי Guy Cohen Gil, ‏[guycohengil.com](https://guycohengil.com): אוטומציות ו-AI לעסקי מזון, מאחד שבא מהמסעדנות.

> **נתוני הדגמה.** כל הספקים, המחירים והכמויות בדוגמאות האלה בדויים.

## מה יש כאן

| # | Workflow | מתי רץ | מה הוא עושה |
| --- | --- | --- | --- |
| 00 | ‏System – Error Alerts | כשworkflow אחר נכשל | שולח התראה בטלגרם: שם ה-workflow, הצומת שנכשל, השגיאה, השעה וקישור להרצה |
| 01a | ‏Daily Count List | כל יום ב-08:00 | בודק אילו ספקים מזמינים מהם היום, ושולח רשימת ספירה רק לפריטים שלהם, לפי מקום אחסון ושעת סגירה |
| 01b | ‏Count to Order Draft | כשהצוות שולח ספירה (Webhook) | מחשב כמות לכל פריט: צריכה יומית × ימים עד המשלוח הבא + מלאי ביטחון − מה שנספר, מעוגל ליחידת קנייה. שומר טיוטת הזמנה ב-Airtable ושולח למנהל |
| 01c | ‏Manual Order Line | כשמוסיפים שורה ידנית (Webhook) | מוסיף שורה לאירוע או לספיישל לטיוטה של היום, או פותח טיוטה חדשה מחוץ ליום ההזמנה. השורות האלה לא נכנסות לחישוב הצריכה |
| 02 | ‏Delivery Note Price Check | כשמגיעה תעודת משלוח (Webhook) | משווה כל שורה למחיר המוסכם עם הספק, ומתריע רק כשיש חריגה |
| 03 | ‏Staff Shortage Report | כשעובד מדווח על חוסר (Webhook) | מפרק הודעה חופשית כמו "נגמרו עגבניות, שמנת ובצל סגול" לפריטים, עונה לעובד ומעדכן את המנהל |
| 04 | ‏Weekly Food Cost Report | כל יום ראשון ב-08:00 | עלות מזון באחוזים מול יעד, ושלוש קטגוריות הרכש הגדולות |
| 05 | ‏Supplier Order Cutoff Reminder | כל יום ב-09:00 | אילו ספקים צריך להזמין מהם היום ועד איזו שעה, לפי שעת הסגירה |

## עקרונות הבנייה

- **צמתים מוכנים קודם.** בלי צמתי Code. הלוגיקה בנויה מ-Edit Fields, ‏IF, ‏Filter, ‏Split Out, ‏Summarize, ‏Sort ו-Aggregate, עם expressions.
- **בדיקת קלט בתחילת כל workflow.** שדה `validationError` או צומת IF שנקרא כשאלה (`Is Valid?`). קלט לא תקין עוצר ב-Stop and Error עם הודעה ברורה בעברית.
- **טיפול שגיאות משותף.** כל workflow מחובר ל-`00-system-error-alerts`, כך שכל תקלה בהרצה אמיתית מגיעה לטלגרם.
- **אדם מאשר.** שום דבר לא נשלח לספק לבד. ה-workflows מכינים טיוטה, והמנהל מאשר.
- **הודעות בעברית**, קצרות וברורות, עם escape לכל ערך שמגיע מבחוץ.

## התקנה

1. **ייבוא:** ב-n8n יוצרים workflow חדש, ובתפריט ⋯ בוחרים Import from File. מתחילים מ-`workflows/00-system-error-alerts.json`.
2. **Airtable (ל-01a–01c):** בונים את הבסיס לפי [docs/airtable-schema.md](docs/airtable-schema.md), מוסיפים credential של Airtable, ומחליפים בכל צומת Airtable את `YOUR_AIRTABLE_BASE_ID`.
3. **טלגרם:**
   - יוצרים בוט דרך ‎@BotFather ומוסיפים ב-n8n credential מסוג Telegram.
   - בכל צומת Telegram בוחרים את ה-credential, ומחליפים את `YOUR_TELEGRAM_CHAT_ID` ב-Chat ID שלכם.
4. **פרסום טיפול השגיאות:** מפרסמים (Publish) את workflow 00. זה חובה לפני שמחברים אליו workflows אחרים.
5. **חיבור לשגיאות:** מייבאים את 01a–05, ובכל אחד מהם בוחרים ב-Settings ← Error workflow את `_system - Error Alerts`.
6. **בדיקה:** מריצים לפי [docs/testing.md](docs/testing.md).

## מעבר לעבודה אמיתית

| Workflow | מה מחליפים |
| --- | --- |
| 01a–01c | כבר מחוברים ל-Airtable. בשלב הבא: את ה-Webhook של 01b ו-01c בתשובה בוואטסאפ, ואת הצריכה היומית בחישוב מהיסטוריית ההזמנות והקבלות |
| 02 | את המחירון ב-`Read Delivery Note` בטבלת מחירים מוסכמים. את ה-Webhook בחילוץ שורות מצילום התעודה |
| 03 | את ה-Webhook בטריגר של WhatsApp Business Cloud |
| 04 | את `Load Weekly Data` ברכש מטבלת התעודות ובמכירות מהקופה |
| 05 | את `Load Supplier Schedule` בטבלת ספקים עם ימי אספקה ושעת סגירה |

</div>

---

## English summary

Daily restaurant operations workflows for n8n, plus a shared error-alert workflow: supplier-day count lists and order drafts on Airtable (suggested packs = daily usage × days to next delivery + safety stock − on hand, plus manual lines for events and specials), delivery-note price checks against agreed supplier prices, free-text staff shortage reports, a weekly food-cost report and supplier order cutoff reminders.

Built with native n8n nodes only (no Code nodes), input validation at the start of every workflow, a shared Telegram error workflow and human approval before anything reaches a supplier. All data is demo data. See the Hebrew section above for setup and docs/airtable-schema.md for the ordering base.
