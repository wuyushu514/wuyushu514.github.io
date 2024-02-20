## Oracle SQL


### 

#### listagg

```
SELECT DISTINCT loc.ASSEMBLY_ITEM_NUMBER,
  loc.COMPONENT_ITEM_NUMBER,
  loc.QUANTITY,
  (SELECT listagg(loc2.DESIGNATOR, ',') within GROUP (
  ORDER BY loc2.DESIGNATOR DESC)
  FROM V_PARTUSAGELINK_LOC loc2
  WHERE loc2.COMPONENT_ITEM_NUMBER = loc.COMPONENT_ITEM_NUMBER
  AND loc2.ASSEMBLY_ITEM_NUMBER    = loc.ASSEMBLY_ITEM_NUMBER
  ) AS designatorList
FROM V_PARTUSAGELINK_LOC loc
WHERE loc.ASSEMBLY_ITEM_NUMBER = ?
AND loc.COMPONENT_ITEM_NUMBER  = ?

```


#### xmlagg

listagg資料過長時須要改用xmlagg

```
SELECT loc.ASSEMBLY_ITEM_NUMBER,
  loc.COMPONENT_ITEM_NUMBER,
  loc.QUANTITY,
  rtrim(extract(xmlagg(xmlelement(e, loc.DESIGNATOR
  ||',')), '/E/text()').getclobval(), ',') designatorList
FROM V_PARTUSAGELINK_LOC loc
WHERE loc.ASSEMBLY_ITEM_NUMBER = ?
AND loc.COMPONENT_ITEM_NUMBER  = ?
GROUP BY loc.ASSEMBLY_ITEM_NUMBER,
  loc.COMPONENT_ITEM_NUMBER,
  loc.QUANTITY
```

#### UPDATE FROM

參考下面link, 此外效率上第二種比第一種好

```
link :https://blog.csdn.net/wzy0623/article/details/53927128
```



#### 查找有哪些view使用到某table

```
SELECT distinct OWNER, NAME, REFERENCED_NAME FROM all_dependencies 
start with referenced_name = UPPER('table_name')
connect by nocycle prior UPPER(NAME) = UPPER(referenced_name)
```