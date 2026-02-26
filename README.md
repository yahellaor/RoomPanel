# מדריך מלא ומפורט: הקמה, ניהול ופתרון תקלות בחדרי ישיבות (Microsoft 365)

מדריך זה מספק מענה מקצה לקצה להקמת חדר ישיבות (Room Resource), שיוכו לרשימות חדרים ופתרון בעיית תצוגת שמות הפגישות בפאנלים פיזיים.

---

## 1. דרישות רישוי ומידע מקדים
* **רישוי מכשיר:** כל מכשיר פיזי (כמו פאנל של Yealink, Neat או Logitech) דורש רישיון מסוג **SHARED** (Microsoft Teams Shared Devices). 
* **רישוי משתמש:** אין צורך ברישיון O365 רגיל (כמו E3 או E5) עבור תיבת הדואר של החדר עצמה.
* **מקור טכני:** המדריך מתבסס על המודול הרשמי [ExchangeOnlineManagement 3.9.2](https://www.powershellgallery.com/packages/ExchangeOnlineManagement/3.9.2).

---

## 2. הקמת החדר בפורטל הניהול (GUI)
לפני המעבר ל-PowerShell, יש ליצור את ה"ישות" במערכת:

1.  **יצירת הישות:** היכנסו ל-**Exchange Admin Center** תחת [Resources](https://admin.exchange.microsoft.com/#/resources).
2.  **דיוק בשם:** יש להזין את שם החדר באופן מלא ומדויק. **זהו שם התצוגה (Display Name) שלא ניתן לשינוי בקלות לאחר מכן** ויופיע בלוחות השנה של כלל העובדים.
3.  **ניהול חשבון:** למרות שהחדר נוצר ב-Exchange, ניהול הסיסמה (Reset Password) ושיוך הרישיון מתבצעים ב-[Microsoft 365 Admin Center](https://admin.cloud.microsoft/?#/users).

---

## 3. הכנת סביבת PowerShell (EXO V3)
ב-Windows PowerShell רגיל אין את הפקודות הנדרשות לניהול ענן. עלינו להתקין את המודול **Exchange Online PowerShell V3**.

### שלב א': הרשאת הרצת סקריפטים
פתחו PowerShell כמנהל מערכת (**Run as Administrator**) והריצו את הפקודה הבאה:

Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

*הסבר:* פקודה זו מאפשרת למחשב להריץ סקריפטים שנכתבו מקומית או נחתמו על ידי גורם מהימן.

### שלב ב': התקנת המודול של Exchange Online
אם יש שאלה לגבי NuGet — אשר "Yes". אם תופיע שאלה לגבי "Untrusted Repository" – אשרו ב-**Yes**.

Install-Module -Name ExchangeOnlineManagement

או:

Install-Module ExchangeOnlineManagement

### שלב ג': טעינה והתחברות (Import & Connect)
יש לאשר (Yes) את טעינת המודול במידה ותתבקשו:

Import-Module ExchangeOnlineManagement

עכשיו תוכל להתחבר (ייפתח חלון GUI להזדהות מול חשבון האדמין):

Connect-ExchangeOnline

### שלב ד': בדיקת תקינות ההתקנה
וודאו שהמודול נטען וזמין במערכת:

Get-Module ExchangeOnlineManagement -ListAvailable

---

## 4. ניהול רשימות חדרים (Room Lists)
**מהי Room List?** זוהי קבוצת הפצה מיוחדת המאגדת חדרים לפי מיקום. Outlook ו-Teams מציגים חדרים בחיפוש "עיין בכל החדרים" **רק אם הם משויכים לרשימה כזו.** רשימה זו נוצרת רק דרך PowerShell.

### א. יצירת Room List חדשה
נניח ששם הרשימה הרצוי הוא IL-Rooms:

New-DistributionGroup -Name "IL-Rooms" -RoomList

### ב. שיוך חדר לרשימה
נניח שכתובת החדר היא NewRoom@domain.com ושם הרשימה הוא IL-Rooms:

Add-DistributionGroupMember -Identity "IL-Rooms" -Member "NewRoom@domain.com"

### ג. פקודות בקרה וניהול רשימות
איך לראות אילו Room Lists קיימות בארגון?

Get-DistributionGroup | where {$_.RecipientTypeDetails -eq "RoomList"}

איך לראות אילו חדרים נמצאים בתוך רשימה ספציפית?

Get-DistributionGroupMember -Identity "IL-Rooms"

איך להסיר חדר מרשימה?

Remove-DistributionGroupMember -Identity "IL-Rooms" -Member "RoomName@domain.com"

> **זמן המתנה:** לאחר ביצוע הפקודות, יש להמתין **10–30 דקות** עד שהחדר יופיע בחיפוש ב-Outlook וב-Teams.

---

## 5. פתרון בעיית "שם המארגן" (אי-הופעת שם האירוע)
זוהי תופעה מוכרת שבה הפאנל (כמו Yealink) מציג את שם העובד שהזמין את החדר במקום את נושא הישיבה. זה נובע מהגדרת הפרטיות ברירת המחדל (DeleteSubject = True).

### שלב 1: בדיקת ההגדרות הנוכחיות של החדר
החליפו את המייל במייל של החדר שלכם:

Get-CalendarProcessing -Identity room@domain.com | fl DeleteSubject,AddOrganizerToSubject,RemovePrivateProperty,OrganizerInfo,DeleteComments

### שלב 2: תיקון ההגדרות (הפקודה המנצחת)
הריצו את הפקודה הבאה כדי שהפאנל יציג את שם הפגישה האמיתי:

Set-CalendarProcessing -Identity room@domain.com -DeleteSubject $False -AddOrganizerToSubject $False -RemovePrivateProperty $False -OrganizerInfo $False -DeleteComments $False

### שלב 3: וידוא הצלחה
הערכים DeleteSubject ו-AddOrganizerToSubject חייבים להופיע כעת כ-**False**:

Get-CalendarProcessing room@domain.com | fl DeleteSubject,AddOrganizerToSubject

---

## 6. סיכום Troubleshooting מתקדם
אם השינוי בוצע אך הפאנל עדיין מציג שם מארגן:

1.  **בדיקת רטרואקטיביות:** פגישות שנוצרו **לפני** הפקודה לא ישתנו. מחקו פגישה קיימת וצרו אחת חדשה לבדיקה.
2.  **ניקוי Cache במכשיר:** בצעו הפעלה מחדש (Restart) למכשיר הפיזי או בצעו Sign-out ו-Sign-in מחדש לחשבון החדר במכשיר.
3.  **הגדרות ב-Teams Admin Center:**
    * נווטו ל-Teams Devices -> Panels -> Settings.
    * וודאו שהאופציות **Hide meeting titles** או **Show organizer only** כבויות (Off).
