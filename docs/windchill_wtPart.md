
## WTPart

### 1. 屬性

#### 1-1. 料號與品名


取得料號與品名

```
String number = "5651C5280009";
WTPartMaster master = GenericUtil.getPartMaster(number);
WTPart wtpart = GenericUtil.getLatestVersionPart(master);

String partNumber = wtpart.getNumber();
String desc = wtpart.getName();

System.out.println("料號：" + partNumber);
System.out.println("品名："+desc);

```

#### 1-2. 建立時間 修改者 建立者 修改者

取得建立時間修改者和建立者及修改者

```
wtpart.getCreatorName();
wtpart.getModifierName();
wtpart.getCreateTimestamp();
wtpart.getModifyTimestamp();

```

#### 1-3. 狀態

變更 / 取得 零件狀態

```

String status = "MBOM_RELEASED"; 
LifeCycleHelper.service.setLifeCycleState(wtpart, State.toState(status));

String partStatus = wtpart.getLifeCycleState().toString(); //取得神達零件狀態
System.out.println("partStatus：" + partStatus);

```


### 2. 版本


#### 2-1. 版本 版序


取得版本 版序

```

String verson = wtpart.getVersionInfo().getIdentifier().getValue(); //取得版本
String iteration = wtpart.getIterationInfo().getIdentifier().getValue(); //取得版序
System.out.println("version：" + verson +"."+ iteration);

```

#### 2-2. 上一版 PART


取得上一版PART, 下面函數的input doc可直接丟 wtpart

```

public static RevisionControlled getPreviousVersion(RevisionControlled doc) {
    RevisionControlled temp = null;
    try {
        String source = doc.getVersionIdentifier().getSeries().toString();
        QueryResult qr = VersionControlHelper.service.allVersionsOf(doc);
        while (qr.hasMoreElements()) {
            RevisionControlled p = (RevisionControlled) qr.nextElement();
            String target = p.getVersionIdentifier().getSeries().toString();
            if ((target.length() == source.length()) && target.compareTo(source) < 0) {
                temp = p;
                break;
            } else if (target.length() < source.length()) {
                temp = p;
                break;
            }
        }
    } catch (WTException e) {
        e.printStackTrace();
    }
    return temp;
}

```


#### 2-3. 退版 PART

退一個小版序

```

public static String reversion(String partNum) {
	String errMsg = "";
	WTPart part = null;
	try {
		QueryResult qr = ConfigHelper.service.filteredIterationsOf(GenericUtil.getPartMaster(partNum), new LatestConfigSpec());
        while(qr.hasMoreElements()){
            part = (WTPart)qr.nextElement();
            //String version =  VersionControlHelper.getVersionIdentifier((Versioned) part).getValue();
            //String iteration = VersionControlHelper.getIterationIdentifier((Iterated) part).getValue();
            //System.out.println("***********"+version + "." + iteration);
            ConflictResolution[] resolutions = { new ConflictResolution(VersionControlConflictType.LATEST_ITERATION_DELETE, VersionControlResolutionType.ALLOW_LATEST_ITERATION_DELETE)};
            WTSet toDelete = new WTHashSet();
            toDelete.add(part);
            VersionControlHelper.service.deleteIterations(toDelete, resolutions);
        }
	} catch (Exception e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
		errMsg = "reversion error and partNum:"+partNum;
	}

	return errMsg;
}


```


