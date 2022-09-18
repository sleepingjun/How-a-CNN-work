[完整版分析報告跟程式碼說明於此](https://drive.google.com/file/d/1ZsimUzSOlDegLBrJxMok6df8sYyilb3W/view?usp=sharing)  

## 前言  
CNN可視化可分成兩種:向前傳播法跟反向傳播法。  
1. 向前傳播法是把模型學習的期間學到的東西圖示出來，比如說可視化filter跟feature map。   
2. 反向傳播法是從特定層反傳回輸入，例如Deconvolution、Guided-Backpropagation、CAM(類激活映射)等。   

第一部分我用下列三種可視化方法來了解VGG16模型是如何判斷圖片:  
1. Visualization filter  
2. Visualization feature map  
3. Heatmap – Grad-CAM、Grad-CAM ++(方法解釋詳見程式碼說明)  

第二部分是在前一部分第三點的基礎上做進一步的探討:  
1. 不同目標層的比較:  
根據模型結構選擇數字最大的卷積層、最後分支中的卷積層、最後一個混和層等。  
2. 不同結構模型的比較:  
選擇直線結構、殘差結構、堆疊結構、殘差堆疊結構四種不同模型，非直線結構的模型僅挑在前一點中效果最好的一個目標層進行比較。  

說明：Grad-CAM和Grad-CAM++兩者的概念都是以“目標層”得到的輸入會是最接近分類特徵的。論文中以及關於這兩者的分享文章都是直接用model. summary()裡面conv2D數字最大的，也是用最後一個卷積(VGG16用block5_conv3)。但我有個疑問，對於模型的結構不是跟VGG一樣是一條直線到最後的結構，這樣有分支跟殘差結構模型的目標層要如何選擇。會有這樣的疑問是因為在殘差還有分支結構提出後，模型的卷積會持續分開跟連結，可能會有最後面試好幾個卷積卷完再合併，或是數字最大的卷積不是最深的情況，而且也想要了解其他層學到特徵的重要度。關於不同的位置的目標層選擇會有什麼樣的影響，在Grad-CAM這篇論文的後面有提到，他們在基於ResNet的VQA模型可視化時發現Grad-CAM中大部分相鄰層次的微小變化以及涉及降維的層次之間有較大的變化。而在Grad-CAM++則是使用AlexNet跟VGG16來測試，主要還是以不同數據集來測試，文末說明關於其他類型的模型只提說希望可以延伸使用。因此增加了第二部分，做進一步的探討。  

## CAM、Grad-CAM、Grad-CAM++的差異  
Grad-CAM(梯度加權類激活映射)比CAM(類激活映射)方法來說更方便，因為CAM必須要有GAP(global average pooling)，這表示如果原先模型沒有的話必須要改變模型結構並重新訓練。但是Grad-CAM沒有這個困擾，他可以適用所有類型的模型。  

Grad-CAM:  
流程圖:  
(框起來的是目標層，通常是最後一個Conv)  
![target layer](https://github.com/sleepingjun/How-a-CNN-work/blob/main/grad-cam%20target%20layer.png)  
![f1](https://github.com/sleepingjun/How-a-CNN-work/blob/main/grad-cam%20formula1.png)  
![f2](https://github.com/sleepingjun/How-a-CNN-work/blob/main/grad-cam%20formula2.png) 

c: class  
y^c: score of c  
A: feature map activations of a convolutional layer  
k: depth  
L: heatmap  

實現步驟:  
1. 計算圖像的模型輸出和最後的捲積層輸出  
2. 在模型輸出中找到預測類別的索引(前面設為label_index)  
3. 計算預測類和目標層的梯度。  
4. 取全局平均，然後與最後一個卷積層相乘  
5. 標準化heatmap，轉換為RGB並將其疊加在原始圖像上  

Grad-CAM++:  
![f](https://github.com/sleepingjun/How-a-CNN-work/blob/main/grad-cam%2B%2B%20formula.png)  
Grad-CAM++是基於Grad-CAM的改良版，論文中說明與當時最好的方法相比，更好的解釋了目標定位和單個圖形上出現多個目標實例的解釋。  
Grad-CAM++和Grad-CAM的差異在於alpha的計算方式不同。  

## 心得  
心得: 
  其實會寫這個主題純粹是意外，一開始我只是想找把特徵圖畫出來的方法，看一下我嘗試寫的模型到底怎麼把飛機認成青蛙的，結果在可視化研究裡面發現了不少有趣的方法。使用反傳播法來可視化的方法其實非常多，不僅有在前言提到的幾個。在這些之中我會選擇heatmap來進一步研究的原因是因為在嘗試幾種反傳播可視化的方法時，發生了很奇妙的事情(如圖)。  
![h1](https://github.com/sleepingjun/How-a-CNN-work/blob/main/failed%20works.JPG)  

  由於論文提供的程式是用tensorflow1.X的版本，而相關的分享文章都是用pytorch，我再次深深感受到版本不兼容的尷尬之處。弄出這兩張”效果”圖的時候其實我還在嘗試用tensorflow2.X版寫出grad-cam++的程式碼。這段期間，製造出了不少奇形怪狀的圖片。在經歷了打錯層數、梯度算到沒數值、矩陣算到變成Nan等等意外後，我終於做出了理想中的圖，但也在錯誤中發現了如果是用熱圖的話更能清楚的感受到對於增加某些特殊層的效果。  
