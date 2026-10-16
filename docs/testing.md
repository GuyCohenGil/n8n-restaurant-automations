<div dir="rtl">

# בדיקות

כל workflow נבדק פעמיים: פעם עם קלט תקין, ופעם עם קלט שבור בכוונה. ה-workflow של טיפול השגיאות פועל רק בהרצות אמיתיות (Production), ולא בהרצה ידנית מהעורך. לכן את ה-webhooks בודקים אחרי Publish.

## איך בודקים

**workflows מתוזמנים (01a, 04, 05):**
- **קלט תקין:** Execute workflow בעורך.
- **קלט שבור:** מצמידים (Pin) לצומת ה-Load נתונים שבורים מהטבלה למטה ומריצים. התוצאה הצפויה היא עצירה ב-Stop and Error עם ההודעה שבטבלה.

**webhooks (01b, 01c, 02, 03):** מפרסמים, ושולחים POST לכתובת ה-Production עם הקבצים שבתיקייה `samples`:

</div>

```bash
curl -X POST https://YOUR-N8N/webhook/demo-r2-price-check \
  -H "Content-Type: application/json" \
  -d @samples/r2-valid.json

# 01b, 01c
curl -X POST https://YOUR-N8N/webhook/demo-r1-count \
  -H "Content-Type: application/json" \
  -d @samples/r1b-count-valid.json

curl -X POST https://YOUR-N8N/webhook/demo-r1-manual \
  -H "Content-Type: application/json" \
  -d @samples/r1c-manual-valid.json
```

<div dir="rtl">

## תוצאות (14–16/10/2026)

| Workflow | קלט | תוצאה צפויה | תוצאה |
| --- | --- | --- | --- |
| 00 | שגיאה מדומה עם `<Unauthorized>` | התראה בטלגרם, התווים `<` `>` מוצגים נכון | עבר |
| 01a | יום חמישי, בסיס ההדגמה | 3 רשימות ספירה (ירקות, עופות, חלב), ממוינות לפי שעת סגירה ומקום אחסון | עבר |
| 01b | `r1b-count-valid.json` | טיוטה לירקות השרון: 3 × ארגז עגבניות, 1 × שק בצל, 1 × קרטון פטרוזיליה, 1 × ארגז לימונים. מלפפונים: "לא צריך". נשמרו Counts ו-Order Lines | עבר |
| 01b | `r1b-count-invalid.json` | Stop and Error: שורות בלי כמות | עבר |
| 01c | `r1c-manual-valid.json` אחרי 01b (יש טיוטה להיום) | שורה ידנית (אירוע) נוספה לטיוטה הקיימת | עבר |
| 01c | שורה ידנית לספק בלי טיוטה היום | נפתחה טיוטה חדשה מחוץ ליום ההזמנה, והשורה נוספה אליה | עבר |
| 01c | `r1c-manual-invalid.json` | Stop and Error: כמות חייבת להיות גדולה מ-0, סיבה לא חוקית | עבר |
| 02 | `r2-valid.json` | 2 חריגות, תוספת של 13.4 ₪, התראה למנהל | עבר |
| 02 | `r2-no-deviation.json` | בלי חריגות, בלי התראה, ענף Prices OK | עבר |
| 02 | `r2-invalid.json` | Stop and Error: ספק לא מוכר, שורה בלי מחיר, ואז התראת תקלה | עבר |
| 03 | `r3-valid.json` | 3 פריטים, תשובה לעובד והודעה למנהל | עבר |
| 03 | `r3-invalid.json` | Stop and Error: הודעה ריקה, ואז התראת תקלה | עבר |
| 04 | נתוני השבוע | עלות מזון 30.6% מול יעד 30% | עבר |
| 04 | מכירות 0 ובלי רכש | Stop and Error: אין נתוני מכירות | עבר |
| 05 | לוח ספקים, יום רביעי | שני ספקים, ממוינים לפי שעת סגירה | עבר |
| 05 | שורה עם `days: "שלישי"` ו-`cutoff: "12"` | Stop and Error: שורת לוח לא תקינה | עבר |

## באגים שנמצאו ותוקנו

**01b:** צומת Airtable מחזיר כל רשומה בתוך `fields` (`{id, createdTime, fields}`). בהרצה הראשונה ה-expressions קראו `$json.Item` במקום `$json.fields.Item`, אז אף פריט לא הותאם. בדיקת הקלט עצרה את זה ב-Stop and Error, בלי טיוטה ריקה. אחרי התיקון כל השדות נקראים מ-`$json.fields`.

**02:** ב-02, כש-Split Out מוגדר לכלול שדות נוספים, הוא שומר כל שורה מתחת לשם השדה המקורי במקום לפרוס אותה. הפתרון: `destinationFieldName: noteLine`, וקריאה ל-`$json.noteLine.item`. לפני התיקון כל הפריטים הוצגו כ-undefined וסומנו בטעות כחריגה.

</div>
