## sms-demo
### 路程
* 使用者一鍵註冊  
  * 彈跳確認框(是、否)  
    * 是 -> 程式自動發簡訊至我們的手機  
    * 否 -> 關閉對話框，結束註冊  
* 手機發送簡訊(內容直接用我們設定之內容格式發送)  
* 手機接收簡訊(首先使用者須開啟獲取簡訊權限設置)  
  * 使用者接收我們的簡訊前，處在Loading，直至背景獲取簡訊訊息，獲取回傳值  
* 伺服器架設  
  * 檢測手機是否被註冊過，已註冊返回回傳值
  * 在使用者一鍵發送簡訊後，端口產生一組用手機為帳號與一組隨機的密碼   
  * 存至mysql後，得到回傳值並回送至手機  
    
### SMS接收 權限設置
```
<uses-permission android:name="android.permission.RECEIVE_SMS"/>
<uses-permission android:name="android.permission.READ_SMS"/>
<uses-permission android:name="android.permission.SEND_SMS"/>
```
