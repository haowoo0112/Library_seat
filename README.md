# Library_seat

> [Repo](https://github.com/YYHsiang/Library_seat) 

> [Demo Video](https://youtu.be/DGnQsDQReWI)

## 摘要
本專題是應用在圖書館的座位管理系統，我們發現學校的圖書館，在尋找座位時，必須全部巡視過，才能確定是否有空位，所以我們決定利用影像辨識來幫助大家尋找空位。
利用座位上方架設的攝影機，獲得影像，經過辨識之後，得出座位的使用情況，再經由網頁顯示各區域的座位佔用情況，方便使用這更快找到座位，在沒有座位時，也不會白跑一趟。

## 原理
### 1. 攝影機架設
使用Python Tkinter作為圖形界面開發工具。
#### 流程
> Step 1.	使用手機安裝IP Webcam之後，開啟攝影機架設界面，在界面底部輸入對應的IP位址連結相機，成功連結後，會顯示攝影機畫面。  
> Step 2. 在攝影機畫面上使用右上角，「Rect」和「Point」 Selection tool，來分別框選座椅和桌面區域區域，並輸入編號、位置及攝影機資料。  
> Step 3. 按下「Database」，開啟新視窗，儲存當前的設定資料，也可以讀取先前的資料。  
> Step 4. 按下「YOLO detect」，運行影像辨識功能。  

![攝影機架設流程](https://github.com/YYHsiang/Library_seat/blob/master/%E6%94%9D%E5%BD%B1%E6%A9%9F%E6%9E%B6%E8%A8%AD%E6%B5%81%E7%A8%8B.jpg)

#### 資料庫
攝影機設定界面，擁有存取設定資料的功能，使用SQLite架設。其中儲存使用者框選的桌面、座椅的相關資料，以及初始設定時使用的攝影機圖片。

### 2. 影像辨識
辨識座位使用兩種不同的模型，因為我們發現，當桌面放置的物體是雜物，YOLOv3[2]就無法成功辨識，所以使用CNN分類器，判斷桌面是否有物體，為了適應不同桌面，我們將空桌面的圖片與攝影機擷取畫面，進行structure similarity[1]運算，只取出兩張圖片的差異作為CNN分類器的輸入。
影像辨識流程 (圖三)  

> Step 1. 擷取攝影機的畫面，並取出所有座位的資料。一個座位會有兩種檔案，第一種是座椅檔案，存放座椅在圖片中的位置，第二種是桌面檔案，存放桌面在圖片中的位置，以及空桌面的圖片。    
> Step 2. 使用座椅的座標，對擷取畫面進行裁切，並分別將每個座椅的圖片輸入給 YOLOv3[2]，判斷裁切後的圖片是否有人類，作為椅子上是否有坐人的依據。  
> Step 3. 使用桌面的座標，對擷取畫面進行裁切，並分別將每個桌面的圖片與空桌面的圖片，進行structure similarity[1]運算，再將結果當作CNN分類器的輸入，判斷桌面是否有物體。  
> Step 4. 綜合Step 2及Step 3的結果，判斷此座位是否被佔用，並傳送結果至網頁伺服器。  

![影像辨識流程](https://github.com/YYHsiang/Library_seat/blob/master/%E5%BD%B1%E5%83%8F%E8%BE%A8%E8%AD%98%E6%B5%81%E7%A8%8B.jpg)

#### CNN分類器  
經由結構相似 structure similarity[1]處理過後的影像，接著會經過一個CNN，這個CNN由2個卷積層(Convolution Layer)、2個池化層(Polling Layer)、扁平層(Flatten Layer)和2層全連接層(Fully Connected Layer)。
第1個卷積層使用16個5×5的卷積核做卷積，接著2×2的最大值池化層，第2個卷積層使用32個5×5的卷積核做卷積，接著2×2的最大值池化層，使用最大值池化是為了更加凸顯特徵，在平均值池化與最大值池化的比較中，最大值池化的效果較為顯著。

![CNN分類器 架構圖](https://github.com/YYHsiang/Library_seat/blob/master/CNN%E5%88%86%E9%A1%9E%E5%99%A8.jpg)

#### YOLO v3  
YOLO v3使用官方的yolov3.weights，在輸出的地方，只取出人類的標籤，判斷圖片內是否有人。  

### 3. 網頁顯示
網站使用Django架構，其中的功能包含: 顯示座位資訊、查看IP Camera畫面、圖形化系統資料、即時更新網頁資訊。  

#### 顯示座位資訊
在網頁的左邊可以選擇位置，觀看不同地方的座位使用情形，網頁最上方會顯示本區域剩下的座位數，中間顯示場域座位平面圖，平面圖中，空座位顯示藍色，被佔用的座位顯示灰色，並且使用Django-channel，只要座位使用狀況更新，網頁就會同步更新，不須手動按下重新整理按鈕。

#### 圖形化系統資料
將系統中的座位分佈、座位平均佔用時間等等，使用圖表顯示，讓管理者能夠藉由統計資料，改善場域的規劃。

## 成果 Result

![情況一](https://github.com/YYHsiang/Library_seat/blob/master/%E6%83%85%E6%B3%81%E4%B8%80.jpg)
![情況二](https://github.com/YYHsiang/Library_seat/blob/master/%E6%83%85%E6%B3%81%E4%BA%8C.jpg)

#### 攝影機架設介面 
![界面](https://github.com/YYHsiang/Library_seat/blob/master/%E7%95%8C%E9%9D%A2.jpg)

#### 系統分析
![分析](https://github.com/YYHsiang/Library_seat/blob/master/%E7%B6%B2%E9%A0%81%E5%88%86%E6%9E%90.jpg)

#### 即時影像
![即時影像](https://github.com/YYHsiang/Library_seat/blob/master/%E5%8D%B3%E6%99%82%E5%BD%B1%E5%83%8F.jpg)

### Reference
[1] Z. Wang, A. C. Bovik, H. R. Sheikh, and E. P. Simoncelli, “Image quality assessment: From error visibility to structural similarity,” IEEE Trans. Image Process., vol. 13, no. 11, pp. 600– 612, Apr. 2004.
[2] Joseph Redmon, Ali Farhadi, “YOLOv3: An Incremental Improvement”, 	arXiv, 2018.
