<p align="left">
  <a href="README.md">🇷🇺 Русский</a> &middot;
  <a href="README_EN.md">🇬🇧 English</a>
</p>

# תוסף אכיפת שבת ל-MandaGram‏ (C ABI) גרסה 2.0.0

תוסף נייטיב בשפת **C** עבור לקוח **MandaGram**‏ ([mandagram.com](https://mandagram.com)) המבטיח שמירת שבת: נעילת ממשק המשתמש באמצעות שכבת-על (Overlay) אלגנטית בזמני השבת הקדושים וחסימת שליחת הודעות מבלי לקרוס ומבלי לסגור את היישום.

פותח בהתאם למפרט **MandaGram C ABI SDK** (גשר מבוסס Java Foreign Function & Memory API).

---

## 🕊️ תכונות עיקריות

* **שכבת-על לממשק (Overlay Mode)**: עם כניסת השבת, מופעלת שכבת-על מלאה מעל חלון MandaGram עם הכיתוב *«🕯️ שבת שלום! 🕯️»*, המונעת גישה לצ'אטים ולתפריטים תוך השארת היישום פעיל ברקע.
* **יירוט שליחת הודעות (TDLib / FFI Hook)**: התוסף רושם הוק מסוג `register_message_send_hook`, החוסם שליחת הודעות יוצאות ברמת הרשת במהלך השבת.
* **בקרת זמנים ברקע (Background Watchdog)**: תהליכון רקע קל-משקל בודק את זמן המערכת המקומי ומציג או מסתיר את שכבת-העל באופן רציף עם כניסת השבת ויציאתה.
* **תמיכה חוצת-פלטפורמות (Cross-Platform)**:
  * **Windows**: שכבת-על טבעית מבוססת Win32 GDI עם תמיכה מלאה ב-Unicode (גופני Segoe UI, אימוג'י דרך `DrawTextW`), לולאת הודעות מהירה (`MsgWaitForMultipleObjects`), ומניעת דליפות משאבי GDI.
  * **Linux**: שכבת-על טבעית מבוססת X11 עם תמיכה בגופני UTF-8 מדרגיים (`Xutf8DrawString`), הממוקמת ישירות מעל חלון MandaGram ללא הפרעה לשאר סביבת שולחן העבודה.
* **מצב ניפוי שגיאות מהיר (Debug Mode)**: אפשרות להפעלת סימולציית שבת קבועה לבדיקות ופיתוח באמצעות הגדרת קדם-מעבד פשוטה.

---

## ⏰ לוח זמני שבת כברירת מחדל

| יום בשבוע | שעה | סטטוס |
| :--- | :--- | :--- |
| **יום שישי** | החל מ-**18:00** | כניסת שבת (הממשק ננעל, שליחת הודעות נחסמת) |
| **יום שבת** | לאורך כל היום עד **21:30** | שבת קודש (הלקוח נשאר נעול) |
| **יום שבת** | לאחר **21:30** | צאת השבת (הגישה מתחדשת אוטומטית) |
| **ראשון – חמישי** | 24/7 | פעילות רגילה |

ניתן לשנות את הגדרות הזמנים ישירות בתחילת הקובץ [`src/shabbat_plugin.c`](src/shabbat_plugin.c):
```c
#define SHABBAT_START_WDAY 5   /* יום שישי */
#define SHABBAT_START_HOUR 18  /* 18:00 */
#define SHABBAT_START_MIN  0

#define SHABBAT_END_WDAY   6   /* יום שבת */
#define SHABBAT_END_HOUR   21  /* 21:30 */
#define SHABBAT_END_MIN    30
```

---

## 🧪 בדיקות ומצב Debug

לבדיקת פעולת התוסף באופן מיידי (בכל יום ובכל שעה):

1. פתחו את הקובץ [`src/shabbat_plugin.c`](src/shabbat_plugin.c).
2. בטלו את ההערה (Uncomment) בשורה הבאה:
   ```c
   #define FORCE_SHABBAT_DEBUG 1
   ```
3. הדרו מחדש את התוסף (`make`). בהפעלת MandaGram, הממשק יינעל מיד ויוצג המסך: *«🕯️ שבת שלום! 🕯️ הממשק נעול עד מוצאי שבת ב-21:30»*.

---

## 📁 מבנה המאגר

```text
mandagram-shabbat-plugin/
├── Makefile                # פקודות הידור עבור Linux (.so) ו-Windows (.dll)
├── README.md               # תיעוד ברוסית (ברירת מחדל)
├── README_EN.md            # תיעוד באנגלית
├── README_HE.md            # תיעוד בעברית
├── .gitignore              # התעלמות מקובצי הידור בינאריים
├── include/
│   └── mandagram_plugin.h  # הגדרות כותרות C ABI של MandaGram
├── src/
│   └── shabbat_plugin.c    # מימוש התוסף בשפת C
└── out/
    ├── shabbat_plugin.so   # ספרייה משותפת מהודרת ל-Linux x86_64
    └── shabbat_plugin.dll  # קובץ DLL מהודר ל-Windows x64
```

---

## 🔨 הידור (Build)

### דרישות מוקדמות
* `gcc` (להידור ב-Linux).
* `mingw-w64` (`x86_64-w64-mingw32-gcc`, להידור צולב עבור Windows).

### פקודות

הידור ספרייה משותפת עבור Linux‏ (`out/shabbat_plugin.so`):
```bash
make linux
```

הידור ספרייה דינמית עבור Windows‏ (`out/shabbat_plugin.dll`):
```bash
make windows
```

הידור שתי הפלטפורמות בפקודה אחת:
```bash
make both
```

ניקוי תוצרי הידור:
```bash
make clean
```

---

## 📦 התקנה ב-MandaGram

1. הדרו את הקובץ הבינארי המתאים למערכת ההפעלה שלכם (`.so` ללינוקס או `.dll` לווינדוס).
2. העתיקו את הקובץ המהודר אל תיקיית התוספים של MandaGram:
   * **Linux**: בנתיב `~/.local/share/mandagram/plugins/` (או בתיקיית `plugins/` לצד קובץ ההפעלה של MandaGram)
   * **Windows**: בנתיב `%APPDATA%\MandaGram\plugins\` (או בתיקיית `plugins/` לצד `MandaGram.exe`)
3. הפעילו את MandaGram. בלוג הלקוח תופיע הודעת האישור:
   ```text
   [Shabbat] Shabbat Enforcer v2.0 (Overlay Mode) успешно активирован.
   ```

---

## 📜 רישיון

רישיון ציבורי כללי של גנו גרסה 3.0 (GPL-3.0). שבת שלום ומבורך!
