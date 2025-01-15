<img src="https://raw.githubusercontent.com/mytechnotalent/ATmega328P_SSD1306_Driver/refs/heads/main/ATmega328P%20SSD1306%20Driver.png">

## FREE Reverse Engineering Self-Study Course [HERE](https://github.com/mytechnotalent/Reverse-Engineering-Tutorial)

<br>

# ATmega328P SSD1306 Driver
An ATmega328P SSD1306 driver written entirely in Assembler.

<br>

# Code
```assembler
; ===================================================================
; Project: ATmega328P SSD1306 Driver
; ===================================================================
; Author: Kevin Thomas
; E-Mail: ket189@pitt.edu
; Version: 1.0
; Date: 01/09/25
; Target Device: ATmega328P (Arduino Nano)
; Clock Frequency: 16 MHz
; Toolchain: AVR-AS, AVRDUDE
; License: Apache License 2.0
; Description: This program is a simple SSD1306 driver that works
;              capital letters.
; ===================================================================

; ===================================================================
; SYMBOLIC DEFINITIONS
; ===================================================================
.equ     TWBR, 0xB8               ; TWI Bit Rate Register
.equ     TWSR, 0xB9               ; TWI Status Register
.equ     TWDR, 0xBB               ; TWI Data Register
.equ     TWCR, 0xBC               ; TWI Control Register
.equ     TWINT, 7                 ; TWI Interrupt Flag
.equ     TWEA, 6                  ; TWI Enable Acknowledge
.equ     TWSTA, 5                 ; TWI Start Condition
.equ     TWSTO, 4                 ; TWI Stop Condition
.equ     TWEN, 2                  ; TWI Enable Bit
.equ     SSD1306_ADDR, 0x3C       ; SSD1306 I2C Address

; ===================================================================
; PROGRAM ENTRY POINT
; ===================================================================
.global  program                  ; global label; make avail external
.section .text                    ; start of the .text (code) section

; ===================================================================
; PROGRAM LOOP
; ===================================================================
; Description: Main program loop which executes all subroutines and 
;              then repeats indefinitely.
; -------------------------------------------------------------------
; Instructions: AVR Instruction Set Manual
;               6.87 RCALL – Relative Call to Subroutine
;               6.95 LDI – Load Immediate
;               6.90 RJMP – Relative Jump
; ===================================================================
program:
  rcall  TWI_init                 ; initialize TWI
  rcall  SSD1306_init             ; initialize SSD1306
  rcall  SSD1306_clear            ; clear the OLED screen
  ldi    r16, 0                   ; set page to 0
  rcall  set_cursor_page          ; move cursor to page 0
  rcall  display_text_KEVIN_T     ; display "KEVIN T"
  ldi    r16, 1                   ; set page to 1
  rcall  set_cursor_page          ; move cursor to page 1
  rcall  display_text_KEVIN_T     ; display "KEVIN T"
  ldi    r16, 2                   ; set page to 2
  rcall  set_cursor_page          ; move cursor to page 2
  rcall  display_text_KEVIN_T     ; display "KEVIN T"
  ldi    r16, 3                   ; set page to 3
  rcall  set_cursor_page          ; move cursor to page 3
  rcall  display_text_KEVIN_T     ; display "KEVIN T"
  ldi    r16, 4                   ; set page to 4
  rcall  set_cursor_page          ; move cursor to page 4
  rcall  display_text_KEVIN_T     ; display "KEVIN T"
  ldi    r16, 5                   ; set page to 5
  rcall  set_cursor_page          ; move cursor to page 5
  rcall  display_text_KEVIN_T     ; display "KEVIN T"
  ldi    r16, 6                   ; set page to 6
  rcall  set_cursor_page          ; move cursor to page 6
  rcall  display_text_KEVIN_T     ; display "KEVIN T"
  ldi    r16, 7                   ; set page to 7
  rcall  set_cursor_page          ; move cursor to page 7
  rcall  display_text_KEVIN_T     ; display "KEVIN T"
program_loop:
  rjmp   program_loop             ; infinite loop

; ===================================================================
; SUBROUTINE: TWI_init
; ===================================================================
; Description: Initializes the Two-Wire Interface (TWI/I2C) module 
;              on the ATmega128P for communication at ~100kHz.
; -------------------------------------------------------------------
; Instructions: AVR Instruction Set Manual
;               6.95 LDI – Load Immediate
;               6.117 STS – Store Direct to Data Space
;               6.88 RET – Return from Subroutine
; ===================================================================
TWI_init:
  ; -----------------------------------------------------------------
  ; STEP 1: Set TWI Prescaler to 1
  ; -----------------------------------------------------------------
  ldi    r16, 0x00                ; load immediate val 0x00 into r16
                                  ; 0x00 sets TWI Prescaler bits 
                                  ; (TWPS1:0) to 0
                                  ; prescaler value: 1 (as 4^TWPS)
  sts    TWSR, r16                ; store the value in TWI Status
                                  ; Register (TWSR)
                                  ; TWSR[1:0] (TWPS1, TWPS0) set the 
                                  ; prescaler
  ; -----------------------------------------------------------------
  ; STEP 2: Set TWI Bit Rate Register for ~100kHz
  ; -----------------------------------------------------------------
  ldi    r16, 72                  ; load immediate value 72 into r16
                                  ; this value sets the SCL freq
                                  ; SCL = F_CPU / 
                                  ; (16 + 2 * TWBR * 4^TWPS)
                                  ; for F_CPU = 16MHz, TWPS = 1, 
                                  ; TWBR = 72:
                                  ; SCL ≈ 100kHz
  sts    TWBR, r16                ; store the value in TWI Bit Rate 
                                  ; Register (TWBR)
                                  ; TWBR determines the SCL freq
  ; -----------------------------------------------------------------
  ; STEP 3: Enable TWI Module
  ; -----------------------------------------------------------------
  ldi    r16, (1 << TWEN)         ; load immediate value with TWEN 
                                  ; bit set
                                  ; TWEN (TWI Enable Bit) enables the
                                  ; TWI hardware
  sts    TWCR, r16                ; store the value in TWI Control 
                                  ; Register (TWCR)
                                  ; TWCR enables TWI operations with
                                  ; TWEN
  ret                             ; return from subroutine

; ===================================================================
; SUBROUTINE: TWI_write_byte
; ===================================================================
; Description: Sends a start condition, writes the TWI address, 
;              control byte, and data byte, then issues a stop 
;              condition.
; -------------------------------------------------------------------
; Inputs: r22 - Control Byte (0x00: Command, 0x40: Data)
;         r23 - Data Byte
; -------------------------------------------------------------------
; Instructions: AVR Instruction Set Manual
;               6.87 RCALL – Relative Call to Subroutine
;               6.95 LDI – Load Immediate
;               6.75 MOV – Copy Register
;               6.88 RET – Return from Subroutine
;               6.39 CLR - Clear Register
;               6.83 ORI – Logical OR with Immediate
;               6.117 STS – Store Direct to Data Space
;               6.70 LDS – Load Direct from Data Space
;               6.101 SBRS – Skip if Bit in Register is Set
;               6.90 RJMP – Relative Jump
; ===================================================================
TWI_write_byte:
  rcall  .TWI_start               ; send start condition
  ldi    r24, (SSD1306_ADDR << 1) ; load SSD1306 addr w/ write bit
  rcall  .TWI_write               ; write SSD1306 address
  mov    r24, r22                 ; load control byte (Command/Data)
  rcall  .TWI_write               ; write control byte
  mov    r24, r23                 ; load data byte
  rcall  .TWI_write               ; write data byte
  rcall  .TWI_stop                ; send stop condition
  ret                             ; return from subroutine
.TWI_start:
  clr    r16                      ; clear r16 (initialize to 0)
  ori    r16, (1<<TWINT)          ; set TWINT bit to clear int flag
  ori    r16, (1<<TWSTA)          ; set TWSTA bit for start cond
  ori    r16, (1<<TWEN)           ; set TWEN bit to enable TWI
  sts    TWCR, r16                ; write to TWCR to start cond
.TWI_wait_start:
  lds    r17, TWCR                ; read TWCR
  sbrs   r17, TWINT               ; skip if TWINT flag is set
  rjmp   .TWI_wait_start          ; wait for start cond complete
  ret                             ; return from subroutine
.TWI_write:
  sts    TWDR, r24                ; Load data into TWDR
  clr    r16                      ; clear r16 (initialize to 0)
  ori    r16, (1<<TWINT)          ; set TWINT bit clear int flag
  ori    r16, (1<<TWEN)           ; set TWEN bit to enable TWI trans
  sts    TWCR, r16                ; write to TWCR to start trans
.TWI_write_wait:
  lds    r17, TWCR                ; read TWCR
  sbrs   r17, TWINT               ; skip if TWINT flag is set
  rjmp   .TWI_write_wait          ; wait until write is complete
  ret                             ; return from subroutine
.TWI_stop:
  clr    r16                      ; clear r16 (initialize to 0)
  ori    r16, (1<<TWINT)          ; set TWINT bit to clear int flag
  ori    r16, (1<<TWSTO)          ; set TWSTO bit for stop condition
  ori    r16, (1<<TWEN)           ; set TWEN bit to enable TWI
  sts    TWCR, r16                ; write to TWCR to init stop cond
  ret                             ; return from subroutine

; ===================================================================
; SUBROUTINE: SSD1306_init
; ===================================================================
; Description: Initializes the SSD1306 OLED display by sending a 
;              sequence of commands.
; -------------------------------------------------------------------
; Instructions: AVR Instruction Set Manual
;               6.95 LDI – Load Immediate
;               6.87 RCALL – Relative Call to Subroutine
;               6.88 RET – Return from Subroutine
; ===================================================================
SSD1306_init:
  ; -----------------------------------------------------------------
  ; STEP 1: Turn Display OFF
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0xAE                ; cmd: Display OFF
  rcall  TWI_write_byte           ; send command to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 2: Set Display Clock Divide Ratio/Oscillator Frequency
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0xD5                ; cmd: set Disp Clock Divide Ratio
  rcall  TWI_write_byte           ; send cmd to SSD1306
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0x80                ; div Ratio and Oscillator Freq
  rcall  TWI_write_byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 3: Set Multiplex Ratio
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0xA8                ; cmd: Set Multiplex Ratio
  rcall  TWI_write_byte           ; send command to SSD1306
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0x3F                ; 1/64 duty for 128x64 display
  rcall  TWI_write_byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 4: Set Display Offset
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0xD3                ; cmd: Set Display Offset
  rcall  TWI_write_byte           ; send command to SSD1306
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0x00                ; value: No offset
  rcall  TWI_write_byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 5: Set Start Line
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0x40                ; cmd: Set Start Line at 0
  rcall  TWI_write_byte           ; send command to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 6: Enable Charge Pump
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0x8D                ; cmd: Charge Pump Setting
  rcall  TWI_write_byte           ; send command to SSD1306
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0x14                ; value: Enable charge pump
  rcall  TWI_write_byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 7: Set Memory Addressing Mode
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0x20                ; cmd: Set Memory Addressing Mode
  rcall  TWI_write_byte           ; send command to SSD1306
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0x00                ; value: Horizontal Addressing Mode
  rcall  TWI_write_byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 8: Set Segment Re-map
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0xA1                ; cmd: Segment Re-map
  rcall  TWI_write_byte           ; send command to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 9: Set COM Output Scan Direction
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0xC8                ; cmd: COM Output Scan Direction
  rcall  TWI_write_byte           ; send command to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 10: Set COM Pins Hardware Configuration
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0xDA                ; cmd: Set COM Pins Hardware Config
  rcall  TWI_write_byte           ; send command to SSD1306
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0x12                ; value: alt COM pin config
  rcall  TWI_write_byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 11: Set Contrast Control
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0x81                ; cmd: Set Contrast Control
  rcall  TWI_write_byte           ; send command to SSD1306
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0xCF                ; value: Set contrast
  rcall  TWI_write_byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 12: Set Pre-charge Period
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0xD9                ; cmd: Set Pre-charge Period
  rcall  TWI_write_byte           ; send command to SSD1306
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0xF1                ; value: P1 = 15 DCLKs, P2 = 1 DCLK
  rcall  TWI_write_byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 13: Set VCOMH Deselect Level
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0xDB                ; cmd: Set VCOMH Deselect Level
  rcall  TWI_write_byte           ; send command to SSD1306
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0x40                ; Value: 0.77 x Vcc
  rcall  TWI_write_byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 14: Set Entire Display ON/Resume
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0xA4                ; cmd: Resume to RAM Content Disp
  rcall  TWI_write_byte           ; send command to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 15: Set Normal Display
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0xA6                ; cmd: Normal Display
  rcall  TWI_write_byte           ; send command to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 16: Turn Display ON
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; load Control Byte (Command Mode)
  ldi    r23, 0xAF                ; cmd: Display ON
  rcall  TWI_write_byte           ; send command to SSD1306
  ret                             ; return from subroutine

; ===================================================================
; SUBROUTINE: SSD1306_clear
; ===================================================================
; Description: Clears the OLED screen and resets the cursor to the 
;              top-left.
; -------------------------------------------------------------------
; Instructions: AVR Instruction Set Manual
;               6.95 LDI – Load Immediate
;               6.86 PUSH – Push Register on Stack
;               6.87 RCALL – Relative Call to Subroutine
;               6.49 DEC – Decrement
;               6.23 BRNE – Branch if Not Equal
;               6.85 POP – Pop Register from Stack
;               6.88 RET – Return from Subroutine
; ===================================================================
SSD1306_clear:
  ; -----------------------------------------------------------------
  ; STEP 1: Clear OLED Screen
  ; -----------------------------------------------------------------
  ldi    r22, 0x40                ; Control Byte: Data Mode
  ldi    r23, 0x00                ; Data Byte: 0x00 (Clear)
  ldi    r24, 8                   ; 8 pages (for 128x64 display)
.clear_loop:
  push   r24                      ; save page counter
  ldi    r25, 128                 ; 128 columns per page
.clear_page:
  rcall  TWI_write_byte           ; write 0x00 to clear pixel
  dec    r25                      ; decrement column counter
  brne   .clear_page              ; repeat until all cols cleared
  pop    r24                      ; restore page counter
  dec    r24                      ; decrement page counter
  brne   .clear_loop              ; repeat for all 8 pages
  ; -----------------------------------------------------------------
  ; STEP 2: Reset Cursor to Top-Left (0,0)
  ; -----------------------------------------------------------------
  ldi    r22, 0x00                ; Control Byte: Command Mode
  ldi    r23, 0x21                ; cmd: Set Column Address
  rcall  TWI_write_byte           ; send command to SSD1306
  ldi    r23, 0x00                ; Column Start Address (0)
  rcall  TWI_write_byte           ; send start address
  ldi    r23, 0x7F                ; Column End Address (127)
  rcall  TWI_write_byte           ; send end address
  ldi    r22, 0x00                ; Control Byte: Command Mode
  ldi    r23, 0x22                ; cmd: Set Page Address
  rcall  TWI_write_byte           ; send command to SSD1306
  ldi    r23, 0x00                ; Page Start Address (0)
  rcall  TWI_write_byte           ; send start address
  ldi    r23, 0x07                ; Page End Address (7)
  rcall  TWI_write_byte           ; send end address
  ret                             ; return from subroutine

; ===================================================================
; SUBROUTINE: set_cursor_page
; ===================================================================
; Description: Sets the cursor to a specific page (row) on the OLED
;              display.
; -------------------------------------------------------------------
; Inputs: r16 - Page Number (0 to 7 for 128x64 display)
; -------------------------------------------------------------------
; Instructions: AVR Instruction Set Manual
;               6.86 PUSH – Push Register on Stack
;               6.95 LDI – Load Immediate
;               6.87 RCALL – Relative Call to Subroutine
;               6.85 POP – Pop Register from Stack
;               6.75 MOV – Copy Register
;               6.88 RET – Return from Subroutine
; ===================================================================
set_cursor_page:
  ; -----------------------------------------------------------------
  ; STEP 1: Set Column Address to Start at 0
  ; -----------------------------------------------------------------
  push   r16                     ; save r16 to the stack
  ldi    r22, 0x00               ; Control Byte: Command Mode
  ldi    r23, 0x21               ; cmd: Set Column Address
  rcall  TWI_write_byte          ; send command
  ldi    r23, 0x00               ; Column Start Address: 0
  rcall  TWI_write_byte          ; send start address
  ldi    r23, 0x7F               ; Column End Address: 127
  rcall  TWI_write_byte          ; send end address
  ; -----------------------------------------------------------------
  ; STEP 2: Set Page Address
  ; -----------------------------------------------------------------
  ldi    r22, 0x00               ; Control Byte: Command Mode
  ldi    r23, 0x22               ; cmd: Set Page Address
  rcall  TWI_write_byte          ; send command
  pop    r16                     ; restore r16 from the stack
  mov    r23, r16                ; copy page number from r16 to r23
  rcall  TWI_write_byte          ; send page start address
  ; -----------------------------------------------------------------
  ; STEP 3: Reset Cursor to Leftmost Column (Column Address 0)
  ; -----------------------------------------------------------------
  ldi    r22, 0x00               ; Control Byte: Command Mode
  ldi    r23, 0x00               ; cmd: Set Column Address Start
  rcall  TWI_write_byte          ; send column start address
  ret                            ; return from subroutine

; ===================================================================
; SUBROUTINE: display_char
; ===================================================================
; Description: Displays a single character on the OLED screen.
; -------------------------------------------------------------------
; Inputs: r16 - ASCII code of the letter to display
; -------------------------------------------------------------------
; Instructions: AVR Instruction Set Manual
;               6.95 LDI – Load Immediate
;               6.47 CPI – Compare with Immediate
;               6.14 BREQ – Branch if Equal
;               6.20 BRLO – Branch if Lower (Unsigned)
;               6.11 BRCC – Branch if Carry Cleared
;               6.120 SUBI – Subtract Immediate
;               6.87 RCALL – Relative Call to Subroutine
;               6.75 MOV – Copy Register
;               6.73 LSL – Logical Shift Left
;               6.2 ADD – Add without Carry
;               6.1 ADC – Add with Carry
;               6.72 LPM – Load Program Memory
;               6.88 RET – Return from Subroutine
; ===================================================================
display_char:
  ; -----------------------------------------------------------------
  ; STEP 1: Check Character Validity
  ; -----------------------------------------------------------------
  ldi    r22, 0x40                ; Control Byte: Data Mode
  cpi    r16, 0x20                ; check if char is SPACE (0x20)
  breq   .is_space                ; branch if equal to .is_space 
  cpi    r16, 'A'                 ; check if char is 'A' or higher
  brlo   .invalid_char            ; if less than 'A', invalid
  cpi    r16, 'Z'                 ; check if char is 'Z' or lower
  brcc   .invalid_char            ; if greater than 'Z', invalid
  subi   r16, 'A'                 ; .calculate index: r16 - 'A'
  rjmp   .calculate_index         ; jump to .calculate_index
.is_space:
  ldi    r16, 26                  ; index for space in font_table
  rjmp   .calculate_index         ; jump to .calculate_index
.invalid_char:
  ldi    r16, 26                  ; SPACE as default invalid chars
.calculate_index:
  ; -----------------------------------------------------------------
  ; STEP 2: Calculate Font Data Address
  ; -----------------------------------------------------------------
  ldi    r30, lo8(font_table)     ; load lower byte into base addr
  ldi    r31, hi8(font_table)     ; load higher byte into base addr
  mov    r17, r16                 ; copy index to r17
  lsl    r17                      ; r17 = index * 2
  add    r17, r16                 ; r17 = index * 3
  lsl    r17                      ; r17 = index * 6
  add    r17, r16                 ; r17 = index * 7
  add    r30, r17                 ; add offset Z-pointer (low byte)
  adc    r31, r1                  ; add carry Z-pointer (high byte)
  ; -----------------------------------------------------------------
  ; STEP 3: Display Character
  ; -----------------------------------------------------------------
  lpm    r23, Z+                  ; load font byte 1
  rcall  TWI_write_byte           ; write font byte 1
  lpm    r23, Z+                  ; load font byte 2
  rcall  TWI_write_byte           ; write font byte 2
  lpm    r23, Z+                  ; load font byte 3
  rcall  TWI_write_byte           ; write font byte 3
  lpm    r23, Z+                  ; load font byte 4
  rcall  TWI_write_byte           ; write font byte 4
  lpm    r23, Z+                  ; load font byte 5
  rcall  TWI_write_byte           ; write font byte 5
  lpm    r23, Z+                  ; load font byte 6
  rcall  TWI_write_byte           ; write font byte 6
  lpm    r23, Z+                  ; load font byte 7
  rcall  TWI_write_byte           ; write font byte 7
  ret                             ; return from subroutine

; ===================================================================
; SUBROUTINE: display_text_KEVIN_T
; ===================================================================
; Description: Displays the text "KEVIN T" on the current cursor 
;              position of the OLED display.
; -------------------------------------------------------------------
; Instructions: AVR Instruction Set Manual
;               6.95 LDI – Load Immediate
;               6.87 RCALL – Relative Call to Subroutine
;               6.88 RET – Return from Subroutine
; ===================================================================
display_text_KEVIN_T:
  ; Display "KEVIN T"
  ldi    r16, 'K'                 ; load ASCII value of 'K' into r16
  rcall  display_char             ; display 'K'
  ldi    r16, 'E'                 ; load ASCII value of 'E' into r16
  rcall  display_char             ; display 'E'
  ldi    r16, 'V'                 ; load ASCII value of 'V' into r16
  rcall  display_char             ; display 'V'
  ldi    r16, 'I'                 ; load ASCII value of 'I' into r16
  rcall  display_char             ; display 'I'
  ldi    r16, 'N'                 ; load ASCII value of 'N' into r16
  rcall  display_char             ; display 'N'
  ldi    r16, ' '                 ; load ASCII value of space r16
  rcall  display_char             ; display space
  ldi    r16, 'T'                 ; load ASCII value of 'T' into r16
  rcall  display_char             ; display 'T'
  ret                             ; return from subroutine

; ===================================================================
; FONT TABLE: A-Z AND SPACE
; ===================================================================
; Description: Defines the font data for characters A-Z and space as  
;              each character consists of 7 bytes.
; ===================================================================
font_table:
  ; LETTER_A
  .byte 0x00, 0x7C, 0x12, 0x11, 0x12, 0x7C, 0x00
  ; LETTER_B
  .byte 0x00, 0x7F, 0x49, 0x49, 0x49, 0x36, 0x00
  ; LETTER_C
  .byte 0x00, 0x3E, 0x41, 0x41, 0x41, 0x22, 0x00
  ; LETTER_D
  .byte 0x00, 0x7F, 0x41, 0x41, 0x22, 0x1C, 0x00 
  ; LETTER_E
  .byte 0x00, 0x7F, 0x49, 0x49, 0x49, 0x41, 0x00
  ; LETTER_F
  .byte 0x00, 0x7F, 0x09, 0x09, 0x09, 0x01, 0x00
  ; LETTER_G
  .byte 0x00, 0x3E, 0x41, 0x49, 0x49, 0x7A, 0x00
  ; LETTER_H
  .byte 0x00, 0x7F, 0x08, 0x08, 0x08, 0x7F, 0x00
  ; LETTER_I
  .byte 0x00, 0x00, 0x41, 0x7F, 0x41, 0x00, 0x00
  ; LETTER_J
  .byte 0x00, 0x20, 0x40, 0x41, 0x3F, 0x01, 0x00
  ; LETTER_K
  .byte 0x00, 0x7F, 0x08, 0x14, 0x22, 0x41, 0x00
  ; LETTER_L
  .byte 0x00, 0x7F, 0x40, 0x40, 0x40, 0x40, 0x00
  ; LETTER_M
  .byte 0x00, 0x7F, 0x02, 0x0C, 0x02, 0x7F, 0x00
  ; LETTER_N
  .byte 0x00, 0x7F, 0x04, 0x08, 0x10, 0x7F, 0x00
  ; LETTER_O
  .byte 0x00, 0x3E, 0x41, 0x41, 0x41, 0x3E, 0x00
  ; LETTER_P
  .byte 0x00, 0x7F, 0x09, 0x09, 0x09, 0x06, 0x00
  ; LETTER_Q
  .byte 0x00, 0x3E, 0x41, 0x51, 0x21, 0x5E, 0x00
  ; LETTER_R
  .byte 0x00, 0x7F, 0x09, 0x19, 0x29, 0x46, 0x00
  ; LETTER_S
  .byte 0x00, 0x46, 0x49, 0x49, 0x49, 0x31, 0x00
  ; LETTER_T
  .byte 0x00, 0x01, 0x01, 0x7F, 0x01, 0x01, 0x00
  ; LETTER_U
  .byte 0x00, 0x3F, 0x40, 0x40, 0x40, 0x3F, 0x00
  ; LETTER_V
  .byte 0x00, 0x1F, 0x20, 0x40, 0x20, 0x1F, 0x00
  ; LETTER_W
  .byte 0x00, 0x3F, 0x40, 0x38, 0x40, 0x3F, 0x00
  ; LETTER_X
  .byte 0x00, 0x63, 0x14, 0x08, 0x14, 0x63, 0x00
  ; LETTER_Y
  .byte 0x00, 0x07, 0x08, 0x70, 0x08, 0x07, 0x00
  ; LETTER_Z
  .byte 0x00, 0x61, 0x51, 0x49, 0x45, 0x43, 0x00
  ; SPACE
  .byte 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00
```

<br>

## License
[Apache License 2.0](https://github.com/mytechnotalent/ATmega328P_SSD1306_Driver/blob/main/LICENSE)

