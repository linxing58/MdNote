# shader之渐变长方体实现（threejs） - 莫小龙 - 博客园
**shader 之渐变长方体实现（threejs）**

**效果：** 

![](https://img2020.cnblogs.com/blog/1015208/202101/1015208-20210123112406309-458025041.png)

**原理：** 

1\. 用 varying 将 position（位置）、normal（法线）从顶点着色器传递到片元着色器

2\. 假设长方体高为 40，将 fract(vPosition.y) 设置为 gl_FragColor 的色值可看到 40 行渐变：

![](https://img2020.cnblogs.com/blog/1015208/202101/1015208-20210123112940510-1006347687.png)

3\. 通过除以长方体高可得：float cy = fract(vPosition.y/40.0);

![](https://img2020.cnblogs.com/blog/1015208/202101/1015208-20210123113126167-803857597.png)

4\. 通过位移计算可得颜色从 0 到 1 的渐变：float cy = fract((vPosition.y - 20.0) / 40.0);

 ![](https://img2020.cnblogs.com/blog/1015208/202101/1015208-20210123113232391-2052561962.png)

5\. 通过位移和拉伸可控制取渐变中的一部分效果：float cy = (fract((vPosition.y - 20.0) / 40.0) + 0.7) \* 0.7;

6\. 得到最终效果

**完整着色器代码：** 

![](https://common.cnblogs.com/images/copycode.gif)

                let v=\`    
                        varying vec3 vNormal;
                        varying vec3 vPosition; void main() { //将attributes的normal通过varying赋值给了向量vNormal
                            vNormal = normal;
                            vPosition \= position; //projectionMatrix是投影变换矩阵 modelViewMatrix是相机坐标系的变换矩阵
                            gl\_Position = projectionMatrix \* modelViewMatrix \* vec4( position.x, position.y, position.z, 1.0 );
                        }
                \`

                let f \=\`
                        varying vec3 vNormal;
                        varying vec3 vPosition; void main() { float cy = (fract((vPosition.y - 20.0) / 40.0) + 0.7) \* 0.7; if(vNormal.x==0.0&&vNormal.y==1.0&&vNormal.z==0.0){
                                cy \= 1.0;
                            }
                            gl\_FragColor \= vec4(0.0, cy, cy, 1.0);
                        }
                    \`

![](https://common.cnblogs.com/images/copycode.gif)

合作：@浩

钻研不易，转载请注明出处。。。。。。 
 [https://www.cnblogs.com/s313139232/p/14316836.html](https://www.cnblogs.com/s313139232/p/14316836.html)
