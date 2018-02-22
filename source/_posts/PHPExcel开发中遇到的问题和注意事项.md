---
title: PHPExcel开发中遇到的问题和注意事项
categories: 编程
tags:
  - phpexcel
translate_title: phpexcel-problems-encountered-in-the-development-and-precautions
date: 2016-07-12 11:37:11
---

标签（空格分隔）： PHPExcel
这段时间博主也在用PHPExcel进行项目开发，里面遇到了很多的问题，也总结了很多的经验，这里就记录下来作为自己以后的注意事项

## 一、phpExcel大数据量情况下内存溢出解决
版本：1.7.6+
在不进行特殊设置的情况下，phpExcel将读取的单元格信息保存在内存中，我们可以通过

```php
PHPExcel_Settings::setCacheStorageMethod()
```
来设置不同的缓存方式，已达到降低内存消耗的目的！
### 1、将单元格数据序列化后保存在内存中

```php
PHPExcel_CachedObjectStorageFactory::cache_in_memory_serialized;
```

### 2、将单元格序列化后再进行Gzip压缩，然后保存在内存中
```php
PHPExcel_CachedObjectStorageFactory::cache_in_memory_gzip;
```

### 3、缓存在临时的磁盘文件中，速度可能会慢一些

```php
PHPExcel_CachedObjectStorageFactory::cache_to_discISAM;
```
### 4、保存在php://temp
```php
PHPExcel_CachedObjectStorageFactory::cache_to_phpTemp;
```
### 5、保存在memcache中
```php
PHPExcel_CachedObjectStorageFactory::cache_to_memcache
$cacheMethod = PHPExcel_CachedObjectStorageFactory::cache_to_memcache;
$cacheSettings = array( 'memcacheServer'  => 'localhost',
                        'memcachePort'    => 11211,
                        'cacheTime'       => 600
                      );
PHPExcel_Settings::setCacheStorageMethod($cacheMethod, $cacheSettings);
```
注意是加在new PHPExcel() 前面：如下
```php
require_once APPPATH .'third_party/PHPExcel/PHPExcel.php';
$cacheMethod = PHPExcel_CachedObjectStorageFactory::cache_to_phpTemp;
$cacheSettings = array('memoryCacheSize'=>'16MB');
PHPExcel_Settings::setCacheStorageMethod($cacheMethod, $cacheSettings);
$objPHPExcel = new PHPExcel();
```
以上转自：[这位兄台](http://www.cnblogs.com/myx/archive/2013/05/20/phpExcel-setCache.html)


## 二、phpexcel单元格内换行
我说的这个换行不是字多了，自动换行的那种，是在特定位置添加换行符然后注意两点
```php
$objPHPExcel ->setActiveSheetIndex(0)->setCellValue( 'A4' , “Hello\nWorld”)；
$objPHPExcel ->setActiveSheetIndex(0)->setCellValue( 'A4' , “Hello\nWorld”)；
```
### 一、要有下面的代码配合
```php
$objPHPExcel->getActiveSheet()->getStyle('A4')->getAlignment()->setWrapText(true);
$objPHPExcel->getActiveSheet()->getStyle('A4')->getAlignment()->setWrapText(true);
```
### 二、是要换行的字符串Hello\nWorld外面必须是双引号,另外，网上有朋友说，通过以下代码：
```php
$objPHPExcel->getActiveSheet()->getStyle('A4')->getAlignment()->setWrapText(true);
$objPHPExcel->getActiveSheet()->getStyle('A4')->getAlignment()->setWrapText(true);
```
也可以实现单元格内换行，但是博主在使用后，貌似没有起到作用，只能通过\n的进行换行操作。

### 三、关于合并单元格的填充
先设置左上角的第一个单元格的值，然后通过mergeCells将单元格合并（左上角：右下角）
```php
$objPHPExcel->getActiveSheet()->setCellValue("A2",$intro_title);
$objPHPExcel->getActiveSheet()->mergeCells("A2:Y3");
$objPHPExcel->getActiveSheet()->setCellValue("A2",$intro_title);
$objPHPExcel->getActiveSheet()->mergeCells("A2:Y3");
```
### 四、对于load加载中文文件名模板显示找不到模板的Error解决方法：
博主在开发中，发现存在有汉字的模板文件，在LINUX环境下提示无法加载的问题，实际操作中发现是因为乱码造成PHPExcel无法识别，网上有朋友介绍可以使用iconv函数进行GBK转UTF8格式转换，博主测试后发现仍然无法识别，最后解决方案为：
```php
$filepath = base64_encode($_SERVER['DOCUMENT_ROOT'].__ROOT__."/Uploads/excel/汇总数据输出/花名汇总表.xlsx");
$real_filepath = base64_decode($filepath);
$objPHPExcel = PHPExcel_IOFactory::load($real_filepath);
$filepath = base64_encode($_SERVER['DOCUMENT_ROOT'].__ROOT__."/Uploads/excel/汇总数据输出/花名汇总表.xlsx");
$real_filepath = base64_decode($filepath);
$objPHPExcel = PHPExcel_IOFactory::load($real_filepath);
```
以后如果有更新，就接着这里记录了~~
常用的API记录，摘自朽木博客，大神的转载~~
```php
include 'PHPExcel.php';
include 'PHPExcel/RichText.php'; //用于输出.xls/.pdf的

//创建一个excel
$objPHPExcel = new PHPExcel();

//保存excel
$objWriter = PHPExcel_IOFactory::createWriter($objPHPExcel, 'Excel5');
$objWriter->save("xxx.xlsx");

//输出XLS到浏览器
$objWriter = PHPExcel_IOFactory::createWriter($objPHPExcel, 'Excel5');
header("Pragma: public");
header("Expires: 0″);
header("Cache-Control:must-revalidate, post-check=0, pre-check=0″);
header("Content-Type:application/force-download");
header("Content-Type:application/vnd.ms-execl");
header("Content-Type:application/octet-stream");
header("Content-Type:application/download");
header('Content-Disposition:attachment;filename="resume.xls"');
header("Content-Transfer-Encoding:binary");
$objWriter->save('php://output');

//输出PDF到浏览器
$rendererName = PHPExcel_Settings::PDF_RENDERER_TCPDF;
$rendererLibrary = 'TCPDF';
$rendererLibraryPath = './Lib/tcpdf/';

PHPExcel_Settings::setPdfRenderer(
    $rendererName,
    $rendererLibraryPath
);

$objWriter = PHPExcel_IOFactory::createWriter($objPHPExcel, 'PDF');
$objWriter->setSheetIndex(0);

header("Pragma: public");
header("Expires: 0″);
header("Cache-Control:must-revalidate, post-check=0, pre-check=0″);
header("Content-Type:application/force-download");
header("Content-Type:application/pdf");
header("Content-Type:application/octet-stream");
header("Content-Type:application/download");
header('Content-Disposition:attachment;filename="resume.pdf"');
header("Content-Transfer-Encoding:binary");
$objWriter->save('php://output');
include 'PHPExcel.php';
include 'PHPExcel/RichText.php'; //用于输出.xls/.pdf的

//创建一个excel
$objPHPExcel = new PHPExcel();

//保存excel
$objWriter = PHPExcel_IOFactory::createWriter($objPHPExcel, 'Excel5');
$objWriter->save("xxx.xlsx");

//输出XLS到浏览器
$objWriter = PHPExcel_IOFactory::createWriter($objPHPExcel, 'Excel5');
header("Pragma: public");
header("Expires: 0″);
header("Cache-Control:must-revalidate, post-check=0, pre-check=0″);
header("Content-Type:application/force-download");
header("Content-Type:application/vnd.ms-execl");
header("Content-Type:application/octet-stream");
header("Content-Type:application/download");
header('Content-Disposition:attachment;filename="resume.xls"');
header("Content-Transfer-Encoding:binary");
$objWriter->save('php://output');

//输出PDF到浏览器
$rendererName = PHPExcel_Settings::PDF_RENDERER_TCPDF;
$rendererLibrary = 'TCPDF';
$rendererLibraryPath = './Lib/tcpdf/';

PHPExcel_Settings::setPdfRenderer(
    $rendererName,
    $rendererLibraryPath
);

$objWriter = PHPExcel_IOFactory::createWriter($objPHPExcel, 'PDF');
$objWriter->setSheetIndex(0);

header("Pragma: public");
header("Expires: 0″);
header("Cache-Control:must-revalidate, post-check=0, pre-check=0″);
header("Content-Type:application/force-download");
header("Content-Type:application/pdf");
header("Content-Type:application/octet-stream");
header("Content-Type:application/download");
header('Content-Disposition:attachment;filename="resume.pdf"');
header("Content-Transfer-Encoding:binary");
$objWriter->save('php://output');
设置excel的属性

//创建人
$objPHPExcel->getProperties()->setCreator("Maarten Balliauw");
//最后修改人
$objPHPExcel->getProperties()->setLastModifiedBy("Maarten Balliauw");
//标题
$objPHPExcel->getProperties()->setTitle("Office 2007 XLSX Test Document");
//题目
$objPHPExcel->getProperties()->setSubject("Office 2007 XLSX Test Document");
//描述
$objPHPExcel->getProperties()->setDescription("Test document for Office 2007 XLSX, generated using PHP classes.");
//关键字
$objPHPExcel->getProperties()->setKeywords("office 2007 openxml php");
//种类
$objPHPExcel->getProperties()->setCategory("Test result file");

//创建人
$objPHPExcel->getProperties()->setCreator("Maarten Balliauw");
//最后修改人
$objPHPExcel->getProperties()->setLastModifiedBy("Maarten Balliauw");
//标题
$objPHPExcel->getProperties()->setTitle("Office 2007 XLSX Test Document");
//题目
$objPHPExcel->getProperties()->setSubject("Office 2007 XLSX Test Document");
//描述
$objPHPExcel->getProperties()->setDescription("Test document for Office 2007 XLSX, generated using PHP classes.");
//关键字
$objPHPExcel->getProperties()->setKeywords("office 2007 openxml php");
//种类
$objPHPExcel->getProperties()->setCategory("Test result file");
其他常用设置

//设置默认字体和大小
$objPHPExcel->getDefaultStyle()->getFont()->setName('宋体');
$objPHPExcel->getDefaultStyle()->getFont()->setSize(10);

//设置当前的sheet
$objPHPExcel->setActiveSheetIndex(0);
//设置sheet的name
$objPHPExcel->getActiveSheet()->setTitle('Simple');
//设置单元格的值
$objPHPExcel->getActiveSheet()->setCellValue('A1', 'String');
$objPHPExcel->getActiveSheet()->setCellValue('A2', 12);
$objPHPExcel->getActiveSheet()->setCellValue('A3', true);
$objPHPExcel->getActiveSheet()->setCellValue('C5', '=SUM(C2:C4)');
$objPHPExcel->getActiveSheet()->setCellValue('B8', '=MIN(B2:C5)');
//合并单元格
$objPHPExcel->getActiveSheet()->mergeCells('A18:E22');
//分离单元格
$objPHPExcel->getActiveSheet()->unmergeCells('A28:B28');
//保护cell
$objPHPExcel->getActiveSheet()->getProtection()->setSheet(true);
$objPHPExcel->getActiveSheet()->protectCells('A3:E13', 'PHPExcel');
//设置格式
// Set cell number formats
echo date('H:i:s') . " Set cell number formatsn";
$objPHPExcel->getActiveSheet()->getStyle('E4')->getNumberFormat()->setFormatCode(PHPExcel_Style_NumberFormat::FORMAT_CURRENCY_EUR_SIMPLE);
$objPHPExcel->getActiveSheet()->duplicateStyle( $objPHPExcel->getActiveSheet()->getStyle('E4'), 'E5:E13' );
//设置宽width
// Set column widths
$objPHPExcel->getActiveSheet()->getColumnDimension('B')->setAutoSize(true);
$objPHPExcel->getActiveSheet()->getColumnDimension('D')->setWidth(12);
//设置高度
$objPHPExcel->getActiveSheet()->getRowDimension('6')->setRowHeight(30);
//设置font
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFont()->setName('Candara');
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFont()->setSize(20);
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFont()->setBold(true);
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFont()->setUnderline(PHPExcel_Style_Font::UNDERLINE_SINGLE);
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFont()->getColor()->setARGB(PHPExcel_Style_Color::COLOR_WHITE);
$objPHPExcel->getActiveSheet()->getStyle('E1')->getFont()->getColor()->setARGB(PHPExcel_Style_Color::COLOR_WHITE);
$objPHPExcel->getActiveSheet()->getStyle('D13')->getFont()->setBold(true);
$objPHPExcel->getActiveSheet()->getStyle('E13')->getFont()->setBold(true);
//设置align
$objPHPExcel->getActiveSheet()->getStyle('D11')->getAlignment()->setHorizontal(PHPExcel_Style_Alignment::HORIZONTAL_RIGHT);
$objPHPExcel->getActiveSheet()->getStyle('D12')->getAlignment()->setHorizontal(PHPExcel_Style_Alignment::HORIZONTAL_LEFT);
$objPHPExcel->getActiveSheet()->getStyle('D13')->getAlignment()->setHorizontal(PHPExcel_Style_Alignment::HORIZONTAL_CENTER);
$objPHPExcel->getActiveSheet()->getStyle('A18')->getAlignment()->setHorizontal(PHPExcel_Style_Alignment::HORIZONTAL_JUSTIFY);
//垂直居中
$objPHPExcel->getActiveSheet()->getStyle('A18')->getAlignment()->setVertical(PHPExcel_Style_Alignment::VERTICAL_CENTER);
//设置column的border
$objPHPExcel->getActiveSheet()->getStyle('A4')->getBorders()->getTop()->setBorderStyle(PHPExcel_Style_Border::BORDER_THIN);
$objPHPExcel->getActiveSheet()->getStyle('B4')->getBorders()->getTop()->setBorderStyle(PHPExcel_Style_Border::BORDER_THIN);
$objPHPExcel->getActiveSheet()->getStyle('C4')->getBorders()->getTop()->setBorderStyle(PHPExcel_Style_Border::BORDER_THIN);
$objPHPExcel->getActiveSheet()->getStyle('D4')->getBorders()->getTop()->setBorderStyle(PHPExcel_Style_Border::BORDER_THIN);
$objPHPExcel->getActiveSheet()->getStyle('E4')->getBorders()->getTop()->setBorderStyle(PHPExcel_Style_Border::BORDER_THIN);
//设置border的color
$objPHPExcel->getActiveSheet()->getStyle('D13')->getBorders()->getLeft()->getColor()->setARGB('FF993300');
$objPHPExcel->getActiveSheet()->getStyle('D13')->getBorders()->getTop()->getColor()->setARGB('FF993300');
$objPHPExcel->getActiveSheet()->getStyle('D13')->getBorders()->getBottom()->getColor()->setARGB('FF993300');
$objPHPExcel->getActiveSheet()->getStyle('E13')->getBorders()->getTop()->getColor()->setARGB('FF993300');
$objPHPExcel->getActiveSheet()->getStyle('E13')->getBorders()->getBottom()->getColor()->setARGB('FF993300');
$objPHPExcel->getActiveSheet()->getStyle('E13')->getBorders()->getRight()->getColor()->setARGB('FF993300');
//设置填充颜色
$objPHPExcel->getActiveSheet()->getStyle('A1')->getFill()->setFillType(PHPExcel_Style_Fill::FILL_SOLID);
$objPHPExcel->getActiveSheet()->getStyle('A1')->getFill()->getStartColor()->setARGB('FF808080');
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFill()->setFillType(PHPExcel_Style_Fill::FILL_SOLID);
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFill()->getStartColor()->setARGB('FF808080');
//加图片
$objDrawing = new PHPExcel_Worksheet_Drawing();
$objDrawing->setName('Logo');
$objDrawing->setDescription('Logo');
$objDrawing->setPath('./images/officelogo.jpg');
$objDrawing->setHeight(36);
$objDrawing->setWorksheet($objPHPExcel->getActiveSheet());

$objDrawing = new PHPExcel_Worksheet_Drawing();
$objDrawing->setName('Paid');
$objDrawing->setDescription('Paid');
$objDrawing->setPath('./images/paid.png');
$objDrawing->setCoordinates('B15');
$objDrawing->setOffsetX(110);
$objDrawing->setRotation(25);
$objDrawing->getShadow()->setVisible(true);
$objDrawing->getShadow()->setDirection(45);
$objDrawing->setWorksheet($objPHPExcel->getActiveSheet());
//超链接
$objPHPExcel->getActiveSheet()->getCell('J9')->getHyperlink()->setUrl('http://www.g.cn/');
//在默认sheet后，创建一个worksheet
echo date('H:i:s') . " Create new Worksheet objectn";
$objPHPExcel->createSheet();
$objWriter = PHPExcel_IOFactory::createWriter($objExcel, 'Excel5');
$objWriter-save('php://output');

//设置默认字体和大小
$objPHPExcel->getDefaultStyle()->getFont()->setName('宋体');
$objPHPExcel->getDefaultStyle()->getFont()->setSize(10);

//设置当前的sheet
$objPHPExcel->setActiveSheetIndex(0);
//设置sheet的name
$objPHPExcel->getActiveSheet()->setTitle('Simple');
//设置单元格的值
$objPHPExcel->getActiveSheet()->setCellValue('A1', 'String');
$objPHPExcel->getActiveSheet()->setCellValue('A2', 12);
$objPHPExcel->getActiveSheet()->setCellValue('A3', true);
$objPHPExcel->getActiveSheet()->setCellValue('C5', '=SUM(C2:C4)');
$objPHPExcel->getActiveSheet()->setCellValue('B8', '=MIN(B2:C5)');
//合并单元格
$objPHPExcel->getActiveSheet()->mergeCells('A18:E22');
//分离单元格
$objPHPExcel->getActiveSheet()->unmergeCells('A28:B28');
//保护cell
$objPHPExcel->getActiveSheet()->getProtection()->setSheet(true);
$objPHPExcel->getActiveSheet()->protectCells('A3:E13', 'PHPExcel');
//设置格式
// Set cell number formats
echo date('H:i:s') . " Set cell number formatsn";
$objPHPExcel->getActiveSheet()->getStyle('E4')->getNumberFormat()->setFormatCode(PHPExcel_Style_NumberFormat::FORMAT_CURRENCY_EUR_SIMPLE);
$objPHPExcel->getActiveSheet()->duplicateStyle( $objPHPExcel->getActiveSheet()->getStyle('E4'), 'E5:E13' );
//设置宽width
// Set column widths
$objPHPExcel->getActiveSheet()->getColumnDimension('B')->setAutoSize(true);
$objPHPExcel->getActiveSheet()->getColumnDimension('D')->setWidth(12);
//设置高度
$objPHPExcel->getActiveSheet()->getRowDimension('6')->setRowHeight(30);
//设置font
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFont()->setName('Candara');
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFont()->setSize(20);
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFont()->setBold(true);
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFont()->setUnderline(PHPExcel_Style_Font::UNDERLINE_SINGLE);
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFont()->getColor()->setARGB(PHPExcel_Style_Color::COLOR_WHITE);
$objPHPExcel->getActiveSheet()->getStyle('E1')->getFont()->getColor()->setARGB(PHPExcel_Style_Color::COLOR_WHITE);
$objPHPExcel->getActiveSheet()->getStyle('D13')->getFont()->setBold(true);
$objPHPExcel->getActiveSheet()->getStyle('E13')->getFont()->setBold(true);
//设置align
$objPHPExcel->getActiveSheet()->getStyle('D11')->getAlignment()->setHorizontal(PHPExcel_Style_Alignment::HORIZONTAL_RIGHT);
$objPHPExcel->getActiveSheet()->getStyle('D12')->getAlignment()->setHorizontal(PHPExcel_Style_Alignment::HORIZONTAL_LEFT);
$objPHPExcel->getActiveSheet()->getStyle('D13')->getAlignment()->setHorizontal(PHPExcel_Style_Alignment::HORIZONTAL_CENTER);
$objPHPExcel->getActiveSheet()->getStyle('A18')->getAlignment()->setHorizontal(PHPExcel_Style_Alignment::HORIZONTAL_JUSTIFY);
//垂直居中
$objPHPExcel->getActiveSheet()->getStyle('A18')->getAlignment()->setVertical(PHPExcel_Style_Alignment::VERTICAL_CENTER);
//设置column的border
$objPHPExcel->getActiveSheet()->getStyle('A4')->getBorders()->getTop()->setBorderStyle(PHPExcel_Style_Border::BORDER_THIN);
$objPHPExcel->getActiveSheet()->getStyle('B4')->getBorders()->getTop()->setBorderStyle(PHPExcel_Style_Border::BORDER_THIN);
$objPHPExcel->getActiveSheet()->getStyle('C4')->getBorders()->getTop()->setBorderStyle(PHPExcel_Style_Border::BORDER_THIN);
$objPHPExcel->getActiveSheet()->getStyle('D4')->getBorders()->getTop()->setBorderStyle(PHPExcel_Style_Border::BORDER_THIN);
$objPHPExcel->getActiveSheet()->getStyle('E4')->getBorders()->getTop()->setBorderStyle(PHPExcel_Style_Border::BORDER_THIN);
//设置border的color
$objPHPExcel->getActiveSheet()->getStyle('D13')->getBorders()->getLeft()->getColor()->setARGB('FF993300');
$objPHPExcel->getActiveSheet()->getStyle('D13')->getBorders()->getTop()->getColor()->setARGB('FF993300');
$objPHPExcel->getActiveSheet()->getStyle('D13')->getBorders()->getBottom()->getColor()->setARGB('FF993300');
$objPHPExcel->getActiveSheet()->getStyle('E13')->getBorders()->getTop()->getColor()->setARGB('FF993300');
$objPHPExcel->getActiveSheet()->getStyle('E13')->getBorders()->getBottom()->getColor()->setARGB('FF993300');
$objPHPExcel->getActiveSheet()->getStyle('E13')->getBorders()->getRight()->getColor()->setARGB('FF993300');
//设置填充颜色
$objPHPExcel->getActiveSheet()->getStyle('A1')->getFill()->setFillType(PHPExcel_Style_Fill::FILL_SOLID);
$objPHPExcel->getActiveSheet()->getStyle('A1')->getFill()->getStartColor()->setARGB('FF808080');
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFill()->setFillType(PHPExcel_Style_Fill::FILL_SOLID);
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFill()->getStartColor()->setARGB('FF808080');
//加图片
$objDrawing = new PHPExcel_Worksheet_Drawing();
$objDrawing->setName('Logo');
$objDrawing->setDescription('Logo');
$objDrawing->setPath('./images/officelogo.jpg');
$objDrawing->setHeight(36);
$objDrawing->setWorksheet($objPHPExcel->getActiveSheet());

$objDrawing = new PHPExcel_Worksheet_Drawing();
$objDrawing->setName('Paid');
$objDrawing->setDescription('Paid');
$objDrawing->setPath('./images/paid.png');
$objDrawing->setCoordinates('B15');
$objDrawing->setOffsetX(110);
$objDrawing->setRotation(25);
$objDrawing->getShadow()->setVisible(true);
$objDrawing->getShadow()->setDirection(45);
$objDrawing->setWorksheet($objPHPExcel->getActiveSheet());
//超链接
$objPHPExcel->getActiveSheet()->getCell('J9')->getHyperlink()->setUrl('http://www.g.cn/');
//在默认sheet后，创建一个worksheet
echo date('H:i:s') . " Create new Worksheet objectn";
$objPHPExcel->createSheet();
$objWriter = PHPExcel_IOFactory::createWriter($objExcel, 'Excel5');
$objWriter-save('php://output');
```
tcpdf中文乱码的问题
下载字体包,解压至fonts目录
修改字体为
```php
$pdf->SetFont('droidsansfallback', '', 12);
```
以上的代码基本在博主的开发中，被当成API文档来看了，实用性很强，这里转载过来，也是基于分享精神给需要的朋友来看吧。原文地址：http://www.xiumu.org/technology/phpexcel-use.shtml



