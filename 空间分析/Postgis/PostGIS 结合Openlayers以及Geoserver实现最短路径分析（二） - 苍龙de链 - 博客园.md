# PostGIS 结合Openlayers以及Geoserver实现最短路径分析（二） - 苍龙de链 - 博客园
前文讲述了怎么用ArcMap制作了测试数据，并导入了PostGIS，接下来我们需要结合PgRouting插件，对入库的数据再进行一下处理。

0、引入扩展包
-------

postgis

pgrouting

postgis_topology

fuzzystrmatch

1、在pgAdmin中，执行下面的sql语句
----------------------

 --添加起点字段source
ALTER TABLE zy ADD COLUMN source integer;

--添加终点字段target
ALTER table zy add column target integer;

--添加道路权重值字段
ALTER TABLE zy ADD COLUMN length double precision;

--建立拓扑关系，这里会为source和target字段赋值（表明，阈值，要素字段名，主键id）
SELECT pgr_createTopology('zy', 0.00001, 'geom', 'gid');  
_###这里在创建完拓扑，source和target赋值了以后，可以给上一篇文章中创建的点图层预留的Id字段赋值了（参看第5步）_

--为source和target字段创建索引（加快查询速度）
CREATE INDEX source_idx ON zy("source"); CREATE INDEX target_idx ON zy("target");

--添加线段端点坐标
ALTER TABLE ZY ADD COLUMN x1 double precision;        --创建起点经度x1
ALTER TABLE ZY ADD COLUMN y1 double precision;        --创建起点纬度y1
ALTER TABLE ZY ADD COLUMN x2 double precision;        --创建起点经度x2
ALTER TABLE ZY ADD COLUMN y2 double precision;        --创建起点经度y2 --给x1、y1、x2、y2赋值
UPDATE ZY SET x1 =ST\_x(ST\_PointN(geom, 1)); UPDATE ZY SET y1 =ST\_y(ST\_PointN(geom, 1)); UPDATE ZY SET x2 =ST\_x(ST\_PointN(geom, ST_NumPoints(geom))); UPDATE ZY SET y2 =ST\_y(ST\_PointN(geom, ST_NumPoints(geom))); 

--为length赋值（以长度作为权重）
UPDATE zy SET length =st_length(geom);

--将长度值赋给reverse_cost，作为路线选择标准（消耗）
ALTER TABLE zy ADD COLUMN reverse_cost double precision; UPDATE zy SET reverse_cost = st_length(geom);

2、测试数据，如果执行下面的语句有结果，则表示数据预处理成功
------------------------------

--通过起点号、终点号查询最短路径 --source为线表起点字段名称 --target为线表终点字段名称 --起点终点前后顺序无固定要求 --length为长度字段，也可以使用自己的评价体系 --1、9为测试使用起点号\\终点号 --zy表名 --id1经过节点号 --id2经过路网线的gid
SELECT seq, id1 AS node, id2 AS edge, cost FROM pgr_dijkstra(' SELECT  gid AS id,
                         source::integer,
                         target::integer,
                         length::double precision AS cost
                        FROM zy', 1, 4, false, false);

3、创建最短路径查询函数（核心）
----------------

\-\- FUNCTION: public.pgr_shortestpath(character varying, double precision, double precision, double precision, double precision)

\-\- DROP FUNCTION public.pgr_shortestpath(character varying, double precision, double precision, double precision, double precision);

CREATE OR REPLACE FUNCTION public.pgr_shortestpath(  
tbl character varying,  
startx double precision,  
starty double precision,  
endx double precision,  
endy double precision)  
RETURNS geometry  
LANGUAGE 'plpgsql'  
COST 100  
VOLATILE STRICT  
AS $BODY$

declare  
v_startLine geometry;--离起点最近的线  
v_endLine geometry;--离终点最近的线  
v_startTarget integer;--距离起点最近线的终点  
v_startSource integer;  
v_endSource integer;--距离终点最近线的起点  
v_endTarget integer;  
v\_statpoint geometry;--在v\_startLine上距离起点最近的点  
v\_endpoint geometry;--在v\_endLine上距离终点最近的点  
v_res geometry;--最短路径分析结果  
v\_res\_a geometry;  
v\_res\_b geometry;  
v\_res\_c geometry;  
v\_res\_d geometry;  
v\_perStart float;--v\_statpoint在v_res上的百分比  
v\_perEnd float;--v\_endpoint在v_res上的百分比  
v\_shPath\_se geometry;--开始到结束  
v\_shPath\_es geometry;--结束到开始  
v_shPath geometry;--最终结果  
tempnode float;  
begin  
--查询离起点最近的线  
--3857坐标系  
--找起点15米范围内的最近线  
execute 'select geom, source, target from ' ||tbl||  
' where ST\_DWithin(geom,ST\_Geometryfromtext(''point('||startx ||' ' || starty||')'',3857),15)  
order by ST\_Distance(geom,ST\_GeometryFromText(''point('|| startx ||' '|| starty ||')'',3857)) limit 1'  
into v\_startLine, v\_startSource ,v_startTarget;  
--查询离终点最近的线  
--找终点15米范围内的最近线  
execute 'select geom, source, target from ' ||tbl||  
' where ST\_DWithin(geom,ST\_Geometryfromtext(''point('|| endx || ' ' || endy ||')'',3857),15)  
order by ST\_Distance(geom,ST\_GeometryFromText(''point('|| endx ||' ' || endy ||')'',3857)) limit 1'  
into v\_endLine, v\_endSource,v_endTarget;  
--如果没找到最近的线，就返回null  
if (v\_startLine is null) or (v\_endLine is null) then  
return null;  
end if ;

raise notice '%', v_startLine;  
raise notice '%', v_endLine;

select ST\_ClosestPoint(v\_startLine, ST\_Geometryfromtext('point('|| startx ||' ' || starty ||')',3857)) into v\_statpoint;  
select ST\_ClosestPoint(v\_endLine, ST\_GeometryFromText('point('|| endx ||' ' || endy ||')',3857)) into v\_endpoint;  
\-\- ST_Distance  
--从开始的起点到结束的起点最短路径  
execute 'SELECT st\_linemerge(st\_union(b.geom)) ' ||  
'FROM pgr_kdijkstraPath(  
''SELECT gid as id, source, target, length as cost FROM ' || tbl ||''','  
||v\_startSource|| ', ' ||'array\['||v\_endSource||'\] , false, false  
) a, '  
||tbl|| ' b  
WHERE a.id3=b.gid  
GROUP by id1  
ORDER by id1' into v_res ;  
--从开始的终点到结束的起点最短路径  
execute 'SELECT st\_linemerge(st\_union(b.geom)) ' ||  
'FROM pgr_kdijkstraPath(  
''SELECT gid as id, source, target, length as cost FROM ' || tbl ||''','  
||v\_startTarget|| ', ' ||'array\['||v\_endSource||'\] , false, false  
) a, '  
||tbl|| ' b  
WHERE a.id3=b.gid  
GROUP by id1  
ORDER by id1' into v\_res\_b ;  
--从开始的起点到结束的终点最短路径  
execute 'SELECT st\_linemerge(st\_union(b.geom)) ' ||  
'FROM pgr_kdijkstraPath(  
''SELECT gid as id, source, target, length as cost FROM ' || tbl ||''','  
||v\_startSource || ', ' ||'array\['||v\_endTarget||'\] , false, false  
) a, '  
|| tbl || ' b  
WHERE a.id3=b.gid  
GROUP by id1  
ORDER by id1' into v\_res\_c ;  
--从开始的终点到结束的终点最短路径  
execute 'SELECT st\_linemerge(st\_union(b.geom)) ' ||  
'FROM pgr_kdijkstraPath(  
''SELECT gid as id, source, target, length as cost FROM ' || tbl ||''','  
||v\_startTarget || ', ' ||'array\['||v\_endTarget||'\] , false, false  
) a, '  
|| tbl || ' b  
WHERE a.id3=b.gid  
GROUP by id1  
ORDER by id1' into v\_res\_d ;  
if(ST\_Length(v\_res) > ST\_Length(v\_res_b)) then  
v\_res = v\_res_b;  
end if;  
if(ST\_Length(v\_res) > ST\_Length(v\_res_c)) then  
v\_res = v\_res_c;  
end if;  
if(ST\_Length(v\_res) > ST\_Length(v\_res_d)) then  
v\_res = v\_res_d;  
end if;  
--如果找不到最短路径，就返回null  
--if(v_res is null) then  
\-\- return null;  
--end if;  
--将v\_res,v\_startLine,v_endLine进行拼接  
raise notice '%', v_res;  
select ST\_LineMerge(ST\_Union(array\[v\_res,v\_startLine,v\_endLine\])) into v\_res;  
raise notice '%', v_res;  
select ST\_Line\_Locate\_Point(v\_res, v\_statpoint) into v\_perStart;  
select ST\_Line\_Locate\_Point(v\_res, v\_endpoint) into v\_perEnd;  
if(v\_perStart > v\_perEnd) then  
tempnode = v_perStart;  
v\_perStart = v\_perEnd;  
v_perEnd = tempnode;  
end if;  
--截取v_res  
--拼接线  
SELECT ST\_Line\_SubString(v\_res,v\_perStart, v\_perEnd) into v\_shPath;  
raise notice '%', v_shPath;  
return v_shPath;  
end;  
$BODY$;

ALTER FUNCTION public.pgr_shortestpath(character varying, double precision, double precision, double precision, double precision)  
OWNER TO postgres;

 4、测试
-----

在调用函数做测试的时候，遇到ST\_Line\_Locate\_Point时报错：“line\_locate\_point : 1st arg isnt a line”，查资料解释说是函数第一个参数格式应该是LineString，我检查了一下我的v\_res格式，是MultiLineString所以报错。（参考了[https://www.jianshu.com/p/4b9d22406bce](https://www.jianshu.com/p/4b9d22406bce)，在此谢谢作者大大，很详细）

![](https://img2018.cnblogs.com/blog/365445/201909/365445-20190930105056566-981048648.png)

知道原因以后怎么解决呢？还是个头疼的问题。我把查询出来的MultiLineString放到ArcMap中，图形化的显示出来，发现这两段竟然是不连通的两端，最后分析应该是数据不连通导致的，两点间查不到结果，没有最短路径。

在ArcMap中利用网络分析工具(networkAnalyse tool)试了一下也不行，果然是这原因，后来换了两个连通的点查询结果就没问题，所以函数没有问题，结果控制一下就行。

5、至此数据预处理结束了，接下来在下一篇中介绍Geoserver中发布数据并调用
----------------------------------------

**6、后续补充**
----------

在代码中提到做ST_Distance时，分别查询了四次查询：

--从开始的起点到结束的起点最短路径 

--从开始的终点到结束的起点最短路径 

--从开始的起点到结束的终点最短路径 

--从开始的终点到结束的终点最短路径

原来在做的时候不是很理解为什么要做这四次查询，是否多余。后来在做带路径的最短路径查询时，有点心得。

![](https://img2018.cnblogs.com/blog/365445/201910/365445-20191017103211967-233345963.png)

实际查询的是A到C，A到D，B到C，B到D

如果不做这步的话，取任意点做路径，万一取到BC或者BD，其实就变成同一个线段了，没有意义了

\_\_EOF\_\_

作　　者：**[苍龙de链](https://www.cnblogs.com/giser-s)**  
出　　处：[https://www.cnblogs.com/giser-s/p/11611655.html](https://www.cnblogs.com/giser-s/p/11611655.html)  
关于博主：编程路上的小学生，热爱技术，喜欢专研。评论和私信会在第一时间回复。或者[直接私信](http://msg.cnblogs.com/msg/send/giser-s)我。  
版权声明：署名 \- 非商业性使用 \- 禁止演绎，[协议普通文本](https://creativecommons.org/licenses/by-nc-nd/4.0/ "协议普通文本") | [协议法律文本](https://creativecommons.org/licenses/by-nc-nd/4.0/legalcode "协议法律文本")。  
声援博主：如果您觉得文章对您有帮助，可以点击文章右下角**【推荐】** 一下。您的鼓励是博主的最大动力！