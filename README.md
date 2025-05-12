
npm install xlsx file-saver

import * as XLSX from 'xlsx';
import * as FileSaver from 'file-saver';
import { Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class ExcelExportService {
  exportMultipleSheets(
    dataSheets: { sheetName: string; data: any[]; columnWidths?: number[] }[],
    fileName: string
  ) {
    const workbook = { Sheets: {}, SheetNames: [] };

    dataSheets.forEach(sheet => {
      const header = Object.keys(sheet.data[0] || {});
      const rows = [header, ...sheet.data.map(obj => header.map(key => obj[key]))];
      const worksheet = XLSX.utils.aoa_to_sheet(rows);

      // Apply bold style to header row
      header.forEach((_, i) => {
        const cellRef = XLSX.utils.encode_cell({ c: i, r: 0 });
        if (!worksheet[cellRef]) return;
        worksheet[cellRef].s = {
          font: { bold: true }
        };
      });

      // Optional: set column widths
      if (sheet.columnWidths) {
        worksheet['!cols'] = sheet.columnWidths.map(w => ({ wch: w }));
      }

      workbook.SheetNames.push(sheet.sheetName);
      workbook.Sheets[sheet.sheetName] = worksheet;
    });

    const excelBuffer = XLSX.write(workbook, {
      bookType: 'xlsx',
      type: 'array',
      cellStyles: true
    });

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











============================================\
 constructor(
    private rendererFactory: RendererFactory2,
    private sanitizer: DomSanitizer,
    @Inject(DOCUMENT) private document: Document
  ) {
    this.renderer = rendererFactory.createRenderer(null, null);
  }

  printDiv(divId: string): void {
    const target: HTMLElement | null = this.document.getElementById(divId);

    if (!target) {
      console.error(`Element with id "${divId}" not found`);
      return;
    }

    const rawHtml: string = target.innerHTML;
    const sanitizedHtml: string | null = this.sanitizer.sanitize(SecurityContext.HTML, rawHtml);

    if (!sanitizedHtml) {
      console.error('Sanitized content is empty or unsafe.');
      return;
    }

    const printWindow: Window | null = window.open('', '', 'width=800,height=600');

    if (!printWindow || !printWindow.document) {
      console.error('Unable to open or access print window.');
      return;
    }

    const doc: Document = printWindow.document;

    // Create HTML structure using Renderer2
    const htmlEl: HTMLElement = this.renderer.createElement('html');
    const headEl: HTMLElement = this.renderer.createElement('head');
    const titleEl: HTMLTitleElement = this.renderer.createElement('title');
    const titleText: Text = this.renderer.createText('Print Preview');
    this.renderer.appendChild(titleEl, titleText);
    this.renderer.appendChild(headEl, titleEl);

    const styleEl: HTMLStyleElement = this.renderer.createElement('style');
    const styleText: Text = this.renderer.createText(`
      body { font-family: Arial, sans-serif; padding: 20px; }
      pre { white-space: pre-wrap; word-break: break-word; }
    `);
    this.renderer.appendChild(styleEl, styleText);
    this.renderer.appendChild(headEl, styleEl);

    const bodyEl: HTMLElement = this.renderer.createElement('body');
    const contentContainer: HTMLElement = this.renderer.createElement('div');
    contentContainer.innerHTML = sanitizedHtml; // safe

    this.renderer.appendChild(bodyEl, contentContainer);
    this.renderer.appendChild(htmlEl, headEl);
    this.renderer.appendChild(htmlEl, bodyEl);

    // Write to print window document
    doc.open();
    doc.write('<!DOCTYPE html>');
    doc.write(htmlEl.outerHTML);
    doc.close();

    printWindow.focus();
    printWindow.print();
    printWindow.close();
  }
