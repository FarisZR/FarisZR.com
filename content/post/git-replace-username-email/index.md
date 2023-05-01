---
title: كيف تغير أسم وبريد مساهم في تاريخ Git
categories: 
    - شروحات
date: 2021-11-01
image: thumbnail.jpeg
slug: git-replace-username-email
canonicalurl: https://discourse.aosus.org/t/topic/2182
tags:
    - برمجة
    - git
---
**هذا المقال كُتب على مجتمع أسس, كامل التعليقات [هنا](https://discourse.aosus.org/t/topic/2182)**

السلام عليكم ورحمة الله وبركاتة
في حاله قد غيرت اسمك في استضافه Git, او انتقلت الى استضافه اخرى, او حتى لو تريد اخفاء بريدك من تاريخ Git. في هذا الموضوع سوف أشرح كيفيه التعديل على تاريخ Git من اجل تعديل اسماء وبريد المساهمين.

![git-preview|690x388](thumbnail.jpeg)
شعار Git هو من تصميم [Jason Long](https://twitter.com/jasonlong)

 اذا بحثت في الانترنت, ستجد الاغلبيه يستخدمون `git-filter-branch` لكن غير منصوح باستخدامها من مشروع Git نفسهم لذلك سنستخدم `git-filter-repo`

## تثبيت git-filter-repo
```
pip3 install git-filter-repo
```

**قبل عمل اي تغيير تاكد من وجود نسخه احتياطيه من المستودع, سواء اشتقاق عام او نسخة محلية**

## تغيير البريد والاسم

> اذا كنت تريد اخفاء بريدك لكن تريد ان تظهر التعديلات باسمك, معظم مواقع استضافة Git تقدم بريد noreply
في Github هو ظاهر في صفحه البريد سيكون العنوان `xxx+username@users.noreply.github.com`
اما Gitlab سيكون موجود في Profile تحت اسم Commit email `xxx+username@users.noreply.gitlab.com`
استضافات gitea عادة تستخدم `username@noreply.giteadoman.tld` ويكون اسم المستخدم بدون اي اضافة

```
git-filter-repo --name-callback 'return name.replace(b"<old-username>", b"<new-username>")' --email-callback 'return email.replace(b"<old@email.com>", b"<new@email.com>")'  --replace-refs delete-no-add --force
```
- `<old-username>` = أسم المستخدم القديم
- `<new-username>` = أسم المستخدم الجديد
- `<old@email.com>` = البريد القديم
- `<new@email.com>` = البريد الجديد

## تغيير الاسم فقط
```
git-filter-repo --name-callback 'return name.replace(b"<old-username>", b"<new-username>")'  --replace-refs delete-no-add --force
```
- `<old-username>` = أسم المستخدم القديم
- `<new-username>` = أسم المستخدم الجديد

## تغيير البريد فقط
> اذا كنت تريد اخفاء بريدك لكن تريد ان تظهر التعديلات باسمك, معظم مواقع استضافة Git تقدم بريد noreply
في Github هو ظاهر في صفحه البريد سيكون العنوان `xxx+username@users.noreply.github.com`
اما Gitlab سيكون موجود في Profile تحت اسم Commit email `xxx+username@users.noreply.gitlab.com`
استضافات gitea عادة تستخدم `username@noreply.giteadoman.tld` ويكون اسم المستخدم بدون اي اضافة
```
git-filter-repo  --email-callback 'return email.replace(b"<old@email.com>", b"<new@email.com>")'  --replace-refs delete-no-add --force
```

- `<old@email.com>` = البريد القديم
- `<new@email.com>` = البريد الجديد

## اضافه رابط المستودع
بعد كل هذه التعديلات سيتم حذف ال remote
هذه الاوامر لاضافه ال remote مره اخرى
```
git remote add origin <repolink>
git remote set-url origin <repolink>
```
- `<repolink>` = رابط المستودع
## رفع التعديلات
```
git push -u origin <target-branch> --force
```
- `<targe-brach>` = اسم الفرع, مثلا main او master
## المصدر
https://stackoverflow.com/a/60364176