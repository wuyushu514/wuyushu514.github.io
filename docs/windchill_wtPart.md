
## WTPart

### 1. WTPart

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

#### 1-3. 上一版PART


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
