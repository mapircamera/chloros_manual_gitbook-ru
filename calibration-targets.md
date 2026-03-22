---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Калибровочные мишени

MAPIR предлагает различные калибровочные мишени для широкого спектра применений. Компактная модель T4-R50, представленная ниже, содержит 4 панели, для которых были измерены показатели светоотражения в диапазоне от 250 до 2500 нм.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Диффузные эталонные мишени T4 имеют следующие кривые отражения, [скачать данные здесь](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Отражение T4 :: 250–2500 нм</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Отражение T4 :: 400–1000 нм</p></figcaption></figure>Диффузные эталонные мишени T4P имеют следующие кривые отражения, [скачать данные здесь](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P Отражение :: 250–2500 нм</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR Отражение T4P :: 400–1000 нм</p></figcaption></figure>На графике отражения видно, что значения представлены в виде зависимости длины волны (ось x) от процента отражения (ось y). Когда мы снимаем изображение калибровочной мишени, мы устанавливаем соотношение между значением пикселя и процентом отражения в пределах спектра, к которому чувствительны все диапазоны датчика камеры.

Это означает, что для каждого изображения, снятого нашими камерами, можно использовать фотографию наших мишеней отражения, таких как [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) или [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), чтобы откалибровать изображения по отражению. После калибровки каждый пиксель изображения соответствует проценту отражательной способности.

Если вы экспортируете откалиброванные изображения в формате Chloros как обычный JPG или TIFF, то процент отражения рассчитывается путем деления значения пикселя на разрядность формата изображения. Так, для JPG делите на 255, а для TIFF — на 65 535. Вы также можете выбрать формат вывода PERCENT в формате Chloros, и тогда каждый пиксель будет иметь значение от 0,0 до 1,0 (от 0% до 100% отражательной способности). Просто имейте в виду, что некоторые приложения для работы с изображениями не поддерживают изображения в процентах (с плавающей запятой), а также что такие изображения занимают много места при хранении.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
