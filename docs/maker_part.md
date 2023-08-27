---
title: "2-1.Maker Part"
description: "Maker Part"
lead: "Maker Code、Maker PN、Document"
date: 2020-11-12T13:26:54+01:00
lastmod: 2020-11-12T13:26:54+01:00
draft: false
images: []
menu:
docs:
parent: "help"
weight: 610
toc: true
---------

> ###### 依據 Maker PN 與 Maker Code 取得 製造商零件
  ![RUNOOB 图标](/images/maker_part.png)

    public static void test() throws Exception {
        String makerPN = "NXH25.000AC12F-KAB4-P2";
        String makerCode = "M0088";
        ManufacturerPart mPart = getManufacturerPart(makerPN,makerCode);

    }

    public static ManufacturerPart getManufacturerPart(String makerPN,String makerCode) throws Exception {
        ManufacturerPart part = null;
        QuerySpec qs = new QuerySpec(ManufacturerPart.class);
        qs.appendWhere(new SearchCondition(ManufacturerPart.class,
            ManufacturerPart.NUMBER, SearchCondition.EQUAL,
            makerPN), new int[]{0});
        qs.appendAnd();
        qs.appendWhere(new SearchCondition(ManufacturerPart.class,
            ManufacturerPart.LATEST_ITERATION, SearchCondition.IS_TRUE), new int[]{0});
        QueryResult qr = PersistenceHelper.manager.find(qs);
        if(qr!=null){
        while (qr.hasMoreElements()){
          ManufacturerPart tempPart = (ManufacturerPart)qr.nextElement();
            if(tempPart.getOrganizationName().toLowerCase().indexOf(makerCode.toLowerCase())!=-1){
              part = tempPart;
              break;
            }
          }
        }
        return part;
    }

> ###### 依據製造商零件，取得文件
  ![RUNOOB 图标](/images/maker_part_doc.png)

    public static void test() throws Exception {
        String makerPN = "NXH25.000AC12F-KAB4-P2";
        String makerCode = "M0088";
        ManufacturerPart mPart = getManufacturerPart(makerPN,makerCode);
        List<WTDocumentMaster> list = getWTDocumentMasterList(mPart);
        for(WTDocumentMaster mdoc:list) {
          List<WTDocument> docList = getWTDocument(mdoc.getNumber());
          for(WTDocument doc:docList) {
            String docNumber = doc.getNumber();
            String ver = doc.getVersionInfo().getIdentifier().getValue();
            String iter = doc.getIterationInfo().getIdentifier().getValue();
            String makerCode = (String)ext.mitac.report.util.IBAValueHelper.getIBAValue(doc, "MakerCode");
            System.out.println("文件編號：" + docNumber);
            System.out.println("文件版本：" + ver + "." + iter);
            System.out.println("文件Maker Code 屬性：" + makerCode);
          }
        }
    }

    public static List<WTDocumentMaster> getWTDocumentMasterList(ManufacturerPart mPart) throws Exception {
      ArrayList<WTDocumentMaster> list = new ArrayList<WTDocumentMaster>();
      QueryResult qr = WTPartHelper.service.getReferencesWTDocumentMasters(mPart);
        while (qr.hasMoreElements())
        {
            Object tempObj = qr.nextElement();
            if (tempObj instanceof WTDocumentMaster)
            {
                WTDocumentMaster wtDocumentMaster = (WTDocumentMaster) tempObj;
                list.add(wtDocumentMaster);
            }
        }
        return list;
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
