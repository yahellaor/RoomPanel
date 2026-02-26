# 📋 Microsoft Exchange Online — ניהול חדרי ישיבות (Room Resources)
> מדריך מקיף להקמה, ניהול ופתרון תקלות — כולל PowerShell ופורטל הניהול

---

## תוכן עניינים

1. [דרישות רישוי ומידע מקדים](#1-דרישות-רישוי-ומידע-מקדים)
2. [הקמת החדר בפורטל הניהול (GUI)](#2-הקמת-החדר-בפורטל-הניהול-gui)
3. [הכנת סביבת PowerShell (EXO V3)](#3-הכנת-סביבת-powershell-exo-v3)
4. [ניהול רשימות חדרים (Room Lists)](#4-ניהול-רשימות-חדרים-room-lists)
5. [פתרון בעיית "שם המארגן" — אי-הופעת שם האירוע](#5-פתרון-בעיית-שם-המארגן--אי-הופעת-שם-האירוע)

---

## 1. דרישות רישוי ומידע מקדים

- כל **מכשיר פיזי** דורש **רישיון SHARED**.
- **לא נדרש** רישוי O365 נוסף.

> 📦 מקור מודול PowerShell:  
> [https://www.powershellgallery.com/packages/ExchangeOnlineManagement/3.9.2](https://www.powershellgallery.com/packages/ExchangeOnlineManagement/3.9.2)

---

## 2. הקמת החדר בפורטל הניהול (GUI)

### יצירת Room Resource

1. היכנס לפורטל ניהול Exchange:  
   👉 [https://admin.exchange.microsoft.com/#/resources](https://admin.exchange.microsoft.com/#/resources)

2. צור **"חדר" / Room Resource** — כתובת מייל תחת הדומיין שלך.

> ⚠️ **חשוב מאוד:** יש לדייק בשם החדר באופן מלא.  
> זהו **שם תצוגה שלא ניתן לשינוי** לאחר היצירה.

### ניהול המשתמש של החדר

ניתן לנהל את החדר (כתובת, סיסמה, רישיון) דרך:  
👉 [https://admin.cloud.microsoft/?#/users](https://admin.cloud.microsoft/?#/users)

פעולות אפשריות דרך פורטל זה:
- יצירת סיסמה / שינוי סיסמה
- שיוך רישיון למשתמש

---

## 3. הכנת סביבת PowerShell (EXO V3)

> ⚠️ ב-Windows PowerShell רגיל **אין** את הפקודה `Connect-ExchangeOnline`.  
> היא מגיעה **רק** אחרי התקנת המודול: **Exchange Online PowerShell V3 (EXO V3 Module)**.

---

### שלב א' — הרשאת הרצת סקריפטים

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

---

### שלב ב' — התקנת המודול של Exchange Online

```powershell
Install-Module -Name ExchangeOnlineManagement
```

או בגרסה הקצרה:

```powershell
Install-Module ExchangeOnlineManagement
```

> 💡 אם מופיעה שאלה לגבי **NuGet** — יש לאשר **"Yes"**.

---

### שלב ג' — טעינה והתחברות (Import & Connect)

**טעינת המודול:**
```powershell
Import-Module ExchangeOnlineManagement
```
> ענה **YES** לכל ההנחיות.

**התחברות ל-Exchange Online:**
```powershell
Connect-ExchangeOnline
```

> 🖥️ חלון GUI יקפוץ — יש להתחבר לחשבון **האדמין של Microsoft** שינהל את החדרים.

---

### שלב ד' — בדיקת תקינות ההתקנה

```powershell
Get-Module ExchangeOnlineManagement -ListAvailable
```

---

## 4. ניהול רשימות חדרים (Room Lists)

### מהי Room List?

**Room List** היא קבוצה שמכילה חדרי ישיבות לפי מיקום (בניין / קומה).  
Outlook ו-Teams **מציגים רק חדרים שנמצאים בתוך Room List** תחת:
- "עיין בכל החדרים"
- רשימת החדרים המוצעים

> ⚠️ חדר חדש שלא שויך לרשימה — **לא יופיע** בחיפוש ב-Outlook/Teams.

> 🔒 **Room List נוצרת רק דרך PowerShell** — לא דרך ה-Admin Center.

---

### א. יצירת Room List חדשה

לאחר התחברות ל-Exchange Online PowerShell:

```powershell
New-DistributionGroup -Name "IL-Rooms" -RoomList
```

פקודה זו יוצרת רשימת חדרים בשם **IL-Rooms**.

---

### ב. הוספת חדר לרשימת חדרים

נניח שהחדר נקרא: `NewRoom@domain.com`  
ושם הרשימה: `IL-Rooms`

```powershell
Add-DistributionGroupMember -Identity "IL-Rooms" -Member "NewRoom@domain.com"
```

> ⏱️ **המתנה של עד 30 דקות:** אחרי יצירת חדר חדש, ל-Outlook ול-Teams לוקח זמן עד שהוא מופיע בחיפוש.  
> תוך **10–30 דקות** החדר יופיע ב-Outlook וב-Teams.

---

### ג. פקודות בקרה וניהול רשימות

**לראות אילו Room Lists קיימות:**
```powershell
Get-DistributionGroup | where {$_.RecipientTypeDetails -eq "RoomList"}
```

**לראות אילו חדרים נמצאים בתוך רשימה:**
```powershell
Get-DistributionGroupMember -Identity "IL-Rooms"
```

**להסיר חדר מרשימה:**
```powershell
Remove-DistributionGroupMember -Identity "IL-Rooms" -Member "RoomName@domain.com"
```

---

## 5. פתרון בעיית "שם המארגן" — אי-הופעת שם האירוע

### רקע — למה זה קורה?

התצוגה שבה **Room Panel מציג את שם המארגן במקום שם הפגישה** היא תופעה מוכרת ב-Teams Rooms / Yealink Panels.  
היא כמעט תמיד נובעת מהגדרות ברירת מחדל בחשבון החדר (Resource Mailbox).

> ✅ **זה לא באג — זו הגדרה, וניתן לתקן.**

**ברירת המחדל של Exchange Online** היא שחדרי ישיבות **מסתירים את נושא הפגישה** ומציגים רק את שם המארגן — **מטעמי פרטיות**.

ההגדרות הרלוונטיות:

| הגדרה | ברירת מחדל | השפעה |
|---|---|---|
| `DeleteSubject` | `True` | מוחק את שם הפגישה |
| `AddOrganizerToSubject` | `True` | מחליף בשם המארגן |
| `RemovePrivateProperty` | `True` | מסיר מאפייני פרטיות |

כאשר `DeleteSubject = True` → החדר מוחק את שם הפגישה ומחליף אותו בשם המארגן → הפאנל מציג את שם המארגן בלבד.

---

### שלב 1 — בדיקת ההגדרות הנוכחיות של החדר

לאחר התחברות ל-Exchange Online PowerShell, הרץ (החלף `room@domain.com` בכתובת החדר שלך):

```powershell
Get-CalendarProcessing -Identity room@domain.com | fl DeleteSubject,AddOrganizerToSubject,RemovePrivateProperty
```
❗שים לב לשינוי לנתוניך האישיים


**אם החדר מציג את שם המארגן — סביר שתראה:**
```
DeleteSubject             : True
AddOrganizerToSubject     : True
RemovePrivateProperty     : True
```

---

### שלב 2 — תיקון ההגדרות

```powershell
Set-CalendarProcessing -Identity room@domain.com -DeleteSubject $False -AddOrganizerToSubject $False -RemovePrivateProperty $False
```
❗שים לב לשינוי לנתוניך האישיים

> ⏱️ המתן **5–10 דקות** — ה-Room Panel יתעדכן אוטומטית.

---

### שלב 3 — וידוא הצלחה

```powershell
Get-CalendarProcessing room@domain.com | fl DeleteSubject,AddOrganizerToSubject
```

**התוצאה הצפויה:**
```
DeleteSubject             : False
AddOrganizerToSubject     : False
```

**מה יקרה אחרי התיקון?**
- ✅ הפאנל יציג את **שם הפגישה האמיתי**
- ✅ לא יוצג שם המארגן במקום הנושא
- ✅ Outlook / Teams יראו את שם הפגישה גם בלוח השנה של החדר

---

### 🔬 פתרון מתקדם — אם הבעיה נמשכת לאחר השינוי

אם הגדרות ה-Calendar Processing עודכנו אבל ה-Room Panel **עדיין** מציג את שם המארגן, יש לעבור על הצעדים הבאים:

---

#### 🔍 שלב 1 — לוודא שההגדרות באמת עודכנו

הרץ שוב:

```powershell
Get-CalendarProcessing -Identity NewRoom@domain.com | fl DeleteSubject,AddOrganizerToSubject,RemovePrivateProperty
```
❗שים לב לשינוי לנתוניך האישיים


**התוצאה חייבת להיות:**
```
DeleteSubject             : False
AddOrganizerToSubject     : False
RemovePrivateProperty     : False
```

> אם אחד מהם עדיין `True` → השינוי **לא הוחל**.

---

#### 🔍 שלב 2 — לוודא שאין Override נוסף

יש עוד שתי הגדרות שפחות מוכרות, אבל אם הן מופעלות — הן **ממשיכות להסתיר** את שם הפגישה:

- `OrganizerInfo`
- `DeleteComments`

**בדוק גם אותן:**

```powershell
Get-CalendarProcessing -Identity NewRoom@domain.com | fl OrganizerInfo,DeleteComments
```
❗שים לב לשינוי לנתוניך האישיים

| הגדרה | אם `True` — ההשפעה |
|---|---|
| `OrganizerInfo` | החדר עדיין מציג את שם המארגן |
| `DeleteComments` | לפעמים משפיע על תצוגת הנושא |

**כדי לכבות:**

```powershell
Set-CalendarProcessing -Identity NewRoom@domain.com -OrganizerInfo $False -DeleteComments $False
```
❗שים לב לשינוי לנתוניך האישיים

---

#### 🔍 שלב 3 — האם הפגישה נוצרה לפני שינוי ההגדרות?

> ⚠️ **זה קריטי!**  
> פגישות שנוצרו **לפני** שינוי ההגדרות **לא מתעדכנות אוטומטית**.  
> ה-Room Panel ימשיך להציג את שם המארגן עבורן.

**כדי לבדוק אם זה המקרה:**
1. מחק פגישה קיימת
2. צור פגישה **חדשה לגמרי**
3. בדוק אם הפאנל מציג את שם הפגישה החדש

> אם הפגישה החדשה מוצגת נכון → הבעיה הייתה בפגישות ישנות בלבד ✅

---

#### 🔍 שלב 4 — לוודא שה-Room Panel עצמו לא שומר Cache

פאנלים של Yealink שומרים **Cache** של לוח השנה.

בצע:
1. **הפעלה מחדש** של הפאנל
2. **ניתוק וחיבור מחדש** לחשבון החדר
3. המתנה של **5–10 דקות**

---

#### 🔍 שלב 5 — לוודא שהחדר לא מוגדר כ-"Private Meeting Mode"

יש הגדרה ב-Teams Admin Center שיכולה לגרום לפאנל להציג שם מארגן בלבד:

**נתיב:**  
`Teams Admin Center` → `Devices` → `Panels` → בחר את הפאנל → `Settings`

**חפש את:**
- `Hide meeting titles`
- `Show organizer only`

> ⚙️ **כבה אותם** אם הם מופעלים.  
> נתיב ישיר: `Teams Devices → Panels → Settings → Privacy → Hide meeting titles`

---

## 📎 סיכום פקודות PowerShell מהיר

```powershell
# התחברות
Connect-ExchangeOnline

# יצירת Room List
New-DistributionGroup -Name "IL-Rooms" -RoomList

# הוספת חדר לרשימה
Add-DistributionGroupMember -Identity "IL-Rooms" -Member "room@domain.com"

# הצגת כל ה-Room Lists
Get-DistributionGroup | where {$_.RecipientTypeDetails -eq "RoomList"}

# הצגת חדרים ברשימה
Get-DistributionGroupMember -Identity "IL-Rooms"

# הסרת חדר מרשימה
Remove-DistributionGroupMember -Identity "IL-Rooms" -Member "room@domain.com"

# בדיקת Calendar Processing
Get-CalendarProcessing -Identity room@domain.com | fl DeleteSubject,AddOrganizerToSubject,RemovePrivateProperty,OrganizerInfo,DeleteComments

# תיקון תצוגת שם פגישה
Set-CalendarProcessing -Identity room@domain.com -DeleteSubject $False -AddOrganizerToSubject $False -RemovePrivateProperty $False -OrganizerInfo $False -DeleteComments $False
```

---

> 📝 **הערה:** לאחר כל שינוי ב-Calendar Processing יש להמתין **5–10 דקות** לעדכון.  
> לאחר יצירת חדר חדש או הוספה לרשימה — יש להמתין **10–30 דקות** להופעה ב-Outlook/Teams.
