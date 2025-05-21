@ViewChild('textboxRef', { static: true }) textboxRef!: ElementRef;

ngAfterViewInit() {
  const clearIcon = this.textboxRef.nativeElement.querySelector('.clear-icon-container');
  if (clearIcon) {
    clearIcon.setAttribute('matTooltip', 'Clear the field');
  }
}

#textboxRef
