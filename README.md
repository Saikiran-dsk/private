
npm install xlsx file-saver


import * as XLSX from 'xlsx';
import * as FileSaver from 'file-saver';
import { Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class ExcelExportService {
  exportMultipleSheets(dataSheets, fileName) {
    const workbook = { Sheets: {}, SheetNames: [] };

    dataSheets.forEach(sheet => {
      const worksheet = XLSX.utils.json_to_sheet(sheet.data);
      workbook.SheetNames.push(sheet.sheetName);
      workbook.Sheets[sheet.sheetName] = worksheet;
    });

    const excelBuffer = XLSX.write(workbook, { bookType: 'xlsx', type: 'array' });
    const blob = new Blob([excelBuffer], { type: 'application/octet-stream' });
    FileSaver.saveAs(blob, `${fileName}_${Date.now()}.xlsx`);
  }
}


import { Component } from '@angular/core';
import { ExcelExportService } from './excel-export.service';

@Component({
  selector: 'app-my-component',
  template: `<button (click)="downloadExcel()">Download Excel</button>`
})
export class MyComponent {
  constructor(private excelService: ExcelExportService) {}

  downloadExcel() {
    const users = [
      { Name: 'Alice', Age: 25 },
      { Name: 'Bob', Age: 30 }
    ];

    const products = [
      { Product: 'Laptop', Price: 1000 },
      { Product: 'Phone', Price: 500 }
    ];

    const dataSheets = [
      { sheetName: 'Users', data: users },
      { sheetName: 'Products', data: products }
    ];

    this.excelService.exportMultipleSheets(dataSheets, 'MyReport');
  }
}
