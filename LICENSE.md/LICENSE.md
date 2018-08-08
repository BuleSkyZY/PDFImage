package com.utils;

import org.apache.pdfbox.io.RandomAccessBuffer;
import org.apache.pdfbox.pdfparser.PDFParser;
import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.pdmodel.PDPage;
import org.apache.pdfbox.rendering.PDFRenderer;
import org.apache.pdfbox.text.PDFTextStripper;

import javax.imageio.IIOImage;
import javax.imageio.ImageIO;
import javax.imageio.ImageWriter;
import javax.imageio.stream.ImageOutputStream;
import java.awt.image.BufferedImage;
import java.io.*;
import java.util.Iterator;

public class PDFImage {
    public final  static String  IMG_TYPE_JPG = "jpg";
    public final  static String  IMG_TYPE_PNG = "png";
	public static void main( String[] args ) throws IOException{
       pdf2img(new File("assets", "恒信玺利实业股份有限公司.pdf"), "D:",IMG_TYPE_PNG);
    }


    /**
     * PDF转图片
     * @param file pdf文件的路径
     * @param savePath 图片保存的地址
     * @param imgType 图片保存方式
     */
    public static void pdf2img(File file,String savePath,String imgType){
    	String fileName = file.getName();
        fileName = fileName.substring(0,fileName.lastIndexOf("."));
        InputStream is = null;
        PDDocument PDFDocment = null;
        try {
            is = new BufferedInputStream(new FileInputStream(file));
            //创建PDF文件解析器
            PDFParser parser = new PDFParser(new RandomAccessBuffer(is));
            parser.parse();
            //获取解析后的PDF文档
            PDFDocment = parser.getPDDocument();
        	// 获取PDF文本
            int pageCount = PDFDocment.getNumberOfPages();
			StringWriter sw = new StringWriter();
			PDFTextStripper stripper = new PDFTextStripper();
			stripper.setSortByPosition(true);
			stripper.setStartPage(1);
			stripper.setEndPage(pageCount);
			stripper.writeText(PDFDocment, sw);
			sw.getBuffer().toString();
			String text = stripper.getText(PDFDocment);
			System.out.println(text);
			System.out.println("PDF共"+pageCount+"页");
            PDFRenderer renderer = new PDFRenderer(PDFDocment);
            for (int i = 0; i < pageCount; i++) {
                //构造保存文件名称格式
                String saveFileName = savePath+"\\"+fileName+"-"+i+"."+imgType;
                //获取当前页对象
                PDPage page =  PDFDocment.getPage(i);
                //图片转换
                pdfPage2Img(page,saveFileName,imgType,renderer,i);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            if(PDFDocment != null){
                try {
                	PDFDocment.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * 将PDF单页转换为图片
     * @param page 当页对象
     * @param saveFileName 保存的图片名称
     * @param imgType 保存的图片类型
     * @param renderer 用于获取BufferedImage
     * @param index 页索引
     * @throws IOException
     */
    public static void pdfPage2Img(PDPage page,String saveFileName,String imgType,PDFRenderer renderer,int index) throws IOException{
        //构造图片
        BufferedImage img_temp  = renderer.renderImage(index);
        //设置图片格式
        Iterator<ImageWriter> it = ImageIO.getImageWritersBySuffix(imgType);
        //将文件写出
        ImageWriter writer = (ImageWriter) it.next();
        ImageOutputStream imageout = ImageIO.createImageOutputStream(new FileOutputStream(saveFileName));
        writer.setOutput(imageout);
        writer.write(new IIOImage(img_temp, null, null));
    }
}
