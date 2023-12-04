
## Workflow

### 1. 角色

#### 1-1. 於WF新增角色

若須於WF內角色清單內新增自定義角色，需於下面設定檔設定
Windchill/wtCustom/wt/project/RoleRB.rbInfo
Windchill/wtCustom/wt/project/RoleRB_zh_TW.rbInfo

設定值如下，其中order號碼每次跳號64，其中MI_ENGINEER為KEY

```
MI_ENGINEER.value=MI Engineer
MI_ENGINEER.shortDescription=MI Engineer
MI_ENGINEER.order=9424
```


#### 1-2. 增加 成員

於單據中(路由/處理紀錄)角色內增加成員
下面的roleName指的是key
key值可參考\Windchill_11.0\Windchill\wtCustom\wt\project\RoleRB.rbInfo 大小寫敏感

```

WorkflowUtil.addPrincipalToRole(self, roleName, userAd);

public static void addPrincipalToRole(Object processObj, String roleName, String strUserID) throws WTException {
    WTPrincipal principal = (WTPrincipal)wt.org.OrganizationServicesHelper.manager.getPrincipal(strUserID);
    if (processObj == null || roleName == null || roleName.length() == 0 || principal == null)
        return;
    else
        addPrincipalToRole(processObj, roleName, principal);
}

public static void addPrincipalToRole(Object processObj, String roleName, WTPrincipal principal) throws WTException {
    if (processObj == null || roleName == null || roleName.length() == 0 || principal == null)
        return;
    Team team = null;
    WfProcess wfprocess = getProcess(processObj);
    wt.team.TeamReference teamreference = wfprocess.getTeamId();
    team = (Team) teamreference.getObject();
    Role role = Role.toRole(roleName);
        
    //String originalUser = principal.getName();
    //String replaceUser = CommonUtil.mappingTMSUserName(originalUser); //Get mapping user from TMS_USER_MAPPING table.
    //principal = OrganizationServicesHelper.manager.getPrincipal(replaceUser); //replace principal
        
    team.addPrincipal(role, principal);
    //System.out.println("ext.generic.uril.WorkflowUtil.addPrincipalToRole: Replace = " + originalUser + " change to " + replaceUser);
    //System.out.println("........." + principal.getName());
    team = (Team) PersistenceHelper.manager.refresh(team);
    team = (Team) PersistenceHelper.manager.save(team);
    System.out.println("ext.generic.uril.WorkflowUtil.addPrincipalToRole: roleName = " + roleName + "-User=" + principal.getName());
}


```

#### 1-3. 刪除 成員

於單據中(路由/處理紀錄)角色內刪除成員
下面的roleName指的是key

```

TeamUtil.removeUser(pbo, Arrays.asList(PROJECT_LEADER, L9_PM, SALES, PMP_ENGINEER));

public static String removeUser(WTObject pbo, List<String> roleNameList) {
	StringBuilder sb = new StringBuilder();
	for(int i = 0; i<roleNameList.size(); i++){
		sb.append(removeUser(pbo, roleNameList.get(i)));
	}
		
	String msg = sb.toString();
	sb.setLength(0);
	return msg;
}


public static String removeUser(WTObject pbo, String roleName) {
	String msg = "";
	try {
		Team team = ext.mitac.ecm.WorkflowUtil.getTeam(pbo);
		Role role = _Role.toRole(roleName);
		team = ext.mitac.ecm.WorkflowUtil.refreshTeam_FromDB(team);
		Enumeration rolePrincipals = team.getPrincipalTarget(role);
		boolean hasPrincipals = false;
		while (rolePrincipals.hasMoreElements()) {
			WTPrincipalReference prnplRef = (WTPrincipalReference) rolePrincipals.nextElement();
			hasPrincipals = true;
		}
			
		if (hasPrincipals) {
			team.deleteRole(role);
		}
	} catch (Exception e) {
		msg = "removeUser error roleName:"+roleName;
		e.printStackTrace();
	}

	return msg;
}

```


#### 1-4. 取得單據角色成員

```
取得單據中的角色成員

```
public static List getParticipants(ObjectReference self, String rolename) {
    List list = new ArrayList();
    WTPrincipal wtprincipal = null;
    try {
        Role role = Role.toRole(rolename);
        WfProcess pProcess = null;
        Persistable persistable = self.getObject();
        if (persistable instanceof WfActivity)
            pProcess = ((WfActivity) persistable).getParentProcess();
        else
            pProcess = (WfProcess) persistable;
        pProcess = (WfProcess) PersistenceHelper.manager.refresh(pProcess);

        TeamManaged object = (TeamManaged) pProcess;
        Team team = TeamHelper.service.getTeam(object);

        for (Enumeration enum1 = team.getPrincipalTarget(role); enum1.hasMoreElements();) {
            wtprincipal = ((WTPrincipalReference) enum1.nextElement()).getPrincipal();
            System.out.println("wtprincipal.getName() = " + wtprincipal.getName());
            list.add(wtprincipal);
        }

    } catch (WTException ex) {
        ex.printStackTrace();
    }
    return list;
}

```


### 2. 投票

#### 2-1. 取得路由/處理紀錄內指定角色的成員

getRolePrincipal(eco,"QE(PCBA) ENGINEER")
QE(PCBA) ENGINEER為流程狀況中的名稱

```
public static ArrayList<WTPrincipal> getRolePrincipal(WTChangeOrder2 eco,String roleName) throws WTException {
    ArrayList<WTPrincipal> list = new ArrayList<WTPrincipal>();
    wt.team.Team team = (wt.team.Team) eco.getTeamId().getObject();
    Vector<Role> roles = team.getRoles();
    java.util.Enumeration principals;
    try {
      principals = team.getPrincipalTarget(wt.project.Role.toRole(roleName));
      while(principals.hasMoreElements())
      {
        wt.org.WTPrincipalReference principalRef = (wt.org.WTPrincipalReference)principals.nextElement();
        wt.org.WTPrincipal wtprincipal = principalRef.getPrincipal();
        list.add(wtprincipal);
      }
    } catch (WTInvalidParameterException | WTException e) {
      e.printStackTrace();
    } // replace the role KEY appropriately

    return list;
}
```

#### 2-2. 取得路由/處理紀錄內指定角色實際簽核人員

getVotePerson(eco,"QE-MB")
QE-MB為流程狀況中的角色

```
public static Set<String> getVotePerson(WTChangeOrder2 eco,String roleName) throws WTException {
    Set<String> votors = new HashSet<String>();
    WTContainer wtContainer = eco.getContainer();
    QueryResult wfs = WfEngineHelper.service.getAssociatedProcesses(eco, null, WTContainerRef.newWTContainerRef(wtContainer));
        if (wfs.size() > 0){
        while(wfs.hasMoreElements()){
            WfProcess wfprocess = (WfProcess)wfs.nextElement();
            QueryResult qrWfVotingEventAudits = NmWorkflowHelper.service.getVotingEventsForProcess(wfprocess);
            while (qrWfVotingEventAudits != null && qrWfVotingEventAudits.hasMoreElements()) {
                WfVotingEventAudit wfvotingeventaudit = (WfVotingEventAudit) qrWfVotingEventAudits.nextElement();
                WTPrincipalReference wtprincipalreference = wfvotingeventaudit.getAssigneeRef();
                WTPrincipal wtprincipal = null;
                if (wtprincipalreference != null) {
                    wtprincipal = (WTPrincipal) wtprincipalreference.getObject();
                }
                String displayRole = "";
                if(wfvotingeventaudit.getRole()!=null){
                    displayRole = wfvotingeventaudit.getRole().getDisplay(Locale.ENGLISH);
                }
                System.out.println("displayRole:" + displayRole);
                if(roleName.equalsIgnoreCase(displayRole)) {
                    votors.add(wtprincipal.getName());
                }
            }
        }
    }
    return votors;
}
```

#### 2-3. 查詢自動簽核的投票狀況 (SQL)

-- 查詢自動簽核的投票狀況

```
select Wv.*,sign.*
from
  PLM_AUTO_SIGN sign,
  Wfprocess Wp,Wfassignedactivity Wfc,
  Wfassignment Waa
  left join Workitem Wi on Wi.Ida3c4 =  Waa.Ida2a2
  right join (Select Activitykey, Activityname, Ida3a6, Role, Eventlist, Usercomment,Ida2a2 As Wv_Ida2a2 ,Ida3b6  From Wfvotingeventaudit) Wv on Wv.Ida3b6 = Wi.Ida2a2
where Wp.Ida2a2 = Wfc.Ida3parentprocessref and Waa.Ida3a4 = Wfc.Ida2a2
and Wp.ida2a2 = replace(sign.WF_PROCESS_OID,'OR:wt.workflow.engine.WfProcess:','')
and Wfc.Name = sign.WF_ACTIVITYNAME order by sign.trx_id desc;
```

-- 查詢自動簽核的投票狀況 (以前稱作 auto by pass，但因為稽核問題，統一改成 by pass)
```
select ida2a2,Activitykey,Role,Eventlist,USERCOMMENT
from Wfvotingeventaudit
where USERCOMMENT='by pass';
```

#### 2-4. 修改簽核人員的註解 (SQL) 

-- 修改簽核人員的註解

```
update Wfvotingeventaudit set USERCOMMENT='by pass' where ida2a2 = '8437249888' and USERCOMMENT='auto by pass';
select * from Wfvotingeventaudit where ida2a2 = '8437249888';

update Wfvotingeventaudit set USERCOMMENT='by pass' where ida2a2 in (
  select ida2a2 from Wfvotingeventaudit where USERCOMMENT='auto by pass'
);

select ida2a2 from Wfvotingeventaudit where USERCOMMENT='by pass';
```

### 3



### 4