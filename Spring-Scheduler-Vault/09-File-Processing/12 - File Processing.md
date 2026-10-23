---
tags: [files, excel, csv, cloud-storage, interview]
---

# File Processing

Related: [[11 - Email Services]] · [[02 - Configuration Classes Overview|03 - Configuration Classes Overview]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
Scheduler jobs often need to **generate reports** (Excel), **parse incoming data** (CSV, uploaded files), and **store/retrieve files** from cloud storage or FTP.

## 🔑 Core Pieces

| Component | Purpose |
|---|---|
| `UploadFileReader` | Reads uploaded files (likely CSV/Excel dropped by partners) |
| `ExcelGenerator` | Builds `.xlsx` report files |
| **Apache POI** (`poi`, `poi-ooxml`) | The library actually used to read/write Excel files programmatically |
| **Apache Commons CSV** | Parses/writes CSV with proper quoting/escaping (safer than manual `String.split(",")`) |
| **Google Cloud Storage** | Where generated reports/files are stored long-term (see `TSLGoogleCloudConfig`) |
| **FTP** (`commons-net`) | Legacy file transfer protocol still used by some partners |

## 🛠️ Excel Generation with Apache POI

```java
public class ExcelGenerator {
    public byte[] generateReport(List<Trip> trips) throws IOException {
        try (Workbook workbook = new XSSFWorkbook();
             ByteArrayOutputStream out = new ByteArrayOutputStream()) {

            Sheet sheet = workbook.createSheet("Trips");
            Row header = sheet.createRow(0);
            header.createCell(0).setCellValue("Trip ID");
            header.createCell(1).setCellValue("Distance");

            int rowNum = 1;
            for (Trip t : trips) {
                Row row = sheet.createRow(rowNum++);
                row.createCell(0).setCellValue(t.getId());
                row.createCell(1).setCellValue(t.getDistance());
            }

            workbook.write(out);
            return out.toByteArray();
        }
    }
}
```
**Why `XSSFWorkbook`?** It handles the modern `.xlsx` (Office Open XML) format; `HSSFWorkbook` is for the legacy `.xls` binary format.

## 🛠️ CSV Parsing with Apache Commons CSV

```java
try (Reader reader = new FileReader(file);
     CSVParser parser = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(reader)) {
    for (CSVRecord record : parser) {
        String tripId = record.get("trip_id");
        // ...
    }
}
```
Why not just `String.split(",")`? Because real CSV can have **quoted fields containing commas or newlines** — a naive split breaks on those; Commons CSV handles RFC-4180 quoting correctly.

## 🛠️ Google Cloud Storage

```java
@Configuration
public class TSLGoogleCloudConfig {
    @Bean
    public Storage googleCloudStorage() throws IOException {
        return StorageOptions.newBuilder()
                .setCredentials(GoogleCredentials.fromStream(new FileInputStream("gcs-key.json")))
                .build()
                .getService();
    }
}

// usage
storage.create(BlobInfo.newBuilder("my-bucket", "reports/trip-report.xlsx").build(), excelBytes);
```

## ❓ Interview Q&A

**Q: Why use Apache POI instead of just writing a CSV and calling it an ".xlsx"?**
A: Real Excel features — multiple sheets, formatting, formulas, cell types, column widths — require the actual OOXML structure that POI builds; a CSV renamed to `.xlsx` isn't a valid Excel file and loses all of Excel's spreadsheet features.

**Q: What's the memory concern with generating very large Excel files via POI, and how do you address it?**
A: `XSSFWorkbook` holds the entire workbook in memory, which can cause `OutOfMemoryError` for huge datasets. Apache POI's **`SXSSFWorkbook`** (streaming variant) flushes rows to disk periodically, keeping memory bounded for large exports.

**Q: Why Commons CSV over manual string splitting?**
A: Correctly handles quoted fields, embedded delimiters/newlines inside quotes, and different line-ending conventions per RFC 4180 — manual splitting silently corrupts data in these edge cases.

**Q: Why might a company still use FTP for some integrations given more modern alternatives?**
A: Some legacy partner systems (e.g., older industrial/logistics systems referenced in this module — `FlyashSchedule`, `NINLDataSchedule`) only support FTP for file exchange, and changing that on the partner's end isn't feasible.
