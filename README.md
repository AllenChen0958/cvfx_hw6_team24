ISA525700 Computer Vision for Visual Effects
Assignment 6: multi-view 3D visual effects


#好讀版網址(格式易讀，影片可直接預覽):https://hackmd.io/e3dCTMIlSd6pyDSXsri2OA?both

Team 24
===

## introduction
本文以 ORB-SLAM2 在影片中插入 3D 物件，完成後並以其結果與使用市面後製軟體產生之 AR 特效進行比較分析，最後也嘗試以另一種非 ORB-SLAM2 之 SLAM 方法在影片中插入 3D 物件。 

## table of content
[]()
## Take videos by ourselves(5%)

{%youtube sCtCjI2dJps  %}

## Insert 3D model to our video with ORB-SLAM2(10%+10%+5%)
<!-- ### 法ㄧ(ORB SLAM) -->
- 簡介
> 該方法利用SLAM將2D的影像，建成3D的模型，並且可計算出相機在該3D的空間中的位置。本次實驗，及為利用ORB-SLAM得知相機的位置資訊，來投放3D model以達到AR的效果。
- 步驟
    - 先將拍攝的影片，切成一個一個frame，放入名稱為rgb的資料夾內。並建立一個名稱為rgb.txt的文件紀錄剛剛生成的每張frame的時間戳記以及其存放位置。
    - 依照自己的相機規格，修改.yml檔案。舉例來說，如下圖，fx和fy需要查詢自己相機的focal length規格(EX: 下圖為samsung s7故其規格為26mm~98.267717 pixels)。cx以及cy代表，相機中心點，這裡假設沒有偏移量，故相機中心點就是最後相片的中心位置(EX: samsung s7拍攝出來的相片是1920x1080，故其中心位置及為(960,540))
    ![](https://i.imgur.com/H0RqpVT.png)


    - 參照助教給的參考資料，用ORB-SLAM將key frame trajectory抓出來
    其生成檔案KeyFrameTrajectory.txt內容如下：
    ![](https://i.imgur.com/IIP78FR.jpg)
    第一直行為時間戳記
    第二行到第四行 為目前相機在空間的位移(Extrinsic matrix中位移的部分)
    第五到第八行 為目前相機在空間中的轉動(orientation)。其四個量為qx, qy, qz, qw，如下表示
    qx = RotationAxis.x * sin(RotationAngle / 2)
    qy = RotationAxis.y * sin(RotationAngle / 2)
    qz = RotationAxis.z * sin(RotationAngle / 2)
    qw = cos(RotationAngle / 2)
    上式的RotationAngle代表的是弧度
    而使用orientation form來計算角位移，主要是可避免像是[Gimbal Lock](https://www.youtube.com/watch?v=zc8b2Jo7mno)的問題(先轉x軸再轉y軸 和 先轉y軸再轉x軸會有不同結果)，計算起來也較方便

    - 將之前影片切好的frame依序讀入，並根據KeyFrameTrajectory.txt內容得知key frame中的相機位置
    - 將剩下不是key frame的相機位置內差出來。
    - 利用每個frame的相機位置計算Extrinsic matrix
    - 利用相機的Intrinsic matrix和Extrinsic matrix計算出projection matrix，並將3D model利用該matrix投影到每個frame上。
    (3D model的Rotation以及Transform和上色詳見補充資料)
- 影片結果
<!-- [https://drive.google.com/open?id=1HvnVV5PtikgFieYHSlk75l-6ZDP8etSI] -->
{%youtube H5d86scqkzw %}

## Make these visual effects with any post-production software(10%)
- 簡介
> 本次使用的後製軟體是 Adobe After Effect(AE) 以及 Maxon Cinema 4D(C4D)，利用 AE 內建的 motion track抓出特徵點後，建立3D模型匯入C4D進行建模，完成後再匯回 AE 達成 AR效果。
- 步驟
    - 先將影片匯入 AE 進行 motion traking 抓出特徵點。完成結果如下:
    ![](https://i.imgur.com/SWebC4J.jpg)
    - 為了跟 ORB SLAM2 進行比較，在與其相同位置平面上增加一層 3D layer 
    ![](https://i.imgur.com/bk69GQE.jpg)
    - 移除2D影片軌道之後，匯出成 C4D 格式模型檔案，並使用C4D 進行加入模型的動作
    - 模型加入後再確認影片光源，在模型中加入環境光源並使用平方反比定律渲染，結果如下:
    ![](https://i.imgur.com/PlkiZij.png)
    - 把完成的模型再次匯入 AE ，並把影片加回軌道上，並調整茶壺出現時間與 ORB slam2 結果相同
- 影片結果
因影片改變拍攝方式，故輸出影片上面完成結果不盡相同
{%youtube Kjwrfo9s2Uw %}

 
## Compare above methods
下面將進行 ORB-SLAM2 與後製軟體 AE 之比較分析，主要聚焦於速度與輸出品質。
#### 比較影片
{%youtube L0wXeO8J4z0 %}

#### 比較分析
|偵測特徵點時間|插入物件速度|材質細節|立體效果|輸出影片速度|
|--|--|--|--|--|
|1st|1st|2nd|2nd|1st|
|2nd|2nd|1st|1st|2nd|

- 速度上來看，使用ORB-SLAM2理論上可以更快，但是這取決於程式的完整度。
- 品質上因為投影使用程式產生AR在光源產生部分有困難，故立體效果較使用後製軟體差。
- 轉換輸出時間後製軟體因為有進行顯示卡加速優化，故快上許多。
## Use other SLAM methods(10%)
### ORB AR
- 簡介
> 該方法使用Homograhy matrix，反推出相機的Extrinsic matrix。再加上相機本身的Intrinsic Matrix，即可將3D model投影到2D影像上，來達成AR的效果。


- 步驟
- 拍攝某個容易抓出feature點的圖型作為參考圖(本次實驗選擇使用螢幕如下圖)
picture[reference link]
- 使用ORB feature extraction的方式抽出參考圖的feature points
- 使用ORB feature extraction的方式對每個frame都抽出feature points
- 拿參考圖的feature points和每個frame的feature points進行比對，並計算Homograhy matrix
- 依照下式，如欲將3D Model投影至2D的平面，需要知道camera的Intrinsic matrix以及Extrinsic matrix。而目前已知相機本身的Intrinsic matrix K，以及計算出的Homograhy matrix Ｈ，
![](https://i.imgur.com/CXmqpGP.png)








## Conclusion：
- 拍攝影片時，如果要讓ORB-SLAM快速抓到key points，可以在某個位置上下左右移動一下，會使其更容易抓出key points
- 拍攝時，桌面越亂越好，因為這樣會比較容易抓到feature point
- 因為有內差keyframe的關係，故影像會有點像是飄移(不做內差會像是瞬間移動)



## 補充資料:
[3D model投影]

Reference:

[Rotation](http://www.opengl-tutorial.org/intermediate-tutorials/tutorial-17-quaternions/)
