## חלק א — מפת ה-cluster

| רכיב | איפה הוא רץ (הנהלה / כל node) | מה הוא עושה, במשפט אחד | אנלוגיה |
|---|---|---|---|
| API server |רץ בהנהלה | הוא מנהל את כל הבקשות היוצאות והנכנסות|מזכירה בבית ספר |
| etcd |רץ בהנהלה |שומר את כל המידע | יומני נוכחות בבית ספר|
| scheduler |רץ בהנהלה | מקבל podim חדשים ומחליט באיזה node למקם אותם| אחראית מערכת|
| controller manager |רץ בהנהלה |משווה בין המציאות לנדרש | ממלאת מקום|
| kubelet | רץ בnode|מקבל הוראה | מורה בכיתה|
| container runtime |רץ ב node |דואג שהכל יתבצע |אב בית |
| kube-proxy |רץ על כל Node |מתווך בין בקשות כך שבאמת יגיעו למקומן |שומר |
| coredns | רץ בcluster| מתרגם שם לכתובת| אלפון|
## חלק ב

### תרחיש 1: scale
1. הפקודה `kubectl scale` מגיעה ל-API server.
2. המספר 4 נשמר ב-etcd.
3. ה-controller manager רואה שהוגדרו 4 pods ורק 2 רצים, ויוצר 2 pods חדשים.
4. ה-scheduler מחליט על איזה node ירוץ כל pod חדש.
5. ה-kubelet על ה-node מקבל את ההוראה, וה-container runtime מפעיל את ה-containers, עד שכל 4 ה-pods במצב Running.
### תרחיש 2: שלוש בלילה

**container שקרס:**
1. ה-kubelet על ה-node מזהה שה-container קרס, ומפעיל אותו מחדש באותו pod (בעזרת ה-container runtime).

**אילו ה-pod כולו היה נמחק:**
2. ה-controller manager מזהה שחסר pod ויוצר pod חדש במקומו.
3. ה-scheduler מחליט על איזה node ה-pod החדש ירוץ.
4. ה-kubelet על אותו node מקבל את ההוראה, וה-container runtime מפעיל את ה-container.
### תרחיש 3
- controller manager יוצר pods חדשים, ו-scheduler משבץ אותם על nodes 1 ו-3
- ב-Docker Desktop:הכל נופל

### תרחיש 4
- ה- images לא יעבדו כי כתוב NEVER. כלומר, במחשב שלי הוא לא מוריד מהאינטרנט כי קיים כבר על המחשב אך בענן יצטרכו למחוק את NEVER כדי שהוא יוריד מ dockerhub. הוא יוריד את מה שאני העליתי עם השם משתמש שלי. יש לשנות image:<שם משתמש>/greeter:v1
- localhost פונה למחשב שלי ולכן יש להגדיר type:LoadBalancer
וכך הענן נותן כתובת ציבורית.
### תרחיש 5: pod בודד
- ה-pod של kubectl run לא חזר, כי לא היה Deployment שהגדיר כמה pods צריכים לרוץ, ולכן ל-controller manager לא היה פער לזהות.
ה-pod של web חזר, כי ה-Deployment שמר את ההגדרה replicas, וה-controller manager יצר pod חדש כשחסר אחד.
כדי ש-stats יחזור תמיד, הייתי יוצרת לו Deployment עם replicas: 1.