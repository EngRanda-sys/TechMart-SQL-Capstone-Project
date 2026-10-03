# TechMart-SQL-Capstone-Project
SQL data cleaning &amp; analysis capstone project using SQLite and PandasTechMart SQL Capstone Project

تحليل شامل لبيانات متجر TechMart باستخدام SQL (SQLite) وPython (Pandas)، يغطي تنظيف البيانات، التحليل المتقدم، وتحسين أداء الاستعلامات — كمشروع ختامي (Capstone) ضمن دورة تحليل البيانات على منصة Coursera.

📋 نظرة عامة

يحتوي المشروع على 4 جداول مترابطة:

Employee_Records — بيانات الموظفين وأدائهم البيعي
Product_Details — بيانات المنتجات وأسعارها
Customer_Demographics — بيانات الزبائن وبرنامج الولاء
Sales_Transactions — سجل المعاملات البيعية

تم تنفيذ المشروع عبر 4 مراحل: تنظيف البيانات، التحليل المتقدم (CTEs + Window Functions)، تحسين الأداء (Indexing)، وتقرير تحليلي نهائي.

🧹 تنظيف البيانات (Data Cleaning)

تم التعامل مع كل عمود حسب طبيعته، باستخدام أكثر من أسلوب تنظيف لإظهار تنوع الأدوات:

Product_Details → price المشكلة: قيم مفقودة (NULL) القرار: تُركت NULL — لا يوجد بديل موثوق للتعويض
Customer_Demographics → age المشكلة: نصوص مثل "thirty-five" + NULL القرار: تحويل النصوص إلى NULL عبر GLOB، ثم تعويض بالـ Mean
Customer_Demographics → loyalty_program المشكلة: نص 'nan' + NULL القرار: تُركت NULL — القيم (Yes/No) متقاربة جداً (33/32)، لا أساس إحصائي للتعويض
Sales_Transactions → quantity المشكلة: نص "three" + NULL القرار: تحويل النص إلى رقم، وحذف صفوف NULL (17 صف)
Sales_Transactions → employee_id المشكلة: NULL القرار: حذف الصفوف (5 صفوف)
Sales_Transactions → total_amount المشكلة: NULL القرار: حُذفت (11 صف) بعد اكتشاف أن حساب price × quantity غير موثوق

⚠️ اكتشاف مهم: تبيّن أن عمود product_id في جدول Product_Details ليس معرّفاً فريداً — نفس الرقم يمثل عدة منتجات مختلفة تماماً (بأسعار وفئات مختلفة)، مما يجعل أي JOIN عليه غير موثوق بالكامل. هذا الاكتشاف أثّر على قرارات التحليل اللاحقة.

نتيجة التنظيف: انخفض عدد صفوف Sales_Transactions من ~100 إلى 78 صفاً (حذف ما يقارب الثلث)، وهو أمر تم توثيقه ومناقشته كقيد على التحليل.

📊 التحليل المتقدم (Advanced Analysis)

تم استخدام CTEs (Common Table Expressions) وWindow Functions (RANK, PARTITION BY) لتحليل:

أداء الموظفين حسب موقع المتجر
أكثر المنتجات مبيعاً ضمن كل فئة
سلوك الزبائن الشرائي ومقارنته بحالة الاشتراك في برنامج الولاء
أبرز استنتاج (Key Insight)
sql
WITH customer_behavior AS (
    SELECT c.customer_id, c.loyalty_program,
           COUNT(*) AS purchase_count,
           SUM(s.total_amount) AS total_spent
    FROM Customer_Demographics c
    JOIN Sales_Transactions s ON c.customer_id = s.customer_id
    GROUP BY c.customer_id, c.loyalty_program
)
SELECT loyalty_program, COUNT(*) AS num_customers,
       AVG(purchase_count) AS avg_purchases,
       AVG(total_spent) AS avg_spent
FROM customer_behavior
GROUP BY loyalty_program

كشف هذا الاستعلام أن الزبائن غير المشتركين في برنامج الولاء سجلوا متوسط شراء وإنفاق أعلى قليلاً من المشتركين — نتيجة مخالفة للتوقع الشائع، مع التنويه بأن حجم العينة صغير (4 زبائن لكل فئة).

التوصية: مراجعة فعالية برنامج الولاء الحالي، وتحسين جمع بيانات الاشتراك لإجراء تحليل أدق على عينة أكبر.

⚡ تحسين الأداء (Performance Optimization)
إنشاء Indexes على أعمدة الربط (employee_id, product_id, customer_id) في جميع الجداول المعنية لتسريع عمليات JOIN.
إعادة هيكلة استعلام معقد (4 جداول + CTE مزدوج) إلى نسخة أبسط باستخدام Window Function (SUM() OVER (PARTITION BY ...)) بدلاً من CTE ثانٍ منفصل، لتقليل عدد مرات مسح البيانات.
🛠️ التقنيات المستخدمة
SQL (SQLite) — الاستعلامات، CTEs، Window Functions، Indexing
Python — Pandas, sqlite3
Jupyter Notebook — بيئة التنفيذ والتوثيق
📁 محتويات المستودع
Capstone_Project.ipynb — الكود الكامل للمشروع (تنظيف، تحليل، تحسين)
TechMart_Data_Cleaning_Summary.xlsx — جدول ملخص بقرارات التنظيف
Final_Report — التقرير التحليلي النهائي (أهم 3 استنتاجات + توصيات)
✍️ ملاحظة

هذا المشروع جزء من مسار تعلم تحليل البيانات، ويُظهر تطبيقاً عملياً لمهارات SQL الأساسية والمتقدمة على بيانات واقعية غير نظيفة — بما في ذلك اتخاذ قرارات تحليلية مبررة عند مواجهة مشاكل جودة بيانات حقيقية.
