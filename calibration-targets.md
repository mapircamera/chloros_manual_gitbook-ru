---
description: Lab-measured panels used to calibrate captured data in post-processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---
# Калибровочные мишени

MAPIR предлагает различные калибровочные мишени для широкого спектра применений. Компактная мишень T4-R50, показанная ниже, содержит 4 панели, которые были измерены на светоотражение в диапазоне от 250 до 2500 нм.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Диффузные эталонные мишени T4 имеют следующие кривые отражения, [скачать данные здесь](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Отражательная способность :: 250–2500 нм</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Отражательная способность :: 400-1000 нм</p></figcaption></figure>На графике отражательной способности видно, что значения представлены в виде соотношения длины волны (ось x) и процента отражательной способности (ось y). Когда мы снимаем изображение калибровочной мишени, мы создаем соотношение между значением пикселя и процентом отражательной способности в спектре, к которому чувствительны все полосы датчика камеры.

Это означает, что с каждым изображением, снятым с помощью наших камер, вы можете использовать фотографию наших мишеней отражения, таких как [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) или [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), для калибровки изображений по отражению. После калибровки каждый пиксель изображения равен проценту отражательной способности.

Если вы выводите откалиброванные изображения в Chloros в виде типичного JPG или TIFF, то процент отражения рассчитывается путем деления значения пикселя на битовую глубину формата изображения. Так, для JPG делим на 255, а для TIFF делим на 65 535. Вы также можете выбрать вывод в формате PERCENT в Chloros, и тогда каждый пиксель будет иметь значение от 0,0 до 1,0 (0% до 100% отражательной способности). Просто имейте в виду, что некоторые приложения для работы с изображениями не могут принимать изображения в процентах (с плавающей запятой), и они занимают много места при хранении.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
