📝 استقرار Memos روی Kubernetes — گزارش پروژه

این پست خودش روی همون زیرساختی اجرا می‌شه که توضیح می‌ده: یک نمونه‌ی Memos روی Cluster دو-نودی Kubernetes با k3s، پشت PostgreSQL 17 و Traefik Ingress.

آدرس زنده: sharifi.osdl.ir

معماری
Client → DNS (sharifi.osdl.ir) → Traefik Ingress Controller
→ Service: memos (ClusterIP :5230) → Deployment: memos
→ Service: postgres (Headless :5432) → StatefulSet: postgres

Cluster: k3s، دو Node (sharifi-ha1 کنترل‌پلین، sharifi-ha2 ورکر)
دیتابیس: PostgreSQL 17، StatefulSet تک-Replica با PVC اختصاصی
اپلیکیشن: Memos (neosmemo/memos:stable)، Deployment تک-Replica با PVC برای assetها
Ingress: Traefik (پیش‌فرض k3s)، فقط HTTP روی پورت ۸۰
Storage: local-path (StorageClass پیش‌فرض k3s)

Manifestها و کد کامل پروژه
🔗 [https://github.com/amirarshia-gif/memos-task]

محدودیت‌های شناخته‌شده
local-path یک Storage مشترک بین Nodeها نیست — دیسک واقعی هر PVC روی همون Nodeای ساخته می‌شه که Pod اول بار اونجا Schedule شده و برای همیشه بهش قفل می‌مونه (nodeAffinity). جابه‌جایی یک Pod به Node دیگه یعنی دسترسی به داده‌ی قبلی از دست می‌ره.
هر دو سرویس با یک Replica اجرا می‌شن، بدون Failover خودکار. برای آپدیت بدون Downtime از maxSurge/maxUnavailable استفاده شده، نه Replica اضافه.

یک باگ واقعی که در طول کار پیدا شد
چون Service جلوی Memos اسمش memos بود، Kubernetes خودکار یک متغیر محیطی MEMOS_PORT با فرمت tcp://IP:PORT تزریق می‌کرد که با متغیر خود Memos (که انتظار یک عدد ساده داشت) تصادم پیدا می‌کرد. نتیجه: اپلیکیشن بی‌سروصدا روی پورت ۰ بالا می‌اومد. راه‌حل: enableServiceLinks: false به‌همراه تعریف صریح MEMOS_PORT در env.
