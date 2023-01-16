# Cesium着色器内部坐标转换 - 彡丨灬麷 - 博客园
1.绘制世界坐标

点着色器

![](https://common.cnblogs.com/images/copycode.gif)

void main()
{
    vec3 p = vec3(-2160065.339080043, 5088795.039405607, 3170378.7353836372);
    vec4 eyep = czm_modelView * vec4(p, 1.0);
    gl_PointSize = 8.0;
    gl_Position = czm_projection * eyep;
}

![](https://common.cnblogs.com/images/copycode.gif)

片元着色器：

void main()
{
    gl_FragColor = vec4(1,0,0,1);
 }

2.经纬度绘制，内部做坐标转换，转换成世界坐标，不知道有没有简单的方法，这里我们这样处理：

![](https://common.cnblogs.com/images/copycode.gif)

 1 float rx = 6378137.0;
 2 float ry = 6378137.0;
 3 float rz = 6356752.3142451793;
 4 
 5 vec3 normalize1(vec3 point){
 6     float magnitude = sqrt(point.x * point.x + point.y * point.y + point.z * point.z); 7     return point.xyz / magnitude; 8 }
 9 
10 vec3 getWorldPosition(float longitude,float latitude,float height) { 11     float cosLatitude = cos(latitude); 12     vec3 scratchN = vec3(cosLatitude * cos(longitude),cosLatitude * sin(longitude),sin(latitude)); 13     scratchN = normalize(scratchN); 14     vec3 scratchK = vec3(rx * rx * scratchN.x, ry * ry * scratchN.y, rz * rz * scratchN.z); 15      float gamma = sqrt(scratchN.x * scratchK.x + scratchN.y * scratchK.y + scratchN.z * scratchK.z); 16     scratchK = scratchK.xyz / gamma; 17     scratchN = scratchN.xyz * height; 18     return scratchK + scratchN; 19 } 20 
21 void main() 22 { 23     vec3 p = getWorldPosition(113.0/180.0\*3.1415, 30.0/180.0\*3.1415, 10.0); 24     vec4 eyep = czm_modelView * vec4(p, 1.0); 25     gl_PointSize = 8.0; 26     gl\_Position = czm\_projection * eyep; 27 }

![](https://common.cnblogs.com/images/copycode.gif)