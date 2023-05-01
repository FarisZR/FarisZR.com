---
title: برمجية اقتراحات الكود Github Copilot وتداعيتها على البرمجيات الحرة
categories: 
    - نقاشات
date: 2022-02-20
image: thumbnail.jpeg
canonicalurl: https://discourse.aosus.org/t/topic/2354
slug: github-copilot-foss-copyright
tags:
    - برمجة
    - github
    - حقوق-النشر
    - fsf
    - gpl
    - التراخيص
---

**هذا المقال كُتب على مجتمع أسس, كامل التعليقات [هنا](https://discourse.aosus.org/t/topic/2354)**

السلام عليكم ورحمة الله وبركاتة

في شهر يونيو 2021, أعلنت منصة Github, أكبر منصة أستضافة أكواد برمجية في العالم, وهي تابعة ل Microsoft عن اطلاق نسخه تجريبية من مساعدها البرمجي Github Copilot.

برمجية Copilot هي عبارة عن ذكاء اصطناعي يقترح عليك أكواد برمجية بناءا على الكود البرمجي المكتوب سابقا, ويمكنه اقتراح كود بسيط او حتى دوال(functions) كاملة.

تم تطويرة بالتعاون مع OpenAI ويستخدم نموذج OpenAI Codex للذكاء الاصطناعي, وهو يستطيع فهم الكود البرمجي بشكل اكبر, وهو افضل بشكل كبير في كتابة الكود البرمجية من نموذج [GPT-3](https://en.wikipedia.org/wiki/GPT-3)

## أستخدام كامل المستودعات العامة
أكبر الخلاف على Copilot ليس حول قدراته, او النموذج المغلق الذي تم استخدامه لبناءة(نعم اسم المنظمة OpenAI لكن كل شيء مغلق), المشكلة الاكبر في تراخيص المستودعات العامة.

### البرمجيات المفتوحة
هذه برمجيات تستخدم تراخيص مفتوحة المصدر, والمقصود انه برمجيات تكشف المصدر للعامة ويمكنك تعديلها واعادة توزيعها.
وأشهرها هي تراخيص MIT, BSD, Apache وغيرها من التراخيص.
هذه التراخيص لا تضع حدود على الاستخدام معظم الوقت, يعني بإمكانك استخدام الكود في برنامَج مفتوح تحت ترخيص أخر, او تحت برنامَج مغلق. معظم الوقت فقط تطلب وضع توضيح ملكية الكود لأصحاب المستودع, بالإضافة لحمايات قانونية لكتاب الكود.

### البرمجيات الحرة
البرمجيات الحرة يمكن القول عنها انها مفتوحة ايضا, فمصدرها مفتوح و يمكن لاي احد تعديله وأعاده توزيعة.
الفرق ان البرمجيات الحرة تكون لها شروط اقوى, أهم شرط هو ابقاء الكود مفتوح المصدر.
واكبر ترخيص للبرمجيات الحرة هو الترخيص الشهير [GPL](https://ar.wikipedia.org/wiki/%D8%B1%D8%AE%D8%B5%D8%A9_%D8%AC%D9%86%D9%88_%D8%A7%D9%84%D8%B9%D9%85%D9%88%D9%85%D9%8A%D8%A9) المستخدم من قبل نواة لينكس. أذ يشترط الترخيص نشر الكود المعدل تحت نفس الترخيص في حالة تم توزيعة.

وهناك عدة نسخ معدلة من الترخيص مثل AGPL, وهو أكثر ترخيص صرامة يعدّ مفتوح المصدر حسب علمي, وهو يعتبر الاستخدام عبر الأنترنت توزيع. واحد اشهر البرامج التي تستخدمه هي Nextcloud.

هناك أيضا نسخ مخففة منة, مثل LGPL وهو يسمح للكود أن يتم أستخدامه داخل برمجية اخرى, دون تحويل كامل ترخيص البرمجية الى GPL, مع ابقاء الكود الحر مكشوف.
 ومثال اخر على نسخ معدلة منه هو ترخيص [MPL](https://ar.wikipedia.org/wiki/%D8%B1%D8%AE%D8%B5%D8%A9_%D9%85%D9%88%D8%B2%D9%8A%D9%84%D8%A7_%D8%A7%D9%84%D8%B9%D9%85%D9%88%D9%85%D9%8A%D8%A9) من [Mozilla](https://mozilla.org).

هذا شرح مختصر جدا جدا جدا, يمكن فتح موضوع مُفصّل حول هذا النقاش والفرق بين الفكرتين والتراخيص و التاريخ الحافل بينهم. لكنني حاولت اشرح الفرق بسهولة لأوصل الفكرة.

## مشكلة التراخيص في Copilot
بما أن Copilot يبني معرفته على المستودعات العامة, فهو سيقرا ويتعلم على كود تحت اي ترخيص, من ضمنها تراخيص حرة, كالGPL, وهذه التراخيص كما تطرقت سابقا لها شروط واستخدام اجزاء منها قد يطبق شروطها على البرنامَج الذي تطوره.

لذلك قد يقترح عليك كود تحت ترخيص حر, وانت مشروعك تحت ترخيص مفتوح أو حتى مغلق. 
وهذا لا يصح قانونيا لأنة يكسر شروط الترخيص, وقد يمكن القول عنه سرقة للكود.

## الأوراق البحثية التي تم تمويلها من منظمة البرمجيات الحرة
بعد أعلان Github عن برمجية الذكاء الاصطناعي Copilot, قامت منظمة البرميجات الحرة (FSF) بطلب تقديم الباحثين باوراق بحثية عن حقوق الملكية في البرمجيات الحرة, و كيفية تأثير Copilot وبرمجيات اقتراح الاكواد الذكية عليها لدعمها ماليا.

واعلنت ال FSF عن الأوراق التي تم تمويلها وهي باللغة الانجليزية.

### Copilot, copying, commons, community, culture

* Robert F.J. Seddon, Honorary Fellow, University of Durham
* [PDF](https://static.fsf.org/nosvn/copilot/Copilot-Copying-Commons-Community-Culture.pdf)
* [HTML](https://www.fsf.org/licensing/copilot/copilot-copying-commons-community-culture)
* [CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0/)

### Copyright implications of the use of code repositories to train a machine learning model

* John A. Rothchild, Professor of Law, Wayne State University and Daniel H. Rothchild, PhD candidate, University of California, Berkeley
* [PDF](https://static.fsf.org/nosvn/copilot/Copyright-Implications-of-the-Use-of-Code-Repositories-to-Train-a-Machine-Learning-Model.pdf)
* [HTML](https://www.fsf.org/licensing/copilot/copyright-implications-of-the-use-of-code-repositories-to-train-a-machine-learning-model)
* [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)

### If software is my copilot, who programmed my software?
* Bradley M. Kuhn, Policy Fellow, Software Freedom Conservancy
* [PDF](https://static.fsf.org/nosvn/copilot/if-software-is-my-copilot-who-programmed-my-software.pdf)
* [HTML](https://www.fsf.org/licensing/copilot/if-software-is-my-copilot-who-programmed-my-software)
* [CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0/)

### Interpreting docstrings without common sense
* Darren Abramson, Associate Professor, Dalhousie University and Ali Emami, assistant professor, Brock University
* [PDF](https://static.fsf.org/nosvn/copilot/Interpreting-Docstrings-Without-Common-Sense.pdf)
* [HTML](https://www.fsf.org/licensing/copilot/interpreting-docstrings-without-using-common-sense)
* [CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0/)

### On the nature of AI code copilots
* Stuart Fitzpatrick, Doctoral Candidate, Western Sydney University
* [PDF](https://static.fsf.org/nosvn/copilot/On_the_nature_of_ai_copilots.pdf)
* [HTML](https://www.fsf.org/licensing/copilot/on-the-nature-of-ai-code-copilots)
* [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)

## الخطوات القادمة من منظمة البرمجيات الحرة FSF
حاليا لا يوجد اي خطوات واضحه غير النقاش حول هذه الاوراق.
عدد من الكتاب سيحضرون نقاشات حول محتوى هذه الاوراق.

### مواعيد النقاشات المباشرة

* 13:00 EST (19:00 UTC) Thursday, March 3, IRC ( `#fsf` ), [Q&A with Robert F.J Seddon](https://www.fsf.org/events/copilot-irc-q-a-with-robert-f-j-seddon)
* 13:00 EST (19:00 UTC) Monday, March 7, IRC ( `#fsf` ), [general discussion](https://www.fsf.org/events/join-us-for-a-general-irc-discussion-on-the-papers-selected-as-part-of-our-copilot-call-for-whitepapers)

جميعها على بروتوكول IRC.

مارأيكم ؟ هل يعتبر Copilot محاولة لسرقة الكود البرمجي للبرمجيات الحرة؟ ام انها تقنية جديدة لتحسن وتسرع انتاجية المطور ؟

## المصادر
https://www.fsf.org/news/publication-of-the-fsf-funded-white-papers-on-questions-around-copilot
https://github.blog/2021-06-29-introducing-github-copilot-ai-pair-programmer/
https://copilot.github.com/
https://ar.wikipedia.org/wiki/%D8%AD%D8%B1%D9%83%D8%A9_%D8%A7%D9%84%D8%A8%D8%B1%D9%85%D8%AC%D9%8A%D8%A7%D8%AA_%D8%A7%D9%84%D8%AD%D8%B1%D8%A9
https://ar.wikipedia.org/wiki/%D8%A8%D8%B1%D9%85%D8%AC%D9%8A%D8%A7%D8%AA_%D9%85%D9%81%D8%AA%D9%88%D8%AD%D8%A9_%D8%A7%D9%84%D9%85%D8%B5%D8%AF%D8%B1