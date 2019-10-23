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

_id       => 短訊息序號 如 1,2,3...100  
thread_id => 對話的序號 如 1,2,3...100  
address   => 發件人地址，手機號   
person    => 發件人，返回一個數字就是聯絡人列表裡的序號，陌生人為 null  
date      => 日期  long型。如 1256539465022  
protocol  => 協議 0 SMS_RPOTO, 1 MMS_PROTO   
read      => 是否閱讀 0未讀， 1已讀   
status    => 狀態 -1接收，0 complete, 64 pending, 128 failed   
type      => 型別 1是接收到的，2是已發出   
body      => 短訊息內容   

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
    
    _id     = cursor.getInt(0);
    address = cursor.getString(1);
    body    = cursor.getString(2);
    date    = cursor.getString(3);
    type    = cursor.getInt(4);
    map.put("names", body);
    
    Log.i("Myinfo", "_id=" + _id + " address=" + address + " body=" + body + " date=" + date + " type=" + type);
  }
}
```

設置監聽每秒
```
runnable = new Runnable( ) {
  public void run ( ) {
                   
    handler.postDelayed(this,2000);
    //postDelayed(this,2000)方法安排一個Runnable物件到主執行緒佇列中
  }
};

// 開始Timer
handler.postDelayed(runnable,1000); 
```

綜合以上需求 即可每秒監測SMS訊息是否有獲取到新的使用者發送的註冊SMS
如果收到新的SMS註冊，即可當場發送POST至SERVER，註冊新的使用者，返回註冊資訊
```
public class MainActivity extends AppCompatActivity {

    private ListView mListView;
    private SimpleAdapter sa;
    private Button mButton;
    private List<Map<String, Object>> data;
    public static final int REQ_CODE_CONTACT = 1;

    private Handler handler = new Handler( );
    private Runnable runnable;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initButton();
        initView();
    }

    private void initButton() {
        mButton = (Button) findViewById(R.id.btn);
        mButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                checkSMSPermission();
            }
        });
    }

    private void initView() {
        //得到ListView

        mListView = (ListView) findViewById(R.id.listView);
        data = new ArrayList<Map<String, Object>>();

        sa = new SimpleAdapter(this, data, android.R.layout.simple_list_item_2,
                new String[]{"names", "message"}, new int[]{android.R.id.text1,
                android.R.id.text2});
        mListView.setAdapter(sa);
    }

    /**
     * 檢查申請簡訊許可權
     */
    private void checkSMSPermission() {
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_SMS)
                != PackageManager.PERMISSION_GRANTED) {
            //未獲取到讀取簡訊許可權

            //向系統申請許可權
            ActivityCompat.requestPermissions(this,
                    new String[]{Manifest.permission.READ_SMS}, REQ_CODE_CONTACT);
        } else {
            runnable = new Runnable( ) {
                public void run ( ) {
                    query();
                    handler.postDelayed(this,2000);
                    //postDelayed(this,2000)方法安排一個Runnable物件到主執行緒佇列中
                }
            };
            handler.postDelayed(runnable,1000); // 開始Timer
        }
    }


    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        //判斷使用者是否，同意 獲取簡訊授權
        if (requestCode == REQ_CODE_CONTACT && grantResults.length > 0
                && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            //獲取到讀取簡訊許可權
            checkSMSPermission();
        } else {
            Toast.makeText(this, "未獲取到簡訊許可權", Toast.LENGTH_SHORT).show();
        }
    }

    private void query() {

        data.clear();

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
                data.add(map);
                //通知介面卡發生改變
                sa.notifyDataSetChanged();
            }
        }
    }
}
```
