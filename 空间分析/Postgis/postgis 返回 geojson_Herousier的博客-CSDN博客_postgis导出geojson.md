# postgis 返回 geojson_Herousier的博客-CSDN博客_postgis导出geojson
## GeoJSON 特征[集合](https://so.csdn.net/so/search?q=%E9%9B%86%E5%90%88&spm=1001.2101.3001.7020)：

```json
{
   "type": "FeatureCollection",
   "features": [
       {
           "type": "Feature",
           "geometry": {
               "type": "Point",
               "coordinates": [102.0, 0.5]
           },
           "properties": {
               "prop0": "value0"
           }
       },
       {
           "type": "Feature",
           "geometry": {
               "type": "LineString",
               "coordinates": [
                   [102.0, 0.0],
                   [103.0, 1.0],
                   [104.0, 0.0],
                   [105.0, 1.0]
               ]
           },
           "properties": {
               "prop0": "value0",
               "prop1": 0.0
           }
       },
       {
           "type": "Feature",
           "geometry": {
               "type": "Polygon",
               "coordinates": [
                   [
                       [100.0, 0.0],
                       [101.0, 0.0],
                       [101.0, 1.0],
                       [100.0, 1.0],
                       [100.0, 0.0]
                   ]
               ]
           },
           "properties": {
               "prop0": "value0",
               "prop1": {
                   "this": "that"
               }
           }
       }
   ]
}

```

## ST_AsGeoJSON 函数:

```sql
SELECT
	ST_AsGeoJSON(ST_Transform(geom,4326))
FROM
	tb_base_hazard

```

![](https://img-blog.csdnimg.cn/20210701113744340.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhb19mbHk=,size_16,color_FFFFFF,t_70)

可以看出 ST_AsGeoJSON 返回一个 Geometry

## 此为 postgis 3.x 以下方法

```sql
WITH feature AS (
	SELECT
		'Feature' AS "type",
		st_asgeojson ( geom ) :: json AS "geometry",
		(
			SELECT
				json_strip_nulls ( row_to_json ( fields ) )
			FROM
				(
					SELECT
						id, hazard_name
				) as fields
		) AS "properties" 
	FROM
		tb_base_hazard AS h 
	WHERE
		1=1
	),
	features AS ( 
		SELECT 
			'FeatureCollection' AS "type", 
			array_to_json ( ARRAY_AGG ( feature.* ) ) AS "features" 
		FROM feature 
	) 
		SELECT row_to_json ( features.* ) FROM features;

```

## 此为 postgis 3.x 方法 更快更简洁

[官方文档](http://postgis.net/docs/manual-3.1/ST_AsGeoJSON.html)  
![](https://img-blog.csdnimg.cn/20210701114058171.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhb19mbHk=,size_16,color_FFFFFF,t_70)

可见直接返回单个 **feature**

```sql
WITH features AS (
	SELECT
	( 
		SELECT 
			st_asgeojson ( fields.* ) :: json 
		FROM ( 
			SELECT 
				c.ID,c.the_geom 
			) AS fields 
	) AS feature 
	FROM
		cities AS c 
	WHERE
		1 = 1 
	) 
SELECT
	json_build_object ( 
		'type', 'FeatureCollection', 
		'features', json_agg ( features.feature ) 
	) 
FROM
	features;

```

 [https://blog.csdn.net/hao_fly/article/details/118383067](https://blog.csdn.net/hao_fly/article/details/118383067)
