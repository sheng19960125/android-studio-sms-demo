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
    
### SMS接收 權限設置(permission)
將以下放置 AndroidManifest.xml
```
<uses-permission android:name="android.permission.RECEIVE_SMS"/>
<uses-permission android:name="android.permission.READ_SMS"/>
<uses-permission android:name="android.permission.SEND_SMS"/>
```
### SMS架構
_id => 短訊息序號 如100  
thread_id => 對話的序號 如100  
address => 發件人地址，手機號.如+8613811810000  
person => 發件人，返回一個數字就是聯絡人列表裡的序號，陌生人為null  
date => 日期  long型。如1256539465022  
protocol => 協議 0 SMS_RPOTO, 1 MMS_PROTO   
read => 是否閱讀 0未讀， 1已讀   
status => 狀態 -1接收，0 complete, 64 pending, 128 failed   
type => 型別 1是接收到的，2是已發出   
body => 短訊息內容   
service_center => 簡訊服務中心號碼編號。如+8613800755500  

### 主程式架構
實現持續監聽SMS訊息  
首先 判斷是否使用者同意SMS權限設置
要求權限同意申請  
```
ActivityCompat.requestPermissions(this,new String[]{Manifest.permission.READ_SMS},REQ_CODE_CONTACT);  
```
同意後 SMS獲取 放置ListVIew 方便觀看
```
//讀取所有簡訊
Uri uri = Uri.parse("content://sms/");
ContentResolver resolver = getContentResolver();
Cursor cursor = resolver.query(uri, new String[]{"_id", "address", "body", "date", "type"}, null, null, null);
if (cursor != null && cursor.getCount() > 0) {
  int _id;
  String address;
  String body;
  String date;
  int type;
  
  while (cursor.moveToNext()) {
  
    Map<String, Object> map = new HashMap<String, Object>();
    
    _id = cursor.getInt(0);
    address = cursor.getString(1);
    body = cursor.getString(2);
    date = cursor.getString(3);
    type = cursor.getInt(4);
    map.put("names", body);
    
    Log.i("Myinfo", "_id=" + _id + " address=" + address + " body=" + body + " date=" + date + " type=" + type);
  }
}
```

