dtccEmailValidator(): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const value = control.value;
    if (value && !value.toLowerCase().endsWith('@dtcc.com')) {
      return { invalidDomain: 'Email must end with @dtcc.com' };
    }
    return null;
  };
}
this.dtccEmailValidator()
