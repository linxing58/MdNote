# java 实现 图片不规则(多边形)裁剪,绘图,添加水印 - 三人語歌 - 博客园
应业务需求, 需要对图片进行二次处理, 裁剪出不规则图形保存, 原图画不规则图形.

![](https://common.cnblogs.com/images/copycode.gif)

import java.awt.AlphaComposite; import java.awt.BasicStroke; import java.awt.Color; import java.awt.Font; import java.awt.Graphics; import java.awt.Graphics2D; import java.awt.Image; import java.awt.Rectangle; import java.awt.RenderingHints; import java.awt.TexturePaint; import java.awt.geom.AffineTransform; import java.awt.geom.GeneralPath; import java.awt.geom.Rectangle2D; import java.awt.image.BufferedImage; import java.io.\*; import java.util.ArrayList; import java.util.HashMap; import java.util.Iterator; import java.util.List; import java.util.Map; import javax.imageio.ImageIO; import javax.imageio.ImageReadParam; import javax.imageio.ImageReader; import javax.imageio.stream.ImageInputStream; import javax.swing.ImageIcon; public class ImageUtils { /\*\* \* 矩形裁剪 jpg
     \* @param inputFilePath 图片输入路径
     \* @param outFilePath 图片输出路径
     \* @param x 原点 x 坐标
     \* @param y 原点 y 坐标
     \* @param width 宽
     \* @param height 高
     \* @throws IOException \*/
    public static void cut_JPG(String inputFilePath, String outFilePath, int x, int y, int width, int height) throws IOException {
        ImageInputStream imageStream \\= null; try {
            FileInputStream input \\= new FileInputStream(inputFilePath);
            Iterator<ImageReader> readers = ImageIO.getImageReadersByFormatName("jpg");
            ImageReader reader \\= readers.next();
            imageStream \\= ImageIO.createImageInputStream(input);
            reader.setInput(imageStream, true);
            ImageReadParam param \\= reader.getDefaultReadParam();

            Rectangle rect \= new Rectangle(x, y, width, height);
            param.setSourceRegion(rect);
            BufferedImage bi \= reader.read(0, param);
            
            FileOutputStream out \= new FileOutputStream(outFilePath);//输出图片的地址
            ImageIO.write(bi, "jpg", out);
            
            input.close();
            out.close();
        } finally {
            imageStream.close();
        }
    } /\*\* \* 矩形裁剪png
     \* @param inputFilePath 图片输入路径
     \* @param outFilePath 图片输出路径
     \* @param x 原点x坐标
     \* @param y 原点y坐标
     \* @param width 宽
     \* @param height 高
     \* @throws IOException \*/
    public static void cut\_PNG(String inputFilePath, String outFilePath, int x, int y, int width, int height) throws IOException {
        ImageInputStream imageStream \= null; try {
            FileInputStream input \= new FileInputStream(inputFilePath);
            Iterator<ImageReader> readers = ImageIO.getImageReadersByFormatName("png");
            ImageReader reader \= readers.next();
            imageStream \= ImageIO.createImageInputStream(input);
            reader.setInput(imageStream, true);
            ImageReadParam param \= reader.getDefaultReadParam();
            
            Rectangle rect \= new Rectangle(x, y, width, height);
            param.setSourceRegion(rect);
            BufferedImage bi \= reader.read(0, param);
            
            FileOutputStream out \= new FileOutputStream(outFilePath);//输出图片的地址
            ImageIO.write(bi, "png", out);
            input.close();
            out.close();
        } finally {
            imageStream.close();
        }
    } /\*\* \* 矩形裁剪
      \* @param inputFilePath 图片输入路径
      \* @param outFilePath 图片输出路径
      \* @param type 图片类型
      \* @param x 原点x坐标
      \* @param y 原点y坐标
      \* @param width 宽
      \* @param height 高
      \* @throws IOException \*/
    public static void cut\_Image(String inputFilePath, String outFilePath, String type,int x, int y, int width, int height) throws IOException {
        ImageInputStream imageStream \= null; try {
            FileInputStream input \= new FileInputStream(inputFilePath);
            String imageType\=(null\==type||"".equals(type))?"jpg":type;
            Iterator<ImageReader> readers = ImageIO.getImageReadersByFormatName(imageType);
            ImageReader reader \= readers.next();
            imageStream \= ImageIO.createImageInputStream(input);
            reader.setInput(imageStream, true);
            ImageReadParam param \= reader.getDefaultReadParam();
            Rectangle rect \= new Rectangle(x, y, width, height);
            param.setSourceRegion(rect);
            BufferedImage bi \= reader.read(0, param);
            
            FileOutputStream out \= new FileOutputStream(outFilePath);//输出图片的地址

 ImageIO.write(bi, imageType, out);
            input.close();
            out.close();
        } finally {
            imageStream.close();
        }
    } /\*\* \* 图片矩形裁切
     \* @param inputFilePath 图片输入路径
     \* @param outFilePath 图片输出路径
     \* @param x 原点 x 坐标
     \* @param y 原点 y 坐标
     \* @param width 宽
     \* @param height 高 \*/
    public static void cut_Image2(String inputFilePath,String outFilePath,int x, int y, int width, int height) { try {
            File f \\= new File(inputFilePath);
            File t \\= new File(outFilePath); if (t.exists()) {
                t.delete();
            } // 图片输入流
            ImageInputStream input = ImageIO.createImageInputStream(f); // 图片读取器
            Iterator<ImageReader> it = ImageIO.getImageReaders(input); if (it.hasNext()) {
                ImageReader r \\= it.next(); // 设置输入流
                r.setInput(input, true); // 读取参数
                ImageReadParam param = r.getDefaultReadParam(); // 创建要截取的矩形范围
                Rectangle rect = new Rectangle(x, y, width, height); // 设置截取范围参数
 param.setSourceRegion(rect); // 读取截图数据
                BufferedImage bi = r.read(0, param); // 保存图片
                ImageIO.write(bi, "jpg", t);
            }
            input.close();
        } catch (Exception e) {
            e.printStackTrace();
        }

    } /\*\* \* 裁剪(多边形)
     \* @param inputFilePath 图片输入路径
     \* @param outFilePath 图片输出路径
     \* @param x\[\] x轴坐标点数组
     \* @param y\[\] y轴坐标点数组
     \* @param n 坐标点数量
     \* @throws IOException \*/
    public static void cutPolygon\_Image( String inputFilePath,String outFilePath,int x\[\], int y\[\],int n) throws IOException { try {
            BufferedImage image \= ImageIO.read(new File(inputFilePath));
            GeneralPath clip \= new GeneralPath(GeneralPath.WIND\_EVEN\_ODD, n); //int x\[\]={860,650,300,1700}; //int y\[\]={20,20,1000,1000};
            clip.moveTo(x\[0\], y\[0\]); for (int i = 1; i < x.length; i++) {
                clip.lineTo(x\[i\], y\[i\]);
            }
            clip.closePath();
            
            Rectangle bounds \= clip.getBounds(); //BufferedImage img = new BufferedImage(bounds.width, bounds.height, BufferedImage.TYPE\_INT\_BGR);
            BufferedImage img = new BufferedImage(image.getWidth(), image.getHeight(), BufferedImage.TYPE\_INT\_BGR);
            Graphics2D g2d \= img.createGraphics(); //clip.transform(AffineTransform.getTranslateInstance(0, 0));

 g2d.setClip(clip); //g2d.translate(0, 0);
            g2d.drawImage(image, 0, 0, null);
            g2d.dispose();

            FileOutputStream out \= new FileOutputStream(outFilePath);//输出图片的地址
            ImageIO.write(img, "jpg", out);
            out.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    } /\*\* \* 绘图(图片上添加文字)
     \* @param inputFilePath 图片输入路径
     \* @param outFilePath 图片输出路径
     \* @param list 文字内容及坐标
     \* @throws IOException \*/
    public static void drawingText\_Image( String inputFilePath,String outFilePath,List<Map<String,Object>> list) throws IOException { try { /\*List<Map<String, Object>> listMap = new ArrayList<Map<String,Object>>();
            Map<String, Object> map = null;
            
            map = new HashMap<String , Object>();
            map.put("text","文字内容");
            map.put("x", 100);
            map.put("y", 100);
            listMap.add(map);\*/ BufferedImage image \= ImageIO.read(new File(inputFilePath));
            Graphics2D g \=(Graphics2D) image.getGraphics();
            g.setRenderingHint(RenderingHints.KEY\_TEXT\_ANTIALIASING,RenderingHints.VALUE\_TEXT\_ANTIALIAS\_ON);
            g.setColor(Color.YELLOW); //字体颜色
            g.setFont(new Font("微软雅黑", Font.BOLD, 32)); //字体样式
            for (Map<String, Object> map : list) {
                g.drawString((String) map.get("text"), (int)map.get("x"),(int)map.get("y"));
            }
            g.dispose();
            FileOutputStream out \= new FileOutputStream(outFilePath);//输出图片的地址
            ImageIO.write(image, "jpg", out);
            out.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    } /\*\* \* 绘图(画矩形框)
     \* @param inputFilePath 图片输入路径
     \* @param outFilePath 图片输出路径
     \* @param x 原点x坐标
     \* @param y 原点y坐标
     \* @param width 宽
     \* @param height 高 \*/
    public static void drawingRectangular\_Image( String inputFilePath,String outFilePath, int x, int y, int width, int height) throws IOException { try {
            
            BufferedImage image \= ImageIO.read(new File(inputFilePath));
            Graphics2D g \=(Graphics2D) image.getGraphics();
            g.setColor(Color.YELLOW);//画笔颜色
            g.setStroke(new BasicStroke(3.0f));//设置线宽
            g.drawRect(x, y, width, height);//矩形框(原点x坐标，原点y坐标，矩形的长，矩形的宽) //g.fillRect(x, y, width, height);//画矩形着色块 //g.draw3DRect(x, y, width, height,true);//画三维矩形

 g.dispose();
            FileOutputStream out \\= new FileOutputStream(outFilePath);// 输出图片的地址
            ImageIO.write(image, "jpg", out);
            out.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    } /\*\* \* 绘图 (画多边形框)
     \* @param inputFilePath 图片输入路径
     \* @param outFilePath 图片输出路径
     \* @param x\[] x 轴坐标点数组
     \* @param y\[] y 轴坐标点数组
     \* @param n 坐标点数量
     \* @throws IOException \*/
    public static void drawingPolygon_Image( String inputFilePath,String outFilePath, int x\[], int y\[],int n) throws IOException { try {

            BufferedImage image \= ImageIO.read(new File(inputFilePath));
            Graphics2D g \=(Graphics2D) image.getGraphics();
            g.setColor(Color.YELLOW);//画笔颜色
            g.setStroke(new BasicStroke(3.0f));//设置线宽 //int x\[\]={860,650,300,1700}; //int y\[\]={20,20,1000,1000};
            g.drawPolygon(x,y,n);// 画多边形

 g.dispose();
            FileOutputStream out \\= new FileOutputStream(outFilePath);// 输出图片的地址
            ImageIO.write(image, "jpg", out);
            out.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    } /\*\* \* 添加图片水印 
     \* @param iconPath 水印图片路径
     \* @param inputFilePath 源图片路径  
     \* @param outFilePath 图片输出路径  
     \* @param degree 水印图片旋转角度 \*/  
    public static void imageWatermarking_Image(String iconPath, String inputFilePath,String outFilePath, Integer degree) { try {  
            Image srcImg \\= ImageIO.read(new File(inputFilePath));

            BufferedImage buffImg \= new BufferedImage(srcImg.getWidth(null),
                    srcImg.getHeight(null), BufferedImage.TYPE\_INT\_RGB); // 得到画笔对象 // Graphics g= buffImg.getGraphics(); 
            Graphics2D g = buffImg.createGraphics(); // 设置对线段的锯齿状边缘处理 

 g.setRenderingHint(RenderingHints.KEY_INTERPOLATION,
                    RenderingHints.VALUE_INTERPOLATION_BILINEAR);   

            g.drawImage(srcImg.getScaledInstance(srcImg.getWidth(null), srcImg   
                    .getHeight(null), Image.SCALE\_SMOOTH), 0, 0, null); if (degree != null) { // 设置水印旋转 

 g.rotate(Math.toRadians(degree),
                        (double) buffImg.getWidth() / 2,
                        (double) buffImg.getHeight() / 2);
            } // 水印图象的路径 水印一般为 gif 或者 png 的，这样可设置透明度 
            ImageIcon imgIcon = new ImageIcon(iconPath); // 得到 Image 对象。
            Image img = imgIcon.getImage(); float alpha = 0.4f; // 透明度 
 g.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_ATOP,alpha)); // 表示水印图片的位置 
            g.drawImage(img, 15, 30, null);

            g.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC\_OVER));

            g.dispose();

            OutputStream out \= new FileOutputStream(outFilePath); // 生成图片
            ImageIO.write(buffImg, "jpg", out);

            out.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    } /\*\* \* 添加文字水印
      \* @param inputFilePath 源图片路径
      \* @param outFilePath 保存的图片路径
      \* @param waterMarkContent 水印内容
      \* @param degree 旋转角度
      \* @param xmove 水印间距
      \* @param ymove 水印间距 \*/
     public static void textWatermarking\_Image(String inputFilePath,String outFilePath,
             String waterMarkContent,Integer degree , int xmove,int ymove) { try { // 读取原图片信息
             File srcImgFile = new File(inputFilePath);// 得到文件
             Image srcImg = ImageIO.read(srcImgFile);// 文件转化为图片
             int srcImgWidth = srcImg.getWidth(null);// 获取图片的宽
             int srcImgHeight = srcImg.getHeight(null);// 获取图片的高 // 加水印
             BufferedImage bufImg = new BufferedImage(srcImgWidth, srcImgHeight,BufferedImage.TYPE\_INT\_RGB);
             Graphics2D g \= bufImg.createGraphics(); // 设置对线段的锯齿状边缘处理

 g.setRenderingHint(RenderingHints.KEY_INTERPOLATION,RenderingHints.VALUE_INTERPOLATION_BILINEAR); // g.drawImage(srcImg, 0, 0, srcImgWidth, srcImgHeight, null);
 g.drawImage(
                     srcImg.getScaledInstance(srcImg.getWidth(null),
                             srcImg.getHeight(null), Image.SCALE_SMOOTH), 0, 0,null); // 设置水印旋转
             if (null != degree) {
                 g.rotate(Math.toRadians(degree),
                         (double) bufImg.getWidth() / 2,
                         (double) bufImg.getHeight() / 2);
             }
             Font font \\= new Font("宋体", Font.PLAIN, 20);

             g.setColor(new Color(107, 109, 106)); //水印颜色
             g.setFont(font); // 设置字体 // g.setFont(new Font("宋体", Font.PLAIN, srcImg.getWidth(null)/300\*15)); // 设置水印文字透明度 // g.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC\_ATOP,0.5f)); // 设置水印的坐标
             int x = -srcImgWidth / 2; int y = -srcImgHeight / 2; int markWidth = font.getSize() \* getTextLength(waterMarkContent);// 字体长度
             int markHeight = font.getSize();// 字体高度 // BigDecimal number= new BigDecimal(srcImg.getWidth(null)); // BigDecimal olDnumber= new BigDecimal(800); // Integer MOVE = number.divide(olDnumber).intValue()\*150; // 循环添加水印
             while (x < srcImgWidth \* 1.5) {
                 y \= -srcImgHeight / 2; while (y < srcImgHeight \* 1.5) {
                     g.drawString(waterMarkContent, x, y);
                     y += markHeight + ymove;
                 }
                 x += markWidth + xmove;
             }
             g.dispose(); // 输出图片
             FileOutputStream outImgStream = new FileOutputStream(outFilePath);
             ImageIO.write(bufImg, "jpg", outImgStream);
             outImgStream.flush();
             outImgStream.close();
         } catch (Exception e) {
             e.printStackTrace();
         }
     } /\*\* \* 获取文本长度。汉字为1:1，英文和数字为2:1 \*/
     private static int getTextLength(String text) { int length = text.length(); for (int i = 0; i < text.length(); i++) {
             String s \= String.valueOf(text.charAt(i)); if (s.getBytes().length > 1) {
                 length++;
             }
         }
         length \= length % 2 == 0 ? length / 2 : length / 2 + 1; return length;
     } public static void main(String\[\] args) {
        String inputFilePath \= "E:/picOcr/original/20200615000000001.jpg";
        String outFilePath \= "E:/picOcr/interception/20200615000000001.jpg";
        
        String iconPath \= "E:/picOcr/2020592210413248.jpg"; try { //裁剪矩形 //ImageUtils.cut\_JPG(inputFilePath, outFilePath, 400,300,900,750); //ImageUtils.cut\_Image(inputFilePath, outFilePath,"jpg", 400,300,900,750); //ImageUtils.cut\_Image2(inputFilePath, outFilePath, 400,300,900,750); //裁剪多边形
            /\*int x\[\]={860,650,300,1700};
            int y\[\]={20,20,1000,1000};
            ImageUtils.cutPolygon\_Image(inputFilePath, outFilePath,x,y,4);\*/
            
            //绘图多边形
            /\*int x\[\]={860,650,400,300,1700};
            int y\[\]={20,20,500,1000,1000};
            ImageUtils.drawingPolygon\_Image(inputFilePath, outFilePath, x, y, 5);\*/
            
            //绘图矩形 //ImageUtils.drawingRectangular\_Image(inputFilePath, outFilePath, 400, 300, 900,750); //绘图 添加文字
            /\*List<Map<String, Object>> listMap = new ArrayList<Map<String,Object>>();
            Map<String, Object> map = null;
            
            map = new HashMap<String , Object>();
            map.put("text","文字内容");
            map.put("x", 100);
            map.put("y", 100);
            listMap.add(map);
            
            map = new HashMap<String , Object>();
            map.put("text","图片注释");
            map.put("x", 100);
            map.put("y", 140);
            listMap.add(map);
            
            ImageUtils.drawingText\_Image(inputFilePath, outFilePath, listMap);\*/
            
            //添加图片水印 //ImageUtils.imageWatermarking\_Image(iconPath, inputFilePath, outFilePath, -45); //添加文字水印
            ImageUtils.textWatermarking\_Image(inputFilePath, outFilePath, "版权所有", -45, 80, 80);
            
        } catch (Exception e) {
            e.printStackTrace();
        }
        
        
    }

} 

![](https://common.cnblogs.com/images/copycode.gif) 
 [https://www.cnblogs.com/Lixiaogang/p/13157343.html](https://www.cnblogs.com/Lixiaogang/p/13157343.html)
