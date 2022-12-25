# 利用 [jsPDF](https://github.com/parallax/jsPDF) 生成 PDF 檔的方式

## 截圖後放入 PDF 檔
1. 先利用 [html2canvas](https://html2canvas.hertzen.com/) 將畫面所要區域截圖
2. 將截圖放入 jsPDF 所生成的 PDF 物件中存檔
   ```HTML
   <div id="printToPdf">測試文字</div>

   <script>
        let printContent = document.getElementById("printToPDF");
        html2canvas(printContent, {
            allowTaint: false,
            scale: 4 // 截圖的解析度倍數，基準值是 96dpi，愈高倍數所截的圖愈細緻，但是檔案也愈大。
        }).then(canvas => {
            let imageData = canvas.toDataURL('image/png');
            const a4Width = 210; // A4 紙張的寬度
            const a4Height = 297; // A4 紙張的高度
            const margin = 20;
            const imageWidth = a4Width - 2 * margin;
            const pageHeight = a4Height - 2 * margin;
            const imageHeight = (canvas.height * imageWidth) / canvas.width;
            let heightLeft = imageHeight;
            let position = margin;
            heightLeft -= pageHeight;

            let doc = new jsPDF("p", "mm", "a4");
            doc.setFont("NotoSansTC-Regular", "normal").setFontSize(12);
            doc.addImage(imageData, 'PNG', margin, position, imageWidth, pageHeight);
            while (heightLeft >= 0) {
                position = heightLeft - imageHeight;
                doc.addPage();
                doc.addImage(imageData, 'PNG', margin, position, imageWidth, pageHeight);
                heightLeft -= pageHeight;
            }
            doc.save(new Date().toLocaleString() + ".pdf");
        });
   </script>
   ```
   
## 直接使用 jsPDF 將 HTML 轉成 PDF
建立 jsPDF 物件後，把 PDF 檔內容放入 doc.html() 中
   ```JS
   let printContent = document.getElementById("printToPDF");
   let doc = new jsPDF("p", "mm", "a4"); //此為預設規格，不輸入也會得到一樣的物件
   
   doc.setFont("NotoSansTC-Regular", "normal").setFontSize(12);

    let options = {
        callback: function (doc) {
            doc.save(new Date().toLocaleString() + ".pdf");
        },
        margin: [0, 20, 0, 20],
        autoPaging: "text",
        x: 0,
        y: 0,
        maxWidth: 170
    }

   doc.html(printContent, options)
   ```

## 兩者的優點及缺點
||截圖後放入jsPDF物件|從 HTML 轉成 PDF|
|---|---|---|
|優點|版面可以與瀏覽器所看到的一致|應該是比較正確的做法|
|缺點|無法搜尋文字<br>檔案較大<br>如要產生多頁檔案，要重複相同步驟許多次|格式很難調整<br>中文字較難處理<br>中英文字可能會重疊|

## 注意事項
要讓 PDF 檔中出現中文字，要做兩件事：
1. jsPDF 物件要加入字體以及設定字體
   ```HTML
   <script>
       let doc = new jsPDF();
       doc.setFont("NotoSansTC-Regular", "normal");
   </script>
   ```
2. 網頁中也要指定字體
    ```HTML
    <div id=printToPdf style="font-family: NotoSansTC-Regular">
        測試文字
    </div>
    ```


## 參考資料
- [jsPDF Documentation](http://raw.githack.com/MrRio/jsPDF/master/docs/index.html)
- [jsPDF Live Demo](http://raw.githack.com/MrRio/jsPDF/master/index.html)
- [jsPDF upgrade from 1.5.3 to 2.5.1 don't work](https://stackoverflow.com/questions/71820359/jspdf-upgrade-from-1-5-3-to-2-5-1-dont-work/71820447#71820447)
- [How to save a image in multiple pages of pdf using jspdf](https://stackoverflow.com/questions/24069124/how-to-save-a-image-in-multiple-pages-of-pdf-using-jspdf)
- [html2canvas Documentation](https://html2canvas.hertzen.com/documentation)
- [jsPDF AutoTable](https://github.com/JonatanPe/jsPDF-AutoTable)
- [[前端軍火庫]jsPDF - 前端直接產生PDF也沒問題！](https://dotblogs.com.tw/wellwind/2016/12/29/front-end-jspdf)
- [JS小學堂｜HTML to PDF轉存頁面](https://medium.com/anna-hsaio-%E5%89%8D%E7%AB%AF%E9%96%8B%E7%99%BC%E8%A8%98/js%E5%B0%8F%E5%AD%B8%E5%A0%82-html-to-pdf%E8%BD%89%E5%AD%98%E9%A0%81%E9%9D%A2-fa8925b75c3d)