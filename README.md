# BatteryMonitor
RCC->AHB2ENR |=  ( RCC_AHB2ENR_ADCEN );
RCC->CCIPR   &= ~( RCC_CCIPR_ADCSEL );
RCC->CCIPR   |=  ( 3 << RCC_CCIPR_ADCSEL_Pos ); 

//to read ADC channel 6 from A1
GPIOA->OTYPER       &= ~( 0x1 << 1 );
GPIOA->PUPDR        &= ~( 0x3 << ( 1 * 2 ) );
GPIOA->OSPEEDR      &= ~( 0x3 << ( 1 * 2 ) );
GPIOA->MODER        &= ~( 0x3 << ( 1 * 2 ) );
GPIOA->MODER        |=  ( 0x3 << ( 1 * 2 ) );

// Simple way to do a busy-loop delay with GCC.
void __attribute__( ( optimize( "O0" ) ) ) delay_cycles( uint32_t cyc ) {
  for ( uint32_t d_i = 0; d_i < cyc; ++d_i ) { asm( "NOP" ); }
}
int main( void ) {
  // ( ... )
  // Bring the ADC out of 'deep power-down' mode.
  ADC1->CR    &= ~( ADC_CR_DEEPPWD );
  // Enable the ADC voltage regulator.
  ADC1->CR    |=  ( ADC_CR_ADVREGEN );
  // Delay for a handful of microseconds.
  delay_cycles( 100 );
  // Calibrate the ADC if necessary.
  if ( perform_calibration ) {
    ADC1->CR  |=  ( ADC_CR_ADCAL );
    while ( ADC1->CR & ADC_CR_ADCAL ) {};
  }
  // 
  
 // First, set the number of channels to read during each sequence.
// (# of channels = L + 1, so set L to 0)
ADC1->SQR1  &= ~( ADC_SQR1_L );
// Configure the first (and only) step in the sequence to read channel 6.
ADC1->SQR1  &= ~( 0x1F << 6 );
ADC1->SQR1  |=  ( 6 << 6 );
// Configure the sampling time to 640.5 cycles.
ADC1->SMPR1 &= ~( 0x7 << ( 6 * 3 ) );
ADC1->SMPR1 |=  ( 0x7 << ( 6 * 3 ) );

// Perform a single ADC conversion.
// (Assumes that there is only one channel per sequence)
uint16_t adc_single_conversion( ADC_TypeDef* ADCx ) {
  // Start the ADC conversion.
  ADCx->CR  |=  ( ADC_CR_ADSTART );
  // Wait for the 'End Of Conversion' flag.
  while ( !( ADCx->ISR & ADC_ISR_EOC ) ) {};
  // Read the converted value (this also clears the EOC flag).
  uint16_t adc_val = ADCx->DR;
  // Wait for the 'End Of Sequence' flag and clear it.
  while ( !( ADCx->ISR & ADC_ISR_EOS ) ) {};
  ADCx->ISR |=  ( ADC_ISR_EOS );
  // Return the ADC value.
  return adc_val;
  // adc_val between 0 and 4095 for 0V and supply voltage respectively 
}



