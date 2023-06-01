---
title: "用java将数据写入到Excel中"
date: 2023-04-20T02:42:17+09:00
draft: false
categories:
- IT
tags:
- java
---



如果要创建三个工作表并将它们写入Excel文件中，可以使用以下代码：

```java
import java.io.FileOutputStream;
import java.io.IOException;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;

public class WriteExcel {
    public static void main(String[] args) {
        HSSFWorkbook workbook = new HSSFWorkbook();
        
        // 第一个工作表
        Sheet sheet1 = workbook.createSheet("Sheet1");
        Row headerRow1 = sheet1.createRow(0);
        String[] headers1 = {"Column 1", "Column 2", "Column 3"};
        int headerCellnum1 = 0;
        for (String header : headers1) {
            Cell headerCell = headerRow1.createCell(headerCellnum1++);
            headerCell.setCellValue(header);
        }
        String[][] data1 = {{"1", "2", "3"}, {"4", "5", "6"}, {"7", "8", "9"}};
        int rownum1 = 1;
        for (String[] rowData : data1) {
            Row row = sheet1.createRow(rownum1++);
            int cellnum = 0;
            for (String cellData : rowData) {
                Cell cell = row.createCell(cellnum++);
                cell.setCellValue(cellData);
            }
        }
        
        // 第二个工作表
        Sheet sheet2 = workbook.createSheet("Sheet2");
        Row headerRow2 = sheet2.createRow(0);
        String[] headers2 = {"Column A", "Column B", "Column C"};
        int headerCellnum2 = 0;
        for (String header : headers2) {
            Cell headerCell = headerRow2.createCell(headerCellnum2++);
            headerCell.setCellValue(header);
        }
        String[][] data2 = {{"A1", "B1", "C1"}, {"A2", "B2", "C2"}, {"A3", "B3", "C3"}};
        int rownum2 = 1;
        for (String[] rowData : data2) {
            Row row = sheet2.createRow(rownum2++);
            int cellnum = 0;
            for (String cellData : rowData) {
                Cell cell = row.createCell(cellnum++);
                cell.setCellValue(cellData);
            }
        }
        
        // 第三个工作表
        Sheet sheet3 = workbook.createSheet("Sheet3");
        Row headerRow3 = sheet3.createRow(0);
        String[] headers3 = {"Name", "Age", "Gender"};
        int headerCellnum3 = 0;
        for (String header : headers3) {
            Cell headerCell = headerRow3.createCell(headerCellnum3++);
            headerCell.setCellValue(header);
        }
        String[][] data3 = {{"Alice", "30", "Female"}, {"Bob", "25", "Male"}, {"Charlie", "40", "Male"}};
        int rownum3 = 1;
        for (String[] rowData : data3) {
            Row row = sheet3.createRow(rownum3++);
            int cellnum = 0;
            for (String cellData : rowData) {
                Cell cell = row.createCell(cellnum++);
                cell.setCellValue(cellData);
            }
        }
        
        // 写入文件
        try {
            FileOutputStream outputStream = new FileOutputStream("output.xls");
            workbook.write(outputStream);
            workbook.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

这个代码使用HSSFWorkbook创建一个新的Excel文件，并创建一个名为"Sheet1"的工作表。然后，它在第一行插入表头(如果不需要表头也可不加)，然后在接下来的几行中插入数据。最后，它使用FileOutputStream将Excel文件写入磁盘上的文件"output.xls"中。需要注意的是，如果要运行这个代码，需要将Apache POI库添加到项目依赖中，可以在pom.xml文件中添加以下依赖：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>4.1.2</version>
</dependency>

<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>4.1.2</version>
</dependency>
```

