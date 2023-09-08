## ChangeOrder 變更通知



### 1. 詳細資訊 

#### 1-1. 取得 WTChangeOrder2 企業屬性

```
public static void test() throws Exception {
    String ecno = "C386210005";
    WTChangeOrder2 eco = ext.generic.util.ChangeUtil.getChangeNoticeByNumber(ecno, true);

    String buName = (String) ext.mitac.report.util.IBAValueHelper.getIBAValue(eco, "BUname");
    String description = eco.getDescription();

    System.out.println("單據號碼："+eco.getNumber());
    System.out.println("單據名稱："+eco.getName());
    System.out.println("單據屬性(BU)："+buName);
    System.out.println("單據描述："+description);
}

```

#### 1-2. 取得 WTChangeOrder2 單據狀態

```

String ecno = "C386210005";
WTChangeOrder2 eco = ext.generic.util.ChangeUtil.getChangeNoticeByNumber(ecno, true);
String ecnState = eco.getLifeCycleState().toString();

```

#### 1-3. 附件檔案 - 以 ChangeList.xls 為例

```

public static void test() throws Exception {
    String ecno = "C528210057";
    WTChangeOrder2 eco = ext.generic.util.ChangeUtil.getChangeNoticeByNumber(ecno, true);

    //ChangeList.xls
    File changeList = getChangeItem(eco);
}

public static File getChangeItem(Object businessObject) throws WTException, IOException {
    System.out.println("getChangeItem.....");
    String fileName = "";
    List<Object> changeItemList = null;
    WTChangeOrder2 co = (WTChangeOrder2) businessObject;
    InputStream is = null;
    QueryResult qr = null;
    qr = ContentHelper.service.getContentsByRole(co, ContentRoleType.SECONDARY);
    ApplicationData ap = null;
    while (qr.hasMoreElements()) {
        Object o = qr.nextElement();
        if (o instanceof ApplicationData) {
            ap = (ApplicationData) o;
            fileName = ap.getFileName();
            if (fileName.startsWith("ChangeList_")
                    && fileName.endsWith(".xls")) {
                qr = null;
                break;
            }
            ap = null;
        }
    }

    if (ap == null) {
        return null;
    }

    is = ContentServerHelper.service.findContentStream(ap);
    if (is == null) {
        throw new WTException("Could not get input stream from content file!");
    }

    WTProperties wtproperties = WTProperties.getServerProperties();
    String filepath = wtproperties.getProperty("mitac.report.download.path");

    String path = filepath + fileName; // 抓取檔案存放路徑
    File file = new File(path);
    FileUtils.copyInputStreamToFile(is, file);
    return file;
}

```

### 2. 實行計畫

#### 2-1. 取得 受影響物件

```
public static void test() throws Exception {

    String ecno = "C386210005";
    WTChangeOrder2 eco = ext.generic.util.ChangeUtil.getChangeNoticeByNumber(ecno, true);

    // 受影響物件
    List<Changeable2> beforChangeableList =  ext.generic.util.ChangeUtil.getChangeNoticeItemsBefore(eco);
    for(Changeable2 obj :beforChangeableList) {
      if (obj instanceof ManufacturerPart) {
        ManufacturerPart mpart = (ManufacturerPart) obj;
          System.out.println("mpart:"+mpart.getNumber());
          String makerCode = ext.mitac.integration.erp.CommandUtil.getMakerCode(mpart);
          System.out.println("makerCode1:"+makerCode);
        }else if(obj instanceof WTPart){
          WTPart wtPart = (WTPart) obj;
            System.out.println("WTPart:"+wtPart.getNumber());
        }
    }
}

```

#### 2-2. 取得 產生物件 

```

public static void test() throws Exception {
    String ecno = "C386210005";
    WTChangeOrder2 eco = ext.generic.util.ChangeUtil.getChangeNoticeByNumber(ecno, true);
    // 產生物件、結果物件
    List<Changeable2> afterChangeableList =  ext.generic.util.ChangeUtil.getChangeablesAfter(eco);
    for(Changeable2 obj :afterChangeableList) {
        if (obj instanceof ManufacturerPart) {
            ManufacturerPart mpart = (ManufacturerPart) obj;
            System.out.println("mpart:"+mpart.getNumber());
            String makerCode = ext.mitac.integration.erp.CommandUtil.getMakerCode(mpart);
            System.out.println("makerCode1:"+makerCode);
        }else if(obj instanceof WTPart){
            WTPart wtPart = (WTPart) obj;
            System.out.println("WTPart:"+wtPart.getNumber());
        }
    }
}

```


### 3. 變更 (到時候移到Part 內)

#### 3-1. 取得 源於變更通知

```

wt.fc.QueryResult affectCNs = wt.change2.ChangeHelper2.service
		.getLatestUniqueImplementedChangeOrders((wt.change2.Changeable2) part);

while (affectCNs.hasMoreElements()) {
	wt.change2.WTChangeOrder2 ecn = (wt.change2.WTChangeOrder2) affectCNs.nextElement();			
}

```

