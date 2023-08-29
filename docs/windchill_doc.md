
#### WTDocument

> ##### 1.依照文件編號，取得 WTDocument 物件(最新版本版序)

    public static void test() throws Exception {

        String number = "0000725876";
        WTDocument doc = getLastestWTDocument(number);
        String name = doc.getName();
        String vers = doc.getVersionInfo().getIdentifier().getValue();
        String iter = doc.getIterationInfo().getIdentifier().getValue();
        System.out.println("文件名稱："+ name);
        System.out.println("文件版本："+ vers);
        System.out.println("文件版序："+ iter);
    }

    public static WTDocument getLastestWTDocument(String number) throws Exception{

        WTDocumentMaster master = new WTDocumentMaster();
        QuerySpec lqs = new QuerySpec(WTDocumentMaster.class);
        lqs.appendSearchCondition(new SearchCondition(WTDocumentMaster.class, WTDocumentMaster.NUMBER, SearchCondition.EQUAL, number, false));
        QueryResult results = PersistenceHelper.manager.find(lqs);
        while (results.hasMoreElements()) {
          master = (WTDocumentMaster) results.nextElement();
        }

        QueryResult queryResult = wt.vc.VersionControlHelper.service.allVersionsOf(master);
        WTDocument rc = null;
        while (queryResult.hasMoreElements()) {
          WTDocument obj = ((WTDocument) queryResult.nextElement());
            if (!wt.vc.wip.WorkInProgressHelper.isWorkingCopy(obj)) {
                if (rc == null || obj.getVersionIdentifier().getSeries().greaterThan(rc.getVersionIdentifier().getSeries())) {
                    rc = obj;
                }
            }
        }
        if (rc != null) {
            return (WTDocument) wt.vc.VersionControlHelper.getLatestIteration(rc);
        } else {
            return rc;
        }
    }
> ##### 2.依照文件編號，取得 WTDocument 物件 (所有版本版序)

    public static void test() throws Exception {

        String number = "0000725876";
        List<WTDocument> docList = getWTDocument(number);
    }

    public static List<WTDocument> getWTDocument(String number) throws WTException {
        List<WTDocument> docs = new ArrayList<WTDocument>();
        QuerySpec lqs = new QuerySpec(WTDocument.class);
        lqs.appendSearchCondition(new SearchCondition(WTDocument.class, WTDocument.NUMBER, SearchCondition.LIKE, number, false));
        QueryResult results = PersistenceHelper.manager.find(lqs);
        while (results.hasMoreElements()) {
          WTDocument doc = (WTDocument) results.nextElement();
          docs.add(doc);
        }
        return docs;
    }

> ##### 3.刪除 Part Document 的關聯

      -- 刪除 Part Document
      --1.查 所有關聯的 版本/序
      --select count(*),makerpn,maker_code from (
      select distinct
        mpm.wtpartnumber as makerpn
        ,substr(org.name,1,instr(org.name,'(',1,1)-1) as maker_code
        ,mp.statestate as makerpart_state
        ,mp.modifystampa2
        ,mp.versionida2versioninfo
        ,mp.iterationida2iterationinfo
        ,wdm.wtdocumentnumber as docnumber
        ,wd.versionida2versioninfo
        ,wd.iterationida2iterationinfo
        ,wd.statestate as partdoc_state
        ,lnk.ida2a2
      from manufacturerpart mp, manufacturerpartmaster mpm, WTPartReferenceLink lnk,wtdocument wd, wtdocumentmaster wdm --, v_lastestwtdocument vld,
          ,WTOrganization org --, axlentry ax, wtpart wp, wtpartmaster wpm, v_lastestwtpart vl
      where
        mp.ida3masterreference = mpm.ida2a2 and mp.ida2a2 = lnk.ida3a5 and lnk.ida3b5 = wdm.ida2a2
        --and ax.ida3c4 = wp.ida2a2 and wp.ida2a2 = vl.wp_ida2a2 and vl.wpm_ida2a2 = wpm.ida2a2
        --and ax.ida3a4 = mpm.ida2a2
        and mpm.ida3organizationreference = org.ida2a2
        --and ax.amlpreferencedata in (70,80)
        and wdm.ida2a2 = wd.ida3masterreference
        --and wp.statestate <> 'PHASEOUT' --)
        --and mp.statestate = 'CHANGE_IN_WORK'
        --group by makerpn,maker_code
        --and mpm.wtpartnumber = '2215D6740002'
        and wdm.wtdocumentnumber in ('0000349514','0000355577','0000982683','0000605342')
      order by 1,5,6;

      --2.刪除 WTPartReferenceLink 關聯
      select t.*,t.rowid
      from WTPartReferenceLink t
      where t.ida2a2 in (
        select lnk.ida2a2
        from manufacturerpart mp, manufacturerpartmaster mpm, WTPartReferenceLink lnk,wtdocument wd, wtdocumentmaster wdm --, v_lastestwtdocument vld
        , WTOrganization org --, axlentry ax, wtpart wp, wtpartmaster wpm, v_lastestwtpart vl
        where mp.ida3masterreference = mpm.ida2a2 and mp.ida2a2 = lnk.ida3a5 and lnk.ida3b5 = wdm.ida2a2
        --and ax.ida3c4 = wp.ida2a2 and wp.ida2a2 = vl.wp_ida2a2 and vl.wpm_ida2a2 = wpm.ida2a2
        --and ax.ida3a4 = mpm.ida2a2
        and mpm.ida3organizationreference = org.ida2a2
        --and ax.amlpreferencedata in (70,80)
        and wdm.ida2a2 = wd.ida3masterreference
        --and wp.statestate <> 'PHASEOUT' --)
        --and mp.statestate = 'CHANGE_IN_WORK'
        --group by makerpn,maker_code
        --and mpm.wtpartnumber = 'TPD4S010DQAR'
        and wdm.wtdocumentnumber in ('0000845570','0000846005')
      );

      delete from WTPartReferenceLink t where t.ida2a2 in (
        select distinct lnk.ida2a2
        from manufacturerpart mp, manufacturerpartmaster mpm, WTPartReferenceLink lnk,wtdocument wd, wtdocumentmaster wdm --, v_lastestwtdocument vld
        , WTOrganization org --, axlentry ax, wtpart wp, wtpartmaster wpm, v_lastestwtpart vl
        where mp.ida3masterreference = mpm.ida2a2 and mp.ida2a2 = lnk.ida3a5 and lnk.ida3b5 = wdm.ida2a2
        --and ax.ida3c4 = wp.ida2a2 and wp.ida2a2 = vl.wp_ida2a2 and vl.wpm_ida2a2 = wpm.ida2a2
        --and ax.ida3a4 = mpm.ida2a2
        and mpm.ida3organizationreference = org.ida2a2
        --and ax.amlpreferencedata in (70,80)
        and wdm.ida2a2 = wd.ida3masterreference
        --and wp.statestate <> 'PHASEOUT' --)
        --and mp.statestate = 'CHANGE_IN_WORK'
        --group by makerpn,maker_code
        --and mpm.wtpartnumber = 'TPD4S010DQAR'
        and wdm.wtdocumentnumber in ('0000349514')
      );


##### 4.刪除 Part Document 文件 (yushu版) 	

    --查詢所有與該 Part Document 有關之關聯ID:							
    select ida2a2 from WTPartReferenceLink where ida3b5 in (							
      select ida2a2 from wtdocumentmaster 							
      where wtdocumentnumber in ('0000605342','0000355577','0000349514','0000982683')							
    )							
                  
    --刪除該 Part Document 所有關聯:							
    Delete from WTPartReferenceLink 							
    where ida2a2 in(							
      select ida2a2 from WTPartReferenceLink where ida3b5 in (							
        select ida2a2 from wtdocumentmaster 							
        where wtdocumentnumber in ('0000605342','0000355577','0000349514','0000982683')							
      )							
    )							
           
    --然後到 PLM 頁面上刪除這些文件


#### WTDocument 出庫入庫


```
String docNum = "0001154314";
WTDocument doc = DocumentUtil.getLastestWTDocument(docNum);
Workable work = doc;
Folder folder = WorkInProgressHelper.service.getCheckoutFolder();
CheckoutLink checkoutLink = WorkInProgressHelper.service.checkout(work, folder, "Auto CheckOut by System.");
work = checkoutLink.getWorkingCopy();
work = WorkInProgressHelper.service.checkin(work, "check in msg"); 

```
