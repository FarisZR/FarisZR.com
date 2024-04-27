---
title: شركة Oracle ترد على قرار Red hat الاخير بايقاف نشر مصدر RHEL وتسخر منها
categories: 
    - اخبار
date: 2023-07-10
image: thumbnail.jpeg
slug: oracle-attacks-Red-hats-recent-decision
canonicalurl: https://discourse.aosus.org/t/topic/2838
tags:
    - Red hat
    - RHEL
    - RHEL source code
    - RHEL open source
    - رد هات
    - مصدر رد هات
    - مجتمع أسس
    - Oracle Linux
    - أوراكل لينكس
    - لينكس
---

**هذا المقال كُتب على مجتمع أسس [هنا](https://discourse.aosus.org/t/topic/2838)**

أصدرت شركة Oracle اليوم بيانها حول قرار شركة Red hat الأخير بإيقاف نشر مصدر توزيعة RHEL لغير المشتركين
https://discourse.aosus.org/t/topic/2819
يذكر ان توزيعات اخرى مثل Almalinux و Rocky Linux و Cloudlinux أوضحت خططها المستقبلية سابقا
https://discourse.aosus.org/t/topic/2832

شركة Oracle لها تاريخ طويل مع Red hat, من منافسة و تجنب الدفع وغيره.
بالاضافة ان الشركة معروفة بمحبتها لرفع القضايا و الاحتكار, وتاريخ حافل مع البرمجيات المفتوحة مثل مشروع [Openoffice](https://en.wikipedia.org/wiki/OpenOffice.org#History) و [MySQL](https://en.wikipedia.org/wiki/MySQL#Legal_disputes_and_acquisitions) و إيقاف [OpenSolaris](https://en.wikipedia.org/wiki/OpenSolaris)
لذلك كانت مفاجئة ان Oracle تركز على أهمية إبقاء لينكس متوفر للجميع ومجاني في بيانها الأخير:
https://www.oracle.com/news/announcement/blog/keep-linux-open-and-free-2023-07-10/

## سبب اختيار RHEL كأساس ل Oracle Linux وإبقاء التوافقية
يبدأ البيان بإيضاح الشركة لمساهمتها في لينكس خلال 25 سنة, وأنها تقوم بالمساهمة في النواة و انظمة الملفات و الأدوات, و انهم يقومون برفع هذه التعديلات الى المصدر ليستفيد منها كل المجتمع.

ويذكرون أن سبب اختيار RHEL كأساس لOracle Linux هو لتجنب حدوث تفرع في مجتمع لينكس, وان هدفهم بإبقاء التوافقية مع RHEL كان ناجح, لدرجة ان برمجياتهم يتم اختبارها فقط على Oracle Linux, لكنهم كان يعتبرونها متوافقة مع RHEL, و أن طوال فترة وجود التوزيعة تقريبا لم يوجد اي بلاغ عن مشكلة توافقية بين Oracle Linux و RHEL.

## مهاجمة IBM

هاجمت الشركة في التصريح IBM مباشرة, بدلا من التركيز على Red Hat, التي أصبحت فرع منها في عام 2019.
وتقول ان Oracle لديها فكر مختلف تماما عن التوافقية مع ترخيص [GPLv2](https://ar.wikipedia.org/wiki/%D8%B1%D8%AE%D8%B5%D8%A9_%D8%AC%D9%86%D9%88_%D8%A7%D9%84%D8%B9%D9%85%D9%88%D9%85%D9%8A%D8%A9), وانهم سيبقون مصدر Oracle Linux متوفر للجميع, دون اي شروط تمنع اعادة نشر المصدر للمشتركين.

وردت Oracle على سبب Red hat المعطى  في مقال https://www.redhat.com/en/blog/red-hats-commitment-open-source-response-gitcentosorg-changes ان Red hat تحتاج ان تدفع لألاف الموظفين المسؤولين عن دعم وتحديث وإصلاح العلل في RHEL,  بأن شركة Red hat لم تمانع نشر مصدر RHEL للعامة للعديد من السنين, وهذا تغير فقط عندما اشترت IBM شركة Red hat ب 34 مليار دولار في 2019 و ان السبب الحقيقي هو قتل المنافسة.
IBM أوفقت CentOS في 2020, والآن تحاول ايقاف AlmaLinux و RockyLinux لزيادة أعداد المشتركين المدفوعين في RHEL.

وتسخر من IBM, إذ قالت Oracle اذا IBM غير جاهزه للدفع لكل موظفين RHEL, بإمكانهم جعل RHEL نسخه من Oracle Linux, و ان Oracle سوف تأخذ عاتق تحميل التطوير والصيانة.

## توزيعة Oracle Linux باقية وسيتم تطويرها بكل شفافية مع محاولة إبقاء التوافقيه مع RHEL.

وصرحت Oracle أنها ستكمل العمل على لينكس بكامل الشفافية مع محاولة إبقاء التوافقية مع RHEL قدر المستطاع.
وانه لن يكون هناك فرق ملحوظ في أصدار 9.2 الحالي, لكن في الإصدارات المقبلة القادمة ستزداد احتماليه ظهور علل توافقيه, وانهم سيقدمون الدعم لحلها للمشتركين او ISVs(Independent Software Vendor).

ولم تذكر اي تفاصيل حول كيفية تطوير Oracle linux في المستقبل, لكن من المتوقع أن يعتمدون على CentOS stream.

وتكمل الشركة بالوعد بإكمال نشر مصدر توزيعة Oracle linux للعامة, وأنها ترحب بأي توزيعات للنسخ من مصدرها, حتى لو كانت ربحية, وانهم جاهزين للتعاون معهم على محتوى Oracle linux, والتوافقية مع برمجيات Oracle الخاصة.

أيضا تدعو الشركة أي مطور غير معجب بقرار Red hat الاخير للعمل في Oracle, بالإضافة انهم جاهزون لمساعدة مقدمين البرمجيات لدعم Oracle Linux.

**ما رأيكم برد Oracle؟ هل ترونه مجرد تسويق؟ ام فعلا الشركة تهتم بمجتمع Linux؟**