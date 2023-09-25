
## Project

### 1. Project



### 2. Product


#### 2-1. 取得 Product


依照ID取得 Product 

```

public static PDMLinkProduct getProductById(long pdId) throws WTException {
	QuerySpec queryspec = new QuerySpec(PDMLinkProduct.class);
	SearchCondition searchcondition = new SearchCondition(
	    	PDMLinkProduct.class, "thePersistInfo.theObjectIdentifier.id",
		    SearchCondition.EQUAL, pdId);
	queryspec.appendWhere(searchcondition, new int[] { 0 });
	QueryResult queryresult = PersistenceHelper.manager.find(queryspec);

	if (!queryresult.hasMoreElements()) {
		System.out.println("No product found");
	} else {
		return (PDMLinkProduct) queryresult.nextElement();
	}
		
	return null;
}

```

依照 ContainerRef String 取得 Product

```

PDMLinkProduct product = ext.util.ProductUtil.getProductByContainerRef(doc.getContainerReference().toString());

//input:WTDocument doc.getContainerReference().toString()	
public static PDMLinkProduct getProductByContainerRef(String docContainerRef) {
    PDMLinkProduct product = null;
    try {
	   	if(docContainerRef.indexOf("wt.pdmlink.PDMLinkProduct") > -1) {
	   		String[] tempArr = docContainerRef.split(":");
	   		String productId = tempArr.length>1 ? tempArr[1]:"";
	   		if(!StringUtils.isBlank(productId)) {
				product = getProductById(Long.parseLong(productId));
	   		}
	   	}
	} catch (WTException e) {
		e.printStackTrace();
		System.out.println("getProductByContainerRef error");
	}
    return product;
}

```

依照 product name 取得 Product

```

public static PDMLinkProduct getProduct(String pdName) {
	PDMLinkProduct product = null;
	try {
		String productId = getProductId(pdName);
		if(StringUtils.isBlank(productId)) {
			return null;
		}

		product = getProductById(productId);
	} catch (WTException e) {
		e.printStackTrace();
	}

	return product;
}


public static String getProductId(String name) {
	try {
	    List<Map<String, Object>> maps = DBUtil.queryRows("select ida2a2 from PDMLinkProduct where NAMECONTAINERINFO = ?", 
		    	new ArrayList(Arrays.asList(name)), "");
		return DBUtil.getFirstRowValue(maps, "ida2a2");	 
	} catch (Exception exp) {
		exp.printStackTrace();
	}

	return "";
}

```

#### 2-2. 取得 Project code


取得 Product 的 Project code

```

public static String getProjectCode(PDMLinkProduct product) {
	if(Objects.isNull(product))
		return StringUtils.EMPTY;
	try {
		return new IBAUtility(product).getIBAValue(IbaEnum.PROJECT_CODE.getName());
	} catch (WTException e) {
		e.printStackTrace();
		return StringUtils.EMPTY;
	}
}

```

### 3. Model



### 4. Team

此成員代表產品底下成員，並非路由處理紀錄之成員

#### 4-1. get Users by role

根據role id 或 role 撈取產品底下成員

```

public static List<WTPrincipalReference> getUsersByRole(PDMLinkProduct product, Role role) {
		ArrayList<WTPrincipalReference> users = new ArrayList<>();
		try {
			ContainerTeam containerTeam = ContainerTeamHelper.service.getContainerTeam(product);
			return containerTeam.getAllPrincipalsForTarget(role);
		} catch (WTException e) {
			e.printStackTrace();
			System.out.println("getUsersByRole error. product name:"+product.getProductName());
			return users;
		}	
	}
	
public static List<WTPrincipalReference> getUsersByRole(PDMLinkProduct product, String roleName) {
	return getUsersByRole(product, _Role.toRole(roleName));
}

```

#### 4-2. 增加 Users to role


增加 Users to teams

```

public static String addUser(PDMLinkProduct product, String roleName, String acc) {
	String errMsg = "";
	if(null == product)
		return "product is null";
	if(StringUtils.isBlank(roleName) || StringUtils.isBlank(acc))
		return "roleName or acc is null";
	WTUser newUser = PrincipalUtil.getUserByName(acc);
	Role role = _Role.toRole(roleName);
	try {
		ContainerTeamHelper.service.addMember(ContainerTeamHelper.service.getContainerTeam(product), role, newUser);
		PersistenceHelper.manager.save(product);
		PersistenceHelper.manager.refresh(product);
	} catch (WTException e) {
		e.printStackTrace();
	}
	
	return errMsg;
}

```

#### 4-3. 刪除 Users 


```

public static String removeUser(PDMLinkProduct product, String roleName, String acc) {
	String errMsg = "";
	if(null == product)
		return "product is null";
	if(StringUtils.isBlank(roleName) || StringUtils.isBlank(acc))
		return "roleName or acc is null";
	if("wcadmin".equalsIgnoreCase(acc)) {
		acc = "Administrator";
	}
	Role role = _Role.toRole(roleName);
	try {
		List<WTPrincipalReference> users = getUsersByRole(product, role);
		for(WTPrincipalReference user : users) {
			if(acc.equals(user.getIdentity())) {
				ContainerTeamHelper.service.removeMember(ContainerTeamHelper.service.getContainerTeam(product), 
						role, user.getPrincipal());
				PersistenceHelper.manager.save(product);
				PersistenceHelper.manager.refresh(product);
				break;
			}
		}
	} catch (WTException e) {
		e.printStackTrace();
	}
	return errMsg;
}

```

#### 4-4. 是否為 專案團隊

判斷是否為專案團隊

```

public static boolean isTeamMember(PDMLinkProduct product, String acc) {
	if(product==null || StringUtils.isBlank(acc)) {
		return false;
	}
		
	if("wcadmin".equalsIgnoreCase(acc)) {
		acc = "Administrator";
	}
		
	try {
		ContainerTeam team =ContainerTeamHelper.service.getContainerTeam(product);
		HashMap<?, ?> hm= team.getAllMembers();
		for(Object key:hm.keySet()) {
			WTPrincipalReference user = (WTPrincipalReference) key;
			if(acc.equals(user.getName()))
				return true;
		}
	} catch (WTException e) {
		e.printStackTrace();
	}
		
	return false;
}

```