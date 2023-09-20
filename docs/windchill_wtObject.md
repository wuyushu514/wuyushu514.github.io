
## WTObject

### 1. IBA

#### 1-1. 取得IBA屬性


取得IBA屬性

```
ext.generic.util.IBAUtility iba = new ext.generic.util.IBAUtility((wt.doc.WTDocument) primaryBusinessObject);
String ibaKey = "applicant";
String ibaValue = iba.getIBAValue(ibaKey);

String projectCode = new IBAUtility(product).getIBAValue(IbaEnum.PROJECT_CODE.getName());

```


#### 1-2. 更新IBA屬性


```

public static void updateIba(WTPart part, String key, String value) throws WTException, WTPropertyVetoException, RemoteException{
		WTPart checkOutPart = (WTPart)IBAUtility.checkOut(part);
        IBAUtility iba = new IBAUtility(checkOutPart);
        iba.setIBAValue(key, value);
        iba.updateAttributeContainer(checkOutPart);
        PersistenceHelper.manager.save(checkOutPart);
        PersistenceHelper.manager.refresh(checkOutPart);
        if(!WorkInProgressHelper.isCheckedOut(part)) {
        	  IBAUtility.checkIn(part,"update "+key+" to " + value);
    }
}

```

#### 1-3. 取得/設置IBA屬性 for 多重值

```

public static void test() throws Exception {
    //當IBA屬性為多重值時，也就是該屬性於PLM管理類型內條件約束無單一值的條件限制時為多重值
    //取得IBA屬性
    String modelCode = "8055";
    MiTACModel model = modelListUtil.getModel(modelCode);
    String ibaKey = "modelAttr2";
    IBAUtility iba = new IBAUtility(model);
    Vector vec = iba.getIBAValues(ibaKey);
    System.out.println("ibaKey:"+ibaKey);
    for(int i =0; i<vec.size(); i++) 
    	System.out.println("ibaValues i:"+vec.get(i));

    //設置IBA屬性
    Vector vecNew = new Vector();
    vecNew.add("441N41400133");
    vecNew.add("441N64600005");
    model = (MiTACModel) modelListUtil.checkoutObject(model);
    iba.setIBAValues(ibaKey, vecNew);
    iba.updateAttributeContainer(model);
    PersistenceHelper.manager.save(model);
    PersistenceHelper.manager.refresh(model);
    WorkInProgressHelper.service.checkin(model,"");
}

```

#### 1-4. 查詢/修改ibaKey值 SQL


```

--取得指定料號的相關IBA屬性值(middlePf連線)
--除了 MV_STRINGVALUE_PART 還有 MV_UnitVALUE_PART 和 MV_TIMESTAMPVALUE_PART
select DISPLAYNAME, VALUE from MV_STRINGVALUE_PART 
where IDA3A4 = (
    select wp_ida2a2 from mv_part where WTPARTNUMBER= '284260000006'
)   

--利用Model Desc查詢實際上的ibaKey值
select distinct 
    sd.IDA2A2 -- stringvalue 內的 ida3a6值
    , sd.DESCRIPTION -- 頁面上的IBA欄位名稱
    , sd.NAME -- ibaKey
from stringvalue sv, stringdefinition sd
where sv.ida3a6 = sd.ida2a2
    and sd.DESCRIPTION = 'Model Description'
order by sd.IDA2A2

--取得指定料號最新版本之IBA屬性string類別的ibaKey和IbaValue
select distinct 
    sd.IDA2A2 -- stringvalue 內的 ida3a6值
    , sd.DESCRIPTION -- 頁面上的IBA欄位名稱
    , sd.NAME -- ibaKey
    , sv.value2 -- value值
from stringvalue sv, stringdefinition sd
where sv.ida3a6 = sd.ida2a2
    and sv.ida3a4 = (
        select distinct WP_IDA2A2 from V_LASTESTWTPART vl, wtpart wp, wtpartmaster wpm 
        where wpm.IDA2A2 = wp.IDA3MASTERREFERENCE
            and wpm.IDA2A2 = vl.WPM_IDA2A2
            and wpm.WTPARTNUMBER = '284260000006'
    )
order by sd.IDA2A2

--取得指定料號指定IBA屬性值
select VALUE2 from stringvalue 
where ida3a4 = (
    select distinct WP_IDA2A2 from V_LASTESTWTPART vl, wtpart wp, wtpartmaster wpm 
    where wpm.IDA2A2 = wp.IDA3MASTERREFERENCE
        and wpm.IDA2A2 = vl.WPM_IDA2A2
        and wpm.WTPARTNUMBER = '284260000006'
) and ida3a6 = (
    select IDA2A2 from stringdefinition where DESCRIPTION = 'SBU Applicant'
)

--修改指定model指定ibaKey之value值
update stringvalue 
set value2 = 'model desc change test', value = UPPER('model desc change test')
where ida2a2 = (
  select ida2a2 from stringvalue 
  where ida3a4 = (
    select ida2a2 from mitacmodel
    where modelcode = '8055'
    and LATESTITERATIONINFO = '1'
  ) and ida3a6 = (
    select IDA2A2 from stringdefinition where DESCRIPTION = 'Model Description'
  )
)

```


### 2.CheckInOut

不適用於單據類

```
public static void checkInOutTest() throws Exception {
	  String partNum ="227C52800015";
    WTPart part = GenericUtil.getLatestVersionPart(GenericUtil.getPartMaster(partNum));
    Workable copy = (Workable) part;
		GenericUtil.checkInOut(copy, false); //出庫
		//將副本轉型並作處理
		WTPart newPart = (WTPart) copy;
		newPart.setName("newName");
		GenericUtil.checkInOut((Workable) newPart, true); //入庫
}

public static void undoTest() throws Exception {
    String partNum ="227C52800015";
    WTPart part = GenericUtil.getLatestVersionPart(GenericUtil.getPartMaster(partNum));
    Workable copy = (Workable) part;
		GenericUtil.checkInOut(copy, false); //出庫
		//將副本轉型並作處理
		WTPart newPart = (WTPart) copy;
		newPart.setName("newName");
    WorkInProgressHelper.service.undoCheckout((Workable) newPart); //恢復出庫
}

```