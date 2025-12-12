---
description: Панели, измеренные в лабораторных условиях, используемые для калибровки собранных данных при постобработке.
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Калибровочные мишени

MAPIR предлагает различные цели калибровки для широкого спектра применений. Компактный T4-R50, показанный ниже, содержит 4 панели, коэффициент отражения света которых был измерен в диапазоне 250–2500 нм.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

Диффузные эталонные мишени T4 имеют следующие кривые отражения, [данные скачать здесь](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Отражательная способность:: 250–2500 нм</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Отражательная способность:: 400–1000 нм</p></figcaption></figure>

Глядя на график отражения, вы можете видеть, что значения представляют собой длину волны (ось X) и процент отражения (ось Y). Когда мы захватываем изображение калибровочной цели, мы затем создаем взаимосвязь между значением пикселя и процентом отражения в пределах спектра, к которому чувствительна каждая из полос сенсора камеры.

Это означает, что для каждого изображения, которое вы снимаете с помощью наших камер, вы можете использовать фотографию наших объектов по отражательной способности, например [Т4-Р50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) или [Т4-Р125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), для калибровки изображений по отражательной способности. После калибровки каждый пиксель изображения равен проценту отражения.

Если вы выводите откалиброванные изображения в Chloros в обычном формате JPG или TIFF, то процент отражения рассчитывается путем деления значения пикселя на разрядность формата изображения. Итак, для JPG разделите на 255, а для TIFF — на 65 535. Вы также можете выбрать вывод в формате ПРОЦЕНТ в Chloros, и тогда каждый пиксель будет иметь процентное значение от 0,0 до 1,0 (отражение от 0% до 100%). Просто имейте в виду, что некоторые приложения для работы с изображениями не могут принимать изображения в процентах (с плавающей запятой), и они имеют большой размер хранилища.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
