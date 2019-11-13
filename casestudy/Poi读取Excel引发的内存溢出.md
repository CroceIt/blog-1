**ǰ��**  
������������и�����Ŀһֱ�ڴ汨������ʱ�Ļ������ڴ�й©��������Ҫ�������������Ѿ�����Ӱ�����������ˡ�

**����**  
1.dump�ڴ��ļ�  
liunxʹ���������

```
./jmap -dump:format=b,file=heap.hprof pid
```

2.ʹ��Eclipse Memory Analysis���з���

![](https://static.oschina.net/uploads/space/2017/0629/171331_pOtS_159239.jpg)

�쳣���£�

```
t org.apache.poi.xssf.usermodel.XSSFRow.<init>(Lorg/openxmlformats/schemas/spreadsheetml/x2006/main/CTRow;Lorg/apache/poi/xssf/usermodel/XSSFSheet;)V (XSSFRow.java:68)
at org.apache.poi.xssf.usermodel.XSSFSheet.initRows(Lorg/openxmlformats/schemas/spreadsheetml/x2006/main/CTWorksheet;)V (XSSFSheet.java:157)
at org.apache.poi.xssf.usermodel.XSSFSheet.read(Ljava/io/InputStream;)V (XSSFSheet.java:132)
at org.apache.poi.xssf.usermodel.XSSFSheet.onDocumentRead()V (XSSFSheet.java:119)
at org.apache.poi.xssf.usermodel.XSSFWorkbook.onDocumentRead()V (XSSFWorkbook.java:222)
at org.apache.poi.POIXMLDocument.load(Lorg/apache/poi/POIXMLFactory;)V (POIXMLDocument.java:200)
at org.apache.poi.xssf.usermodel.XSSFWorkbook.<init>(Ljava/io/InputStream;)V (XSSFWorkbook.java:179)
```

POI�ڼ���Excel�������ڴ�й©���м䴴���˴����Ķ���ռ���˴������ڴ�

3.�鿴�ϴ���Excel��С  
���鿴���ֺܶ�Excel��С��9M���ļ�

4.�鿴����POI��ȡExcel�ķ�ʽ  
����ʹ�õ����û�ģʽ��������ռ�ô������ڴ棻POI�ṩ��2�ж�ȡExcel��ģʽ���ֱ��ǣ�  
**�û�ģʽ��**Ҳ����poi�µ�usermodel�йذ��������û��Ѻã���ͳһ�Ľӿ���ss���£��������ǰ������ļ���ȡ���ڴ��еģ�  
���ڴ������ݺ������ڴ����������ֻ������������Խ�С�������ݣ�  
**�¼�ģʽ��**��poi�µ�eventusermodel���£������˵ʵ�ֱȽϸ��ӣ������������ٶȿ죬ռ���ڴ��٣�����������������Excel���ݡ�

�����������������ȷ���������ʹ��POI���û�ģʽȥ��ȡExcel���ļ��������ڴ�й©��

**��������**  
����ģ��һ��600kb��С��Excel��test.xlsx�����ֱ�������ģʽ��ȡ��Ȼ��۲��ڴ沨����

1.��Ҫ����Ŀ�maven��

```
<dependencies>
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi-ooxml</artifactId>
        <version>3.6</version>
    </dependency>
    <dependency>
        <groupId>com.syncthemall</groupId>
        <artifactId>boilerpipe</artifactId>
        <version>1.2.1</version>
    </dependency>
</dependencies>
```

2.�û�ģʽ�������£�

```
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
 
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
 
public class UserModel {
 
    public static void main(String[] args) throws InterruptedException {
        try {
            Thread.sleep(5000);
            System.out.println("start read");
            for (int i = 0; i < 100; i++) {
                try {
                    Workbook wb = null;
                    File file = new File("D:/test.xlsx");
                    InputStream fis = new FileInputStream(file);
                    wb = new XSSFWorkbook(fis);
                    Sheet sheet = wb.getSheetAt(0);
                    for (Row row : sheet) {
                        for (Cell cell : row) {
                            System.out.println("row:" + row.getRowNum() + ",cell:" + cell.toString());
                        }
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            Thread.sleep(1000);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

3.�¼�ģʽ�������£�

```
import java.io.InputStream;
 
import org.apache.poi.openxml4j.opc.OPCPackage;
import org.apache.poi.xssf.eventusermodel.XSSFReader;
import org.apache.poi.xssf.model.SharedStringsTable;
import org.apache.poi.xssf.usermodel.XSSFRichTextString;
import org.xml.sax.Attributes;
import org.xml.sax.ContentHandler;
import org.xml.sax.InputSource;
import org.xml.sax.SAXException;
import org.xml.sax.XMLReader;
import org.xml.sax.helpers.DefaultHandler;
import org.xml.sax.helpers.XMLReaderFactory;
 
public class EventModel {
 
    public void processOneSheet(String filename) throws Exception {
        OPCPackage pkg = OPCPackage.open(filename);
        XSSFReader r = new XSSFReader(pkg);
        SharedStringsTable sst = r.getSharedStringsTable();
 
        XMLReader parser = fetchSheetParser(sst);
        InputStream sheet2 = r.getSheet("rId1");
        InputSource sheetSource = new InputSource(sheet2);
        parser.parse(sheetSource);
        sheet2.close();
    }
 
    public XMLReader fetchSheetParser(SharedStringsTable sst) throws SAXException {
        XMLReader parser = XMLReaderFactory.createXMLReader("org.apache.xerces.parsers.SAXParser");
        ContentHandler handler = new SheetHandler(sst);
        parser.setContentHandler(handler);
        return parser;
    }
 
    private static class SheetHandler extends DefaultHandler {
        private SharedStringsTable sst;
        private String lastContents;
        private boolean nextIsString;
 
        private SheetHandler(SharedStringsTable sst) {
            this.sst = sst;
        }
 
        public void startElement(String uri, String localName, String name, Attributes attributes) throws SAXException {
            if (name.equals("c")) {
                System.out.print(attributes.getValue("r") + " - ");
                String cellType = attributes.getValue("t");
                if (cellType != null && cellType.equals("s")) {
                    nextIsString = true;
                } else {
                    nextIsString = false;
                }
            }
            lastContents = "";
        }
 
        public void endElement(String uri, String localName, String name) throws SAXException {
            if (nextIsString) {
                int idx = Integer.parseInt(lastContents);
                lastContents = new XSSFRichTextString(sst.getEntryAt(idx)).toString();
                nextIsString = false;
            }
 
            if (name.equals("v")) {
                System.out.println(lastContents);
            }
        }
 
        public void characters(char[] ch, int start, int length) throws SAXException {
            lastContents += new String(ch, start, length);
        }
    }
 
    public static void main(String[] args) throws Exception {
        Thread.sleep(5000);
        System.out.println("start read");
        for (int i = 0; i < 100; i++) {
            EventModel example = new EventModel();
            example.processOneSheet("D:/test.xlsx");
            Thread.sleep(1000);
        }
    }
}
```

���������Դ��[http://poi.apache.org/spreadsheet/how-to.html#xssf\_sax\_api](http://poi.apache.org/spreadsheet/how-to.html#xssf_sax_api)

4.����VM arguments��-Xms100m -Xmx100m  
UserModel���н��ֱ�ӱ�OutOfMemoryError��������ʾ��

```
Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
    at java.lang.String.substring(String.java:1877)
    at org.apache.poi.ss.util.CellReference.separateRefParts(CellReference.java:353)
    at org.apache.poi.ss.util.CellReference.<init>(CellReference.java:87)
    at org.apache.poi.xssf.usermodel.XSSFCell.<init>(XSSFCell.java:105)
    at org.apache.poi.xssf.usermodel.XSSFRow.<init>(XSSFRow.java:68)
    at org.apache.poi.xssf.usermodel.XSSFSheet.initRows(XSSFSheet.java:157)
    at org.apache.poi.xssf.usermodel.XSSFSheet.read(XSSFSheet.java:132)
    at org.apache.poi.xssf.usermodel.XSSFSheet.onDocumentRead(XSSFSheet.java:119)
    at org.apache.poi.xssf.usermodel.XSSFWorkbook.onDocumentRead(XSSFWorkbook.java:222)
    at org.apache.poi.POIXMLDocument.load(POIXMLDocument.java:200)
    at org.apache.poi.xssf.usermodel.XSSFWorkbook.<init>(XSSFWorkbook.java:179)
    at zh.excelTest.UserModel.main(UserModel.java:23)
```

EventModel�����������У�ʹ��Java VisualVM��ؽ�����£�  
![](https://static.oschina.net/uploads/space/2017/0629/171525_KMbH_159239.jpg)

UserModelģʽ�¶�ȡ600kbExcel�ļ�ֱ���ڴ����������600kbExcel�ļ�ӳ�䵽�ڴ��л���ռ���˲����ڴ棻EventModelģʽ�¿������������С�

5.����VM arguments��-Xms200m -Xmx200m  
UserModel�����������У�ʹ��Java VisualVM��ؽ�����£�

![](https://static.oschina.net/uploads/space/2017/0629/171602_glEi_159239.jpg)

EventModel�����������У�ʹ��Java VisualVM��ؽ�����£�

![](https://static.oschina.net/uploads/space/2017/0629/171624_1j5r_159239.jpg)

UserModelģʽ��EventModelģʽ�������������У����Ǻ�����UserModelģʽ�����ڴ����Ƶ����������cpu��ռ���ϸ��ߡ�

**�ܽ�**  
ͨ���򵥵ķ����Լ�������������ģʽ���бȽϣ����Կ���UserModelģʽ��ʹ�õļ򵥵Ĵ���ʵ���˶�ȡ�������ڶ�ȡ���ļ�ʱCPU���ڴ涼�����룻  
��EventModelģʽ��Ȼ����д�����ȽϷ����������ڶ�ȡ���ļ�ʱCPU���ڴ����ռ�š�