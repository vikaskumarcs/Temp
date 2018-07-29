public class Program
    {



        static void Main(string[] args)
        {
            DataTable dt = new DataTable();
            dt.Columns.Add("ProviderName", typeof(string));
            dt.Columns.Add("ProviderClass", typeof(string));
            dt.Columns.Add("DrugNDC", typeof(string));
            dt.Columns.Add("DrugName", typeof(string));
            dt.Columns.Add("DrugCode", typeof(string));

            // Create a DataRow, add Name and Age data, and add to the DataTable
            DataRow dr = dt.NewRow();
            dr["ProviderName"] = "Provider Name is Class";
            dr["ProviderClass"] = "Provider Class is class";
            dr["DrugNDC"] = "35366454343";
            dr["DrugName"] = "Alferate";
            dr["DrugCode"] = "WRTWYU";
            dt.Rows.Add(dr);

            // Create another DataRow, add Name and Age data, and add to the DataTable
            dr = dt.NewRow();
            dr["ProviderName"] = "Provider Name1";
            dr["ProviderClass"] = "Provider Class1";
            dr["DrugNDC"] = "35366454646";
            dr["DrugName"] = "Zocor";
            dr["DrugCode"] = "ADFJKHK";
            dt.Rows.Add(dr);

            
            var filePath= GeneratePDF(dt, "ExportDrugList");
            Console.ReadKey();
        }

        public class PDFFooter : PdfPageEventHelper
        {
            // write on top of document
            public override void OnOpenDocument(PdfWriter writer, Document document)
            {
                base.OnOpenDocument(writer, document);
                PdfPTable tabFot = new PdfPTable(1);
                tabFot.SpacingAfter = 10F;
                PdfPCell cell;
                tabFot.TotalWidth = 400f;
                cell = new PdfPCell(new Phrase("Header"));
                tabFot.AddCell(cell);
                //tabFot.WriteSelectedRows(0, -1, 150, document.Top, writer.DirectContent);
            }

            // write on start of each page
            public override void OnStartPage(PdfWriter writer, Document document)
            {
                base.OnStartPage(writer, document);
            }

            // write on end of each page
            public override void OnEndPage(PdfWriter writer, Document document)
            {
                base.OnEndPage(writer, document);
                PdfPTable tabFot = new PdfPTable(1);
                PdfPCell cell;
                tabFot.TotalWidth = 400f;
                cell = new PdfPCell(new Phrase("Footer"));
                tabFot.AddCell(cell);
                //tabFot.WriteSelectedRows(0, -1, 150, document.Bottom, writer.DirectContent);
            }

            //write on close of document
            public override void OnCloseDocument(PdfWriter writer, Document document)
            {
                base.OnCloseDocument(writer, document);
            }
        }
        
        private static string GeneratePDF(DataTable dataTable, string fileName)
        {
            string folderPath = @"C:\MyProjects\PdfFiles\";
            var filepath = string.Empty;
            try
            {
                //font
                PdfPTable table = new PdfPTable(dataTable.Columns.Count);
                BaseFont bf = BaseFont.CreateFont(BaseFont.TIMES_ROMAN, BaseFont.CP1252, false);
                var fnt = new Font(bf, 13.0f, 1, BaseColor.BLACK);
                
                //Set columns names in the pdf file
                for (int k = 0; k < dataTable.Columns.Count; k++)
                {
                    PdfPCell cell = new PdfPCell(new Phrase(dataTable.Columns[k].ColumnName, fnt));

                    cell.HorizontalAlignment = PdfPCell.ALIGN_CENTER;
                    cell.VerticalAlignment = PdfPCell.ALIGN_CENTER;
                    //cell.BackgroundColor = BaseColor.GREEN;
                    
                    table.AddCell(cell);
                }
               
                //Add values of DataTable in pdf file
                for (int i = 0; i < dataTable.Rows.Count; i++)
                {
                    for (int j = 0; j < dataTable.Columns.Count; j++)
                    {
                        PdfPCell cellRows = new PdfPCell(new Phrase(dataTable.Rows[i][j].ToString()));
                        if (i % 2 == 0)
                        {
                            cellRows.BackgroundColor = new BaseColor(204, 204, 204);
                        }
                        else {
                            cellRows.BackgroundColor = new BaseColor(255, 255, 255);
                        }
                        Font verdana = FontFactory.GetFont("Verdana", 10, new BaseColor(15, 14, 14));
                        //cellRows.BackgroundColor = new BaseColor(51, 102, 102);
                        //Font RowFont = FontFactory.GetFont(FontFactory.HELVETICA, 12);
                        Chunk chunkRows = new Chunk(dataTable.Rows[i][j].ToString(), verdana);
                        cellRows.AddElement(chunkRows);
                        
                        //Align the cell in the center
                        cellRows.HorizontalAlignment = PdfPCell.ALIGN_CENTER;
                        cellRows.VerticalAlignment = PdfPCell.ALIGN_CENTER;

                        table.AddCell(cellRows);
                    }
                }
                
                //Exporting to PDF
                if (!Directory.Exists(folderPath))
                {
                    Directory.CreateDirectory(folderPath);
                }
                using (FileStream stream = new FileStream(fileName, FileMode.OpenOrCreate))
                {
                    var pdfDoc = new Document();
                    filepath = String.Format("{0}{1}{2}", folderPath, fileName, "_" + DateTime.Now.ToString("ddMMyyHHmmss") + ".pdf");
                    var writer = PdfWriter.GetInstance(pdfDoc, new FileStream(filepath, FileMode.Create));
                    // calling PDFFooter class to Include in document
                    writer.PageEvent = new PDFFooter();
                    pdfDoc.Open();
                    pdfDoc.Add(table);
                    pdfDoc.Close();
                }
            }
            catch (Exception ex)
            {
                throw ex;
            }
            return filepath;
        }
    }
