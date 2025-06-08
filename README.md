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
;              with capital letters.
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
  RCALL  TWI_Init                 ; initialize TWI
  RCALL  SSD1306_Init             ; initialize SSD1306
  RCALL  SSD1306_Clear            ; clear the OLED screen
  LDI    R16, 0                   ; set page to 0
  RCALL  Set_Cursor_Page          ; move cursor to page 0
  RCALL  Display_Eext_KEVIN_T     ; display "KEVIN T"
  LDI    R16, 1                   ; set page to 1
  RCALL  Set_Cursor_Page          ; move cursor to page 1
  RCALL  Display_Text_KEVIN_T     ; display "KEVIN T"
  LDI    R16, 2                   ; set page to 2
  RCALL  Set_Cursor_Page          ; move cursor to page 2
  RCALL  Display_Text_KEVIN_T     ; display "KEVIN T"
  LDI    R16, 3                   ; set page to 3
  RCALL  Set_Cursor_Page          ; move cursor to page 3
  RCALL  Display_Text_KEVIN_T     ; display "KEVIN T"
  LDI    R16, 4                   ; set page to 4
  RCALL  Set_Cursor_Page          ; move cursor to page 4
  RCALL  Display_Text_KEVIN_T     ; display "KEVIN T"
  LDI    R16, 5                   ; set page to 5
  RCALL  Set_Cursor_Page          ; move cursor to page 5
  RCALL  Display_Text_KEVIN_T     ; display "KEVIN T"
  LDI    R16, 6                   ; set page to 6
  RCALL  Set_Cursor_Page          ; move cursor to page 6
  RCALL  Display_Text_KEVIN_T     ; display "KEVIN T"
  LDI    R16, 7                   ; set page to 7
  RCALL  Set_Cursor_Page          ; move cursor to page 7
  RCALL  Display_Text_KEVIN_T     ; display "KEVIN T"
program_loop:
  RJMP   program_loop             ; infinite loop

; ===================================================================
; SUBROUTINE: TWI_Init
; ===================================================================
; Description: Initializes the Two-Wire Interface (TWI/I2C) module 
;              on the ATmega128P for communication at ~100kHz.
; -------------------------------------------------------------------
; Instructions: AVR Instruction Set Manual
;               6.95 LDI – Load Immediate
;               6.117 STS – Store Direct to Data Space
;               6.88 RET – Return from Subroutine
; ===================================================================
TWI_Init:
  ; -----------------------------------------------------------------
  ; STEP 1: Set TWI Prescaler to 1
  ; -----------------------------------------------------------------
  LDI    R16, 0x00                ; load immediate val 0x00 into R16
                                  ; 0x00 sets TWI Prescaler bits 
                                  ; (TWPS1:0) to 0
                                  ; prescaler value: 1 (as 4^TWPS)
  STS    TWSR, R16                ; store the value in TWI Status
                                  ; Register (TWSR)
                                  ; TWSR[1:0] (TWPS1, TWPS0) set the 
                                  ; prescaler
  ; -----------------------------------------------------------------
  ; STEP 2: Set TWI Bit Rate Register for ~100kHz
  ; -----------------------------------------------------------------
  LDI    R16, 72                  ; load immediate value 72 into R16
                                  ; this value sets the SCL freq
                                  ; SCL = F_CPU / 
                                  ; (16 + 2 * TWBR * 4^TWPS)
                                  ; for F_CPU = 16MHz, TWPS = 1, 
                                  ; TWBR = 72:
                                  ; SCL ≈ 100kHz
  STS    TWBR, R16                ; store the value in TWI Bit Rate 
                                  ; Register (TWBR)
                                  ; TWBR determines the SCL freq
  ; -----------------------------------------------------------------
  ; STEP 3: Enable TWI Module
  ; -----------------------------------------------------------------
  LDI    R16, (1 << TWEN)         ; load immediate value with TWEN 
                                  ; bit set
                                  ; TWEN (TWI Enable Bit) enables the
                                  ; TWI hardware
  STS    TWCR, R16                ; store the value in TWI Control 
                                  ; Register (TWCR)
                                  ; TWCR enables TWI operations with
                                  ; TWEN
  RET                             ; return from subroutine

; ===================================================================
; SUBROUTINE: TWI_Write_Byte
; ===================================================================
; Description: Sends a start condition, writes the TWI address, 
;              control byte, and data byte, then issues a stop 
;              condition.
; -------------------------------------------------------------------
; Inputs: R22 - Control Byte (0x00: Command, 0x40: Data)
;         R23 - Data Byte
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
TWI_Write_Byte:
  RCALL  .TWI_Start               ; send start condition
  LDI    R24, (SSD1306_ADDR << 1) ; load SSD1306 addr w/ write bit
  RCALL  .TWI_Write               ; write SSD1306 address
  MOV    R24, R22                 ; load control byte (Command/Data)
  RCALL  .TWI_Write               ; write control byte
  MOV    R24, R23                 ; load data byte
  RCALL  .TWI_Write               ; write data byte
  RCALL  .TWI_Stop                ; send stop condition
  RET                             ; return from subroutine
.TWI_Start:
  CLR    R16                      ; clear R16 (initialize to 0)
  ORI    R16, (1<<TWINT)          ; set TWINT bit to clear int flag
  ORI    R16, (1<<TWSTA)          ; set TWSTA bit for start cond
  ORI    R16, (1<<TWEN)           ; set TWEN bit to enable TWI
  STS    TWCR, R16                ; write to TWCR to start cond
.TWI_Wait_Start:
  LDS    R17, TWCR                ; read TWCR
  SBRS   R17, TWINT               ; skip if TWINT flag is set
  RJMP   .TWI_Wait_Start          ; wait for start cond complete
  RET                             ; return from subroutine
.TWI_Write:
  STS    TWDR, r24                ; Load data into TWDR
  CLR    R16                      ; clear R16 (initialize to 0)
  ORI    R16, (1<<TWINT)          ; set TWINT bit clear int flag
  ORI    R16, (1<<TWEN)           ; set TWEN bit to enable TWI trans
  STS    TWCR, R16                ; write to TWCR to start trans
.TWI_Write_Wait:
  LDS    R17, TWCR                ; read TWCR
  SBRS   R17, TWINT               ; skip if TWINT flag is set
  RJMP   .TWI_Write_Wait          ; wait until write is complete
  RET                             ; return from subroutine
.TWI_Stop:
  CLR    R16                      ; clear R16 (initialize to 0)
  ORI    R16, (1<<TWINT)          ; set TWINT bit to clear int flag
  ORI    R16, (1<<TWSTO)          ; set TWSTO bit for stop condition
  ORI    R16, (1<<TWEN)           ; set TWEN bit to enable TWI
  STS    TWCR, R16                ; write to TWCR to init stop cond
  RET                             ; return from subroutine

; ===================================================================
; SUBROUTINE: SSD1306_Init
; ===================================================================
; Description: Initializes the SSD1306 OLED display by sending a 
;              sequence of commands.
; -------------------------------------------------------------------
; Instructions: AVR Instruction Set Manual
;               6.95 LDI – Load Immediate
;               6.87 RCALL – Relative Call to Subroutine
;               6.88 RET – Return from Subroutine
; ===================================================================
SSD1306_Init:
  ; -----------------------------------------------------------------
  ; STEP 1: Turn Display OFF
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0xAE                ; cmd: Display OFF
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 2: Set Display Clock Divide Ratio/Oscillator Frequency
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0xD5                ; cmd: set Disp Clock Divide Ratio
  RCALL  TWI_Write_Byte           ; send cmd to SSD1306
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0x80                ; div Ratio and Oscillator Freq
  RCALL  TWI_Write_Byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 3: Set Multiplex Ratio
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0xA8                ; cmd: Set Multiplex Ratio
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0x3F                ; 1/64 duty for 128x64 display
  RCALL  TWI_Write_Byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 4: Set Display Offset
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0xD3                ; cmd: Set Display Offset
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0x00                ; value: No offset
  RCALL  TWI_Write_Byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 5: Set Start Line
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0x40                ; cmd: Set Start Line at 0
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 6: Enable Charge Pump
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0x8D                ; cmd: Charge Pump Setting
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0x14                ; value: Enable charge pump
  RCALL  TWI_Write_Byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 7: Set Memory Addressing Mode
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0x20                ; cmd: Set Memory Addressing Mode
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0x00                ; value: Horizontal Addressing Mode
  RCALL  TWI_Write_Byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 8: Set Segment Re-map
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0xA1                ; cmd: Segment Re-map
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 9: Set COM Output Scan Direction
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0xC8                ; cmd: COM Output Scan Direction
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 10: Set COM Pins Hardware Configuration
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0xDA                ; cmd: Set COM Pins Hardware Config
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0x12                ; value: alt COM pin config
  RCALL  TWI_Write_Byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 11: Set Contrast Control
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0x81                ; cmd: Set Contrast Control
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0xCF                ; value: Set contrast
  RCALL  TWI_Write_Byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 12: Set Pre-Charge Period
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0xD9                ; cmd: Set Pre-charge Period
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0xF1                ; value: P1 = 15 DCLKs, P2 = 1 DCLK
  RCALL  TWI_Write_Byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 13: Set VCOMH Deselect Level
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0xDB                ; cmd: Set VCOMH Deselect Level
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0x40                ; Value: 0.77 x Vcc
  RCALL  TWI_Write_Byte           ; send value to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 14: Set Entire Display On/Resume
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0xA4                ; cmd: Resume to RAM Content Disp
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 15: Set Normal Display
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0xA6                ; cmd: Normal Display
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  ; -----------------------------------------------------------------
  ; STEP 16: Turn Display On
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; load Control Byte (Command Mode)
  LDI    R23, 0xAF                ; cmd: Display ON
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  RET                             ; return from subroutine

; ===================================================================
; SUBROUTINE: SSD1306_Clear
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
SSD1306_Clear:
  ; -----------------------------------------------------------------
  ; STEP 1: Clear OLED Screen
  ; -----------------------------------------------------------------
  LDI    R22, 0x40                ; Control Byte: Data Mode
  LDI    R23, 0x00                ; Data Byte: 0x00 (Clear)
  LDI    R24, 8                   ; 8 pages (for 128x64 display)
.Clear_Loop:
  PUSH   R24                      ; save page counter
  LDI    R25, 128                 ; 128 columns per page
.Clear_Page:
  RCALL  TWI_Write_Byte           ; write 0x00 to clear pixel
  DEC    R25                      ; decrement column counter
  BRNE   .Clear_Page              ; repeat until all cols cleared
  POP    R24                      ; restore page counter
  DEC    R24                      ; decrement page counter
  BRNE   .Clear_Loop              ; repeat for all 8 pages
  ; -----------------------------------------------------------------
  ; STEP 2: Reset Cursor to Top-Left (0,0)
  ; -----------------------------------------------------------------
  LDI    R22, 0x00                ; Control Byte: Command Mode
  LDI    R23, 0x21                ; cmd: Set Column Address
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  LDI    R23, 0x00                ; Column Start Address (0)
  RCALL  TWI_Write_Byte           ; send start address
  LDI    R23, 0x7F                ; Column End Address (127)
  RCALL  TWI_Write_Byte           ; send end address
  LDI    R22, 0x00                ; Control Byte: Command Mode
  LDI    R23, 0x22                ; cmd: Set Page Address
  RCALL  TWI_Write_Byte           ; send command to SSD1306
  LDI    R23, 0x00                ; Page Start Address (0)
  RCALL  TWI_Write_Byte           ; send start address
  LDI    R23, 0x07                ; Page End Address (7)
  RCALL  TWI_Write_Byte           ; send end address
  RET                             ; return from subroutine

; ===================================================================
; SUBROUTINE: Set_Cursor_Page
; ===================================================================
; Description: Sets the cursor to a specific page (row) on the OLED
;              display.
; -------------------------------------------------------------------
; Inputs: R16 - Page Number (0 to 7 for 128x64 display)
; -------------------------------------------------------------------
; Instructions: AVR Instruction Set Manual
;               6.86 PUSH – Push Register on Stack
;               6.95 LDI – Load Immediate
;               6.87 RCALL – Relative Call to Subroutine
;               6.85 POP – Pop Register from Stack
;               6.75 MOV – Copy Register
;               6.88 RET – Return from Subroutine
; ===================================================================
Set_Cursor_Page:
  ; -----------------------------------------------------------------
  ; STEP 1: Set Column Address to Start at 0
  ; -----------------------------------------------------------------
  PUSH   R16                     ; save R16 to the stack
  LDI    R22, 0x00               ; Control Byte: Command Mode
  LDI    R3, 0x21                ; cmd: Set Column Address
  RCALL  TWI_Write_Byte          ; send command
  LDI    R23, 0x00               ; Column Start Address: 0
  RCALL  TWI_Write_Byte          ; send start address
  LDI    R23, 0x7F               ; Column End Address: 127
  RCALL  TWI_Write_Byte          ; send end address
  ; -----------------------------------------------------------------
  ; STEP 2: Set Page Address
  ; -----------------------------------------------------------------
  LDI    R22, 0x00               ; Control Byte: Command Mode
  LDI    R23, 0x22               ; cmd: Set Page Address
  RCALL  TWI_Write_Byte          ; send command
  POP    R16                     ; restore R16 from the stack
  MOV    R23, R16                ; copy page number from R16 to R23
  RCALL  TWI_Write_Byte          ; send page start address
  ; -----------------------------------------------------------------
  ; STEP 3: Reset Cursor to Leftmost Column (Column Address 0)
  ; -----------------------------------------------------------------
  LDI    R22, 0x00               ; Control Byte: Command Mode
  LDI    R23, 0x00               ; cmd: Set Column Address Start
  RCALL  TWI_Write_Byte          ; send column start address
  RET                            ; return from subroutine

; ===================================================================
; SUBROUTINE: Display_Char
; ===================================================================
; Description: Displays a single character on the OLED screen.
; -------------------------------------------------------------------
; Inputs: R16 - ASCII code of the letter to display
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
Display_Char:
  ; -----------------------------------------------------------------
  ; STEP 1: Check Character Validity
  ; -----------------------------------------------------------------
  LDI    R22, 0x40                ; Control Byte: Data Mode
  CPI    R16, 0x20                ; check if char is SPACE (0x20)
  BREQ   .Is_Space                ; branch if equal to .Is_Space 
  CPI    R16, 'A'                 ; check if char is 'A' or higher
  BRLO   .Invalid_Char            ; if less than 'A', invalid
  CPI    R16, 'Z'                 ; check if char is 'Z' or lower
  BRCC   .Invalid_Char            ; if greater than 'Z', invalid
  SUBI   R16, 'A'                 ; .calculate index: R16 - 'A'
  RJMP   .Calculate_Index         ; jump to .Calculate_Index
.Is_Space:
  LDI    R16, 26                  ; index for space in Font_Table
  RJMP   .Calculate_Index         ; jump to .Calculate_Index
.Invalid_Char:
  LDI    R16, 26                  ; SPACE as default invalid chars
.Calculate_Index:
  ; -----------------------------------------------------------------
  ; STEP 2: Calculate Font Data Address
  ; -----------------------------------------------------------------
  LDI    R30, LO8(font_table)     ; load lower byte into base addr
  LDI    R31, HI8(font_table)     ; load higher byte into base addr
  MOV    R17, R16                 ; copy index to R17
  LSL    R17                      ; R17 = index * 2
  ADD    R17, R16                 ; R17 = index * 3
  LSL    R17                      ; R17 = index * 6
  ADD    R17, R16                 ; R17 = index * 7
  ADD    R30, R17                 ; add offset Z-pointer (low byte)
  ADC    R31, R1                  ; add carry Z-pointer (high byte)
  ; -----------------------------------------------------------------
  ; STEP 3: Display Character
  ; -----------------------------------------------------------------
  LPM    R23, Z+                  ; load font byte 1
  RCALL  TWI_Write_Byte           ; write font byte 1
  LPM    R23, Z+                  ; load font byte 2
  RCALL  TWI_Write_Byte           ; write font byte 2
  LPM    R23, Z+                  ; load font byte 3
  RCALL  TWI_Write_Byte           ; write font byte 3
  LPM    R23, Z+                  ; load font byte 4
  RCALL  TWI_Write_Byte           ; write font byte 4
  LPM    R23, Z+                  ; load font byte 5
  RCALL  TWI_Write_Byte           ; write font byte 5
  LPM    R23, Z+                  ; load font byte 6
  RCALL  TWI_Write_Byte           ; write font byte 6
  LPM    R23, Z+                  ; load font byte 7
  RCALL  TWI_Write_Byte           ; write font byte 7
  RET                             ; return from subroutine

; ===================================================================
; SUBROUTINE: Display_Text_KEVIN_T
; ===================================================================
; Description: Displays the text "KEVIN T" on the current cursor 
;              position of the OLED display.
; -------------------------------------------------------------------
; Instructions: AVR Instruction Set Manual
;               6.95 LDI – Load Immediate
;               6.87 RCALL – Relative Call to Subroutine
;               6.88 RET – Return from Subroutine
; ===================================================================
Display_Text_KEVIN_T:
  ; Display "KEVIN T"
  LDI    R16, 'K'                 ; load ASCII value of 'K' into R16
  RCALL  Display_Char             ; display 'K'
  LDI    R16, 'E'                 ; load ASCII value of 'E' into R16
  RCALL  Display_Char             ; display 'E'
  LDI    R16, 'V'                 ; load ASCII value of 'V' into R16
  RCALL  Display_Char             ; display 'V'
  LDI    R16, 'I'                 ; load ASCII value of 'I' into R16
  RCALL  Display_Char             ; display 'I'
  LDI    R16, 'N'                 ; load ASCII value of 'N' into R16
  RCALL  Display_Char             ; display 'N'
  LDI    R16, ' '                 ; load ASCII value of space R16
  RCALL  Display_Char             ; display space
  LDI    R16, 'T'                 ; load ASCII value of 'T' into R16
  RCALL  Display_Char             ; display 'T'
  RET                             ; return from subroutine

; ===================================================================
; FONT TABLE: A-Z AND SPACE
; ===================================================================
; Description: Defines the font data for characters A-Z and space as  
;              each character consists of 7 bytes.
; ===================================================================
Font_Table:
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

