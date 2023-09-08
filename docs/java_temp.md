#### 未分類


##### 未分類

###### 1.Sort

Sort the list based on level and actionName

```
public static void sortChangeList(List<ChangeListUtil> changeListReport) {	
	Collections.sort(changeListReport, new Comparator<ChangeListUtil>() {
	    public int compare(ChangeListUtil o1, ChangeListUtil o2) {
	        String x1 = o1.getLevel() + "";
	        String x2 = o2.getLevel() + "";
	        int sComp = x1.compareTo(x2);
	        if (sComp != 0) {
	            return sComp;
	        }

	        String a1 = o1.getActionName();
	        String a2 = o2.getActionName();
	        return a2.compareTo(a1);
	    }
	});
}

```

